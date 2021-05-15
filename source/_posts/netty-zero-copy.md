---
title: Netty————零拷贝
date: 2021-03-29 12:18:39
tags: [Netty]
---



# 什么是零拷贝(zero-copy)


零拷贝指的是没有CPU拷贝，而不是没有拷贝时间。操作系统可以分为用户态和内核态，在内核态缓冲区之间，没有数据是重复的。零拷贝意味着更少的数据复制、更少的上下文切换、CPU缓存伪共享以及无CPU检验和计算。零拷贝是网络编程的核心，很多性能优化都离不开。


文件复制可以分为两个流程，一部分是文件的读，另外一部分是文件的写。在传统的IO文件操作时，假如我们要读取文件，有以下几个流程：
+	调用操作系统提供的底层标准IO系统调用函数read()，进行一次上下文切换（用户态--->内核态）；
+	OS的内核代码将相应的文件数据读取到内核的IO 缓冲区，是一次DMA Copy（内核从磁盘上面读取数据 是 不消耗CPU时间的，是通过磁盘控制器完成）
+	数据再由内核的IO 缓冲区拷贝到进程的私有空间中，是一次CPU Copy
+	read调用返回后，会再进行一次上下文切换（内核态--->用户态）

假如我们要写入文件，有以下几个流程：
+	应用发起写操作，OS进行一次上下文切换（从用户空间切换为内核空间）
+	数据copy到内核缓冲区Socket Buffer，做了一次CPU Copy
+	内核空间再把数据copy到磁盘或其他存储（网卡，进行网络传输），进行了DMA Copy
+	写入结束后返回，又从内核空间切换到用户空间

综上，传统的复制操作会发生四次上下文切换、两次DMA复制、两次CPU复制。为此，零拷贝提供了mmap+write方式、sendfile方式提高复制效率。

# 拷贝优化的演化

1. mmap+write方式 
mmap+write方式是使用虚拟内存的特性，将内核空间和用户空间的虚拟地址映射到同一个物理地址，这样就不需要来回复制了。读取时把数据存放到内核缓冲区而不必应用程序缓冲区，写入数据的时候直接从内核缓冲区读取到内核缓冲区即可，这需要花费一次CPU Copy。这个方式将花费四次上下文切换、两次DMA复制、一次CPU复制。

2. sendfile方式 
sendfile方式这种方式可以替换上面的mmap+write方式，它少了一个应用程序发起write操作，直接发起sendfile操作。这个方式将花费两次上下文切换、两次DMA复制、一次CPU复制。

3. gather操作（零拷贝基础）
而Linux2.4内核进行了优化，提供了gather操作。这个操作在内核空间Read Buffer和Socket Buffer不做数据复制，而是将Read Buffer的内存地址、偏移量记录到相应的Socket Buffer中，这样就不需要复制。这个方式将花费两次上下文切换、两次DMA复制。

JAVA零拷贝 java nio实现零拷贝，JAVA提供了一下方法类：
+	MappedByteBuffer mmap+write方式
+	DirectByteBuffer 堆外内存
+	FileChannel.transferTo


Linux提供的零拷贝技术 Java并不是全支持，支持2种(内存映射mmap、sendfile)：

+	NIO提供的内存映射 MappedByteBuffer，底层就是调用Linux mmap()实现的。
	+	```
	MappedByteBuffer mappedByteBuffer = new RandomAccessFile(file, "r") 
                                 .getChannel() 
                                .map(FileChannel.MapMode.READ_ONLY, 0, len);
	```
+	NIO提供的sendfile
	+	FileChannel.transferTo()方法直接将当前通道内容传输到另一个通道，没有涉及到Buffer的任何操作，NIO中 的Buffer是JVM堆或者堆外内存，但不论如何他们都是操作系统内核空间的内存
	+	transferTo()的实现方式就是通过系统调用sendfile() (当然这是Linux中的系统调用)

# Netty的零拷贝

Netty中的Zero-copy与上面我们所提到到OS层面上的Zero-copy不太一样, Netty的Zero-copy完全是在用户态(Java层面)的，它的Zero-copy的更多的是偏向于优化数据操作这样的概念。

+	应用层数据优化的零拷贝
	+	Netty提供了CompositeByteBuf类，它可以将多个ByteBuf合并为一个逻辑上的ByteBuf，避免了各个ByteBuf之间的拷贝。
		+	```
			// ByteBuf合并操作
			resultBuf.writeBytes(header);
			resultBuf.writeBytes(body);
		```
	+	ByteBuf 支持slice 操作，因此可以将ByteBuf分解为多个共享同一个存储区域的ByteBuf，避免了内存的拷贝。
		+	```
			// ByteBuf分散操作
			ByteBuf header = byteBuf.slice(0,5);
			ByteBuf body = byteBuf.slice(5,10);
		```
	+	通过wrap操作，我们可以将byte[]数组、ByteBuf、 ByteBuffer 等包装成一个 Netty ByteBuf对象，进而避免了拷贝操作。
		+	```
			// byte[]--> ByteBuf
			byte[] bytes;
			ByteBuf byteBuf = Unpooled.buffer();
			byteBuf.writeBytes(bytes);
			Unpooled.wrappedBuffer(bytes);
		```
+	操作系统级别的零拷贝
	+	通过FileRegion包装的FileChannel.tranferTo实现文件传输，可以直接将文件缓冲区的数据发送到目标Channel，避免了传统通过循环write方式导致的内存拷贝问题。（底层依赖 Java NIO FileChannel.transferTo）


# 参考资料
> [零拷贝(zero-copy)](https://juejin.cn/post/6844903984965091336)
[Java中的零拷贝](https://blog.csdn.net/akunshouyoudou/article/details/104637539)