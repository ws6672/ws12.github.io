---
title: Netty（一）网络I/O模型的Java实现
date: 2021-03-15 13:01:03
tags: [Netty]
---


# 一、基础

Java 1.4之前的早期版本，Java对I/O的支持并不完善，开发人员在开发高性能I/O程序的时候，会面临一些巨大的挑战和困难，主要问题如下：
+	没有数据缓冲区，I/O性能存在问题；
+	没有C或者C+＋中的Channel概念，只有输入和输出流；
+	同步阻塞式I/0通信(BIO)，通常会导致通信线程被长时间阻塞；
+	支持的字符集有限，硬件可移植性不好。

Linux的内核将所有外部设备都看做一个文件来操作，对一个文件的读写操作会调用内核提供的系统命令，返回一个 file descriptor (fd, 文件描述符）。而对一个 socket 的读写也会有相应的描述符，称为 socketfd (socket 描述符），描述符就是一个数字，它指向内核中的一个结构体（文件路径， 数据区等一些屈性）。

1. UNIX的网络I/O模型

UNIX提供了5种网络I/O模型:
+	阻塞I/O模型：客户端向服务器端发出请求后，客户端会一直处于等待状态（不会再做其他事情），直到服务器端返回结果或者网络出现问题 ，服务器端同样如此。进程只处理一个请求，并且全程是阻塞的。

+	非阻塞I/O模型：recvfrom 从应用层到内核的时候， 如果该缓冲区没有数据的话，
就直接返回一个EWOULDBLOCK错误。一般都会对非阻塞I/O模型进行轮询，查看缓冲区是否有数据。进程只处理一个请求，通过轮询重复调用尝试获取结果。

+	I/O复用模型：Linux提供select/poll. 进程通过将一个或多个fd传递给select或poll系统调用，阻塞在select操作上，这样select/poll 可以帮我们侦测多个fd是否处于就绪状态。select/poll是顺序扫描fd是否就绪，而且支持的fd数目有限，因此它的使用受到了一些制约。Linux还提供了 个epoll系统调用，epoll使用基于事件驱动方式代替顺序扫描，因此性能更高。当有fd就绪时，立即回调函数rollback。用户调用select时，进程被阻塞，这时候会轮询多个流，当有一个或多个准备好后返回；select具有O(n)的无差别轮询复杂度，同时处理的流越多，无差别轮询时间就越长。

+	信号驱动I/O模型（嵌入式使用较多）：首先开启套接口信号驱动I/O功能，并通过系统调用sigaction 执行一个信号处理函数（此系统调用立即返回， 进程继续工作，它是非阻塞的）。当数据准备就绪时，就为该进程生成一个SIGIO信号，通过信号回调通知应用程序调用recvfrom 来读取数据，并通知主循环函数处理数据。无论如何处理 SIGIO 信号，这种模型的优势在于等待数据报到达(第一阶段)期间，进程可以继续执行，不被阻塞。免去了select 的阻塞与轮询，当有活跃套接字时，由注册的 handler 处理。

+	异步I/O：告知内核启动某个操作，并让内核在整个操作完成后（包括将数据从 内核复制到用户自己的缓冲区）通知我们。 这种模型与信号驱动模型的主要区别是： 信号驱动I/O由内核通知我们何时可以开始一个I/O操作； 异步I/O模型由内核通知我们I/O操作何时已经完成。


2. I/O多路复用技术

在I/O编程过程中，当需要同时处理多个客户端接入请求时，可以利用多线程或者I/O多路复用技术进们处理。I/O多路复用技术通过把多个I/O的阻塞复用到同一个select的阻塞上，从而使得系统在单线程的情况下可以同时处理多个客户端请求。目前支待I/O多路复用的系统调用有select、 pselect、 poll、 epoll，select轮询存在缺陷，所以Linux内核用了epoll替代了select。

epoll的特点如下：

+	支持一个进程打开的 socket 描述符 (FD) 不受限制（仅受限于操作系统的最大文件句柄数，）。select最大的缺陷就是单个进程所打开的 FD 是有一定限制的， 它由 FD_SETSIZE 设置，默认值是1024。epoll 并没有这个限制，它所支持的 FD 上限是操作系统的最大文件句柄数，具体的值可以通过 cat /proc/sys/fs/file- max 查看。
+	I/O效率不会随若 FD数目的增加而线性下降。epoll 是根据每个 fd 上面的 callback 函数实现的，只有 “活跃 ” 的 socket 才会去主动调用 callback 函数。
+	使用 mmap 加速内核与用户空间的消息传递。
+	epoll的API更加简单。

3. Java 的I/O

在JDK1.4推出Java NIO之前，基于Java的所有Socket通信都采用了同步阻塞模式(BIO)，这种请求————应答的通信梑型简化了上层的应用开发 ，但是在性能和可靠性方面 却存在若巨大的瓶颈。JDKl.4版本提供了新的NIO类库，也可以支持非阻塞I/O了。

JDK1.4 时 ， NIO 以 JSR-51 的身份正式随 JDK 发布。 它新增了个 java.nio 包， 提供了很多 进行异步I/O开发的 API 和类库， 主要的类和接口如下：

+	进行异步I/O操作的缓冲区 ByteBuffer 等； 进行异步I/O操作的管道 Pipe;
+	进行各种I/O操作（异步或者同步）的 Channel, 包括 ServerSocketChannel 和SocketChannel; 
+	多种字符集的编码能力和解码能力；
+	实现非阳塞I/O操作的多路复用器 selector;
+	基于流行的 Perl 实现的正则表达式类库；
+	文件通道 FileChannel。


4. AIO

NIO 2.0引入了新的异步通道的概念，并提供了异步文件通道和异步套接宇通道的实现。异步通道提供以下两种方式获取获取操作结果：

+	通过java.uti.concurrent.Future类来表示异步操作的结果 
+	使用回调函数

NIO 2.0的异步套接字通道是真正的异步非阻塞I/O,对应的UNIX网络编程中的事件 驱动I/O (AIO)。 它不需要通过多路复用器(Selector)对注册的通道进行轮询操作即可实现异步读写，从而简化了NIO的编程模型。


# 二、同步阻塞式I/O（BIO）


网络编程的基本模型是 Clieot/Server 模型，也就是两个进程之间进行相互通信，其中服务端提供位置信息（绑定的IP地址和监听端口）， 客户端通过连接操作向服务端监听的地址发起连接请求，通过三次握手建立连接;如果连接建立成功，双方就可以通过网络套接字 (Socket) 进行通信。


BIO服务端：

```
public class BIOServerTest {
    public static void main (String[] args) throws IOException {
        int port = 8080;

        if(args!=null && args.length>0) {
            try {
                port = Integer.parseInt(args[0]);
            }  catch (NumberFormatException e) {
                e.printStackTrace();
            }
        }

        ServerSocket server = null;

        try {
            //从连接队列中取出一个连接，如果没有则等待
            server = new ServerSocket(port);
            Socket socket = null;


            try {
                socket = server.accept();
                new Thread(new BIOServerHandler(socket)).start();
            }  catch (Exception e) {
                e.printStackTrace();
            }  finally {
                if (server != null) {
                    server.close();
                    server = null;
                }
            }


        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            if (server!=null) {
                server.close();
            }
        }
    }
}

public class BIOServerHandler implements Runnable{
    private Socket socket;

    public BIOServerHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {

        BufferedReader in = null;
        PrintWriter out = null;
        try {
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream(), true);
            String body = null;
            while (true) {
//                获取客户端报文
                body = in.readLine();
                if (body == null) {
                    continue;
                }
//                触发回馈
                if ("QUERY TIME ORDER".equals(body)) {
                    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                    out.println(format.format(new Date()));
                }
                System.out.println("Client body："+body);
                break;
            }
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            if(out != null) {
                out.close();
                out = null;
            }
            if(in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                in = null;
            }
            if(socket != null) {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                socket = null;
            }
        }
    }
}
```


BIO客户端：
```
public class BIOClientTest {
    public static void main (String[] args) {
        int port = 8080;
        // 端口参数检测
        if(args!=null && args.length>0) {
            try {
                port = Integer.parseInt(args[0]);
            }  catch (NumberFormatException e) {
                e.printStackTrace();
            }
        }
        Socket socket = null;
        BufferedReader in = null;
        PrintWriter out = null;

        try {
            socket = new Socket("127.0.0.1", port);
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream(), true);
//            查询指令
            out.println("QUERY TIME ORDER");
//            获取回馈
            String resp = in.readLine();
            System.out.println("The resp："+ resp);

        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            if(out != null) {
                out.close();
                out = null;
            }
            if(in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                in = null;
            }
            if(socket != null) {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                socket = null;
            }
        }
    }
}
```

同步式IO的相关代码如上所示。但是存在一个问题，就是当存在一个服务端连接只能处理一个客户端连接，效率过低。而高性能的服务器，需要同时处理成千上万的客户端连接，这种模型是无法满足并发要求的。当然我们也可以基于线程池和同步阻塞式I/O构建伪异步IO,将Socket封装成任务异步调用，客户端代码不变，底层仍然是同步阻塞的。


# 三、非阻塞式IO（NIO）编程

NIO（JDK1.4）模型是一种同步非阻塞IO，主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector（多路复用器）。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(多路复用器)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。


### 基础

0. NIO和BIO的比较

BIO，即阻塞IO；而NIO是非阻塞IO。
BIO是以流的方式处理数据；而NIO是以块的方式处理数据。
BIO是基于字节流和字符流操作；而NIO基于 Channel和Buffer 进行操作，通过通道读取数据到缓冲区，或者通过缓冲区写入到通道中。Selector（选择器）用于监听多个通道的事件，因此可以通过单线程监听多个连接。

1. 缓冲区Buffer

Buffer是一个对象，它包含一些要写入或者要读出的数据。在 NIO 类库中加入 Buffer对象，体现了新库与原IO的一个重要区别。在“面向流的”IO 中，可以将数据直接写入或者将数据直接读到 Stream 对象中。

它是NIO重要的组成部分，表明了NIO是面向缓冲区的而不是流。在 NIO 库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的；在写入数据时，写入到缓冲区中。任何时候访问 NIO 中的数据，都是通过缓冲区进行操作。

缓冲区是一个可读取的内存，实质上是一个数组。通常它是一个字节数组 (ByteBuffer)， 也可以使用其他种类的数组。但是一个缓冲区不仅仅是一个数组。缓冲区提供了对数据的结构化访问以及维护读写位置 (limit) 等信息 。

+	Buffer
	+	ByteBuffer: 字节缓冲区
		+	MappedByteBuffer：支持文件在内存中修改，使用的是堆外内存
	+	CharBuffer: 字符缓冲区
	+	ShortBuffer：短整型缓冲区
	+	IntBuffer：整形缓冲区
	+	LongBuffer: 长整形缓冲区
	+	FloatBuffer：浮点型缓冲区
	+	DoubleBuffer: 双精度浮点型缓冲区
	
Buffer读写数据的步骤如下：
+	把数据写入buffer；
+	调用flip，把buffer从写模式调整为读模式；在读模式下，可以读取所有已经写入的数据。
+	从Buffer中读取数据；
+	调用buffer.clear()或者buffer.compact()；clear会清空整个buffer，compact则只清空已读取的数据。

Buffer缓冲区实质上就是一块内存，可以进行读取和写入。Buffer有四个属性是必备的，分别是：
+	capacity容量：是缓冲区可容纳的最大数据量，缓冲区创建时设置且不可更改。
+	limit限制：表示缓冲区的当前终点，不能对超过的位置进行读写操作
+	position位置：下一个要读或者要写的索引，每次读写缓冲区都会改值
+	mask：标记，用于position的临时保存。调用mask()设置mask=position，再调用reset()可以让position恢复它原来的位置


> 注：NIO还支持通过多个Buffer数组完成读写操作，即Buffer的分散和聚集。分散，是数据写入多个buffer；聚集是从多个buffer读取数据
Buffer的常见方法：

|方法|介绍|
|:--|:--|
|abstract Object array()|返回支持此缓冲区的数组 （可选操作）|
|abstract int arrayOffset()|返回该缓冲区的缓冲区的第一个元素的在数组中的偏移量 （可选操作）|
|int capacity()|返回此缓冲区的容量|
|Buffer clear()|清除此缓存区。将position = 0;limit = capacity;mark = -1;|
|Buffer flip()|flip()方法可以吧Buffer从写模式切换到读模式。调用flip方法会把position归零，并设置limit为之前的position的值。 也就是说，现在position代表的是读取位置，limit标示的是已写入的数据位置。|
|abstract boolean hasArray()|告诉这个缓冲区是否由可访问的数组支持|
|boolean hasRemaining()|return position < limit，返回是否还有未读内容|
|abstract boolean isDirect()|判断个缓冲区是否为 direct|
|abstract boolean isReadOnly()|判断告知这个缓冲区是否是只读的|
|int limit()|返回此缓冲区的限制|
|Buffer position(int newPosition)|设置这个缓冲区的位置|
|int remaining()|return limit - position; 返回limit和position之间相对位置差|
|Buffer rewind()|把position设为0，mark设为-1，不改变limit的值|
|Buffer mark()|将此缓冲区的标记设置在其位置|


例如：
```
public class Test {
	public static void main (String[] args) {
        IntBuffer intBuffer = IntBuffer.allocate(24);
        for (int i=0; i<24; i++) {
            intBuffer.put((int) (Math.random()*100));
        }
        intBuffer.flip(); // 读写切换

        while (intBuffer.hasRemaining()) { // 测试是否有数据
            // Buffer 内部维持一个索引
            System.out.println(intBuffer.get());
        }
		intBuffer.close();
    }
}
```



2. 通道Channel

Channel是一个通道，网络数据通过Channel同时读取和写入。通道与流的不同之处在千通道是双向的，流只是在一个方向上移动（一个流必须是InputStream 或者 OutputStream 的子类），而通道可以用于读、写或者二者同时进行。因为 Channel 是全双工的，所以它可以比流更好地映射底层操作系统的 API。

+	特点
	+	同时读写
	+	异步读写
	+	可以从缓冲区读写数据
+	常用Channel（抽象类）
	+	FileChannel： 文件的数据读写				
	+	SocketChannel： TCP的数据读写，一般是客户端实现
	+	ServerSocketChannel: 允许我们监听TCP链接请求，每个请求会创建会一个SocketChannel，一般是服务器实现
	+	DatagramChannel： UDP的数据读写
+	相关方法
	+	int read(ByteBuffer dst) 从通道读取数据
	+	int write(ByteBuffer src) 写入数据到通道
	+	long transferFrom(ReadableByteChannel src,long position, long count) 从目标通道复制数据到当前通道
	+	long transferTo(long position, long count,WritableByteChannel target) 从当前通道复制数据到目标通道


接口源码：
```
public interface Channel extends Closeable {
    public boolean isOpen();
    public void close() throws IOException;
}
```

transferFrom实例：
```
    public static void main (String[] args) throws IOException {

        FileInputStream fis = new FileInputStream("E:\\wsz6672\\ws6672.github\\source\\image\\19819\\a2.png");
        FileOutputStream fos = new FileOutputStream(new File("E:\\wsz6672\\ws6672.github\\source\\image\\19819\\b1-copy.png"));

        FileChannel fromC = fis.getChannel();
        FileChannel toC = fos.getChannel();
        toC.transferFrom(fromC, 0,fromC.size());

        fromC.close();
        toC.close();
        fis.close();
        fos.close();
    }
```

通道Channel的实例如下：

```
public class ChannelTest {
    public static void main(String[] args) {
        FileChannel fileChannel = null;
        RandomAccessFile raf = null;

        try {
            raf = new RandomAccessFile("F://test.txt","rw");
            fileChannel = raf.getChannel();
            int buf_size = 512;
            if(buf_size>fileChannel.size()) {
                buf_size = (int)fileChannel.size();
            }
            ByteBuffer readbuf = ByteBuffer.allocate(buf_size);
//            ByteBuffer writebuf = ByteBuffer.allocate(buf_size);

            while (fileChannel.read(readbuf) >0) {
                readbuf.flip();
// 就是判断position和limit之间是否有元素
                System.out.print(Charset.forName("UTF-8").decode(readbuf).toString());
                readbuf.clear();

            }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fileChannel != null) {
                    fileChannel.close();
                }
                if (fileChannel != null) {
                    fileChannel.close();
                }
                if (raf != null) {
                    raf.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

3. 多路复用器Selector

多路复用器提供选择已经就绪的任务的能力，Selector会不断地轮询注册在其上的 Channel, 如果某个 Channel 上面发生读或者写事件， 这个 Channel 就处于就绪状态，会被 Selector 轮询出来，然后通过 SelectionKey 可以获取就绪 Channel 的集合，进行后续的 I/O 操作。一个多路复用器 Selector 可以同时轮询多个 Channel, 由千 JDK 使用了 epoll(）代替传统的 select 实现 ， 所以它并没有最大连接句柄 1024/2048 的限制。这也就意味着只需要一个线程负责 Selector 的轮询，就可以接入成千上万的客户端.

Selector能够检测多个注册的通道是否有事件发生，多个Channel以事件的方式注册到同一个 Selector。只有在连接通道有读写事件发生时才进行相应的处理，减少了多线程切换的开销，避免了阻塞导致的性能下降。一个I/O线程可以并发处理N个客户端连接和读写操作，从根本上解决了同步阻塞模型一连接一线程模型的低效问题，架构的性能、弹性伸缩能力和可靠性得到了极大的提升。

+	源码
	+	```
		public abstract class Selector implements Closeable {

			// 初始化类实例
			protected Selector() { }

			//打开一个多路复用器
			public static Selector open() throws IOException {
				return SelectorProvider.provider().openSelector();
			}

			//测试多路复用器是否打开
			public abstract boolean isOpen();
			//返回通道创建者
			public abstract SelectorProvider provider();
			//当前连接的keys
			public abstract Set<SelectionKey> keys();
			//访问“已选择键集（selected key set）”中的就绪通道
			public abstract Set<SelectionKey> selectedKeys();
			
			//不会阻塞，不管什么通道就绪都立刻返回
			public abstract int selectNow() throws IOException;
			
			//和select()一样，除了设置最长阻塞时间timeout毫秒
			public abstract int select(long timeout) throws IOException;
			//阻塞,直到至少有一个通道就绪
			public abstract int select() throws IOException;
			public abstract void close() throws IOException;

		}
	```


4. 三大核心的关系

+	Selector 对应一个线程，一个线程对应多个Channel；每个Channel对应一个Buffer。
+	Selector切换到哪一个通道由监听到的事件决定，它会根据不同的事件在不同的通道上切换。
+	Buffer是一个内存块，底层是一个数组；可读可写，但是需要通过flip 方法切换。
+	channel是双向的


### java实现NIO


1. reactor（反应器）模式

使用单线程模拟多线程，提高资源利用率和程序的效率，增加系统吞吐量。单线程下要多个操作执行完成后才处理其它请求，伪多线程是在单线程处理某些需要长时间等待的操作时，先处理其它请求。


2. 服务端和客户端序列图 

![NIO 服务端序列图](/image/netty/nio-server-time.png)

![NIO 客户端序列图](/image/netty/nio-client-time.png)


3. SocketChannel的方法


register() 方法定义如下：
```
public final SelectionKey register(Selector sel, int ops)
        throws ClosedChannelException
```

第一个参数是 多路复用器，第二个参数是监听的事件，相关事件如下：
+	SelectionKey.OP_ACCEPT 接收就绪（服务器准备好接收连接）,16
+	SelectionKey.OP_CONNECT 连接就绪（通道连接到服务器）,8
+	SelectionKey.OP_WRITE 写就绪,4
+	SelectionKey.OP_READ 读就绪,1
+	```
public static final int OP_ACCEPT = 1 << 4;
public static final int OP_CONNECT = 1 << 3;
public static final int OP_WRITE = 1 << 2;
public static final int OP_READ = 1 << 0;
```

我们也可以同时监听多个事件：`int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE`。


4. SelectionKey

SelectionKey表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。

```
key.attachment(); //返回SelectionKey的attachment，attachment可以在注册channel的时候指定。
key.channel(); // 返回该SelectionKey对应的channel。
key.selector(); // 返回该SelectionKey对应的Selector。
key.interestOps(); //返回代表需要Selector监控的IO操作的bit mask
key.readyOps(); // 返回一个bit mask，代表在相应channel上可以进行的IO操作。
```

5. NIO 服务端和客户端示例

服务端如下：
```
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

/**
 * @author zws
 * @date 2021/3/8 13:55
 * @Description TODO
 */
public class NIOTest implements Runnable {

    private int port;
    //用于轮询的多路复用器
    private Selector selector;
    // 服务器通道
    private ServerSocketChannel acceptor;

    private ServerSocket serverSocket;

    public NIOTest() {
        this(9999);
    }

    public NIOTest(int port) {
        this.port = port;
        try {
//          1. 监听客户端连接
            acceptor = ServerSocketChannel.open();
//          2. 绑定端口、用通道对象生成服务器对象、为服务端Socket绑定监听端口
            acceptor.configureBlocking(false);
            serverSocket = acceptor.socket();
            serverSocket.bind(new InetSocketAddress("localhost", port));
            selector = Selector.open();
            acceptor.register(selector, SelectionKey.OP_ACCEPT);
            System.out.println("Server Start");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        while (selector!=null && selector.isOpen()) {
            try {
                selector.select(1000);
                Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                SelectionKey key = null;
                while (it.hasNext()) {
                    key = it.next();
                    it.remove();
                    handleAccept(key);
                }
            }  catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public void handleAccept(SelectionKey key) {

        if (key.isValid()) {
            if (key.isAcceptable()) {
//                在多路复用器注册客户端连接
                SocketChannel sc = null;
                try {
                    sc = ((ServerSocketChannel) key.channel()).accept();
                    if (sc != null) {
                        sc.configureBlocking(false);
                        sc.register(selector, SelectionKey.OP_READ);
                    }
                }  catch (Exception e) {
                    e.printStackTrace();
                }
            }

            if (key.isReadable()) {
//               处理客户端连接请求
                handleRead(key);
            }
            System.out.println("客户端请求完成");
        }
    }

    public void handleRead(SelectionKey key) {
        SocketChannel sc = null;
        ByteBuffer buffer = null;

        try {
            sc = (SocketChannel)key.channel();
            //创建缓存区
            buffer = ByteBuffer.allocate(1024);
            int byteSize = sc.read(buffer);
            if (byteSize==-1) {
                sc.shutdownInput();
                sc.shutdownOutput();
                sc.close();
                System.out.println("连接断开");
            } else {
                buffer.flip();
                byte[] data = new byte[buffer.remaining()];
                buffer.get(data);
                buffer.clear();
                String content = new String(data);
                System.out.println("Server receiver message: "+content);
                respon(sc, "from Server's respon");
            }

        }  catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void respon(SocketChannel sc, String message) {
        ByteBuffer buffer = null;
        try {
            if (message != null && message.length() != 0) {
                byte[] data = message.getBytes();
                buffer = ByteBuffer.allocate(1024);
                buffer.put(data);
                buffer.flip();
                sc.write(buffer);
                buffer.clear();
            }
        }  catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static void main (String[] args) {
        NIOTest nioTest = new NIOTest(9999);
        new Thread(nioTest).start();
    }
}

```

客户端如下：
```
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

/**
 * @author zws
 * @date 2021/3/9 22:15
 * @Description TODO
 */
public class NIOClient implements Runnable {

    private String  host;
    private int port;
    private Selector selector;
    private SocketChannel socketChannel;

    public NIOClient() {
        this("localhost", 9999);
    }

    public NIOClient(String  host, int port) {
        this.host = host;
        this.port = port;
        try {
            socketChannel = SocketChannel.open();
            selector = Selector.open();
            socketChannel.configureBlocking(false);
            socketChannel.connect(new InetSocketAddress("localhost",port));
            socketChannel.register(selector, SelectionKey.OP_CONNECT);
        }  catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        while (selector != null && selector.isOpen()) {
            try {
                selector.select(1000);
                //获得所有已就绪的通道
                Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                SelectionKey key = null;
                while(it.hasNext()) {
                    key = it.next();
                    it.remove();
                    handleInput(key);
                }

            }  catch (Exception e) {
                e.printStackTrace();
            }
        }
    }



    public void send (SocketChannel sc,String content) {
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        if (content!=null && content.length() != 0) {
            try {
                byte[] data = content.getBytes();
                byteBuffer.put(data);
                byteBuffer.flip();
                sc.write(byteBuffer);
            }  catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public void handleInput(SelectionKey key) {
        SocketChannel sc = null;
        if (key.isValid()) {
            if (key.isConnectable()) {
                try {
                    sc = (SocketChannel) key.channel();
                    if (sc.finishConnect()) {
                        sc.register(selector, SelectionKey.OP_READ);
                        send(sc, "client send message");
                    }
                }  catch (Exception e) {
                    e.printStackTrace();
                }
            } else if (key.isReadable()) {
                handleRead(key);
            }

        }
    }

    private void handleRead(SelectionKey key) {
        SocketChannel sc = (SocketChannel) key.channel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        try {
            int byte_size = sc.read(buffer);
            if (byte_size>0) {
                buffer.flip();
                byte[] data = new byte[buffer.remaining()];
                buffer.get(data);
                buffer.clear();
                String content = new String(data);
                System.out.println(content);
            } else {
                //表示读取数据失败,需要释放资源
                key.cancel();
                sc.close();
                buffer.clear();
            }
        }  catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main (String[] args) {
        NIOClient nioClient = new NIOClient();
        new Thread(nioClient).start();
    }
}

```

由以上例子可知，NIO编程的难度确实比同步阻塞BIO的大很多，在上面的实例中还没考虑半包读、半包写、编码解码等问题。虽然NIO编写复杂，但是它还是在网络编程中应用广泛，优点如下：
+	客户端发起的连接操作是异步的，无须阻塞
+	SocketChannel的读写操作都是异步的， 如果没有可读写的数据它不会同步 等待，直接返回 ， 这样I/O通信线程就可以处理其他的链路， 不需要同步等待这个链路可用。
+	线程模型的优化，通过epoll避免了连接限制。


***

# 四、总结

虽然有五种网络通信的IO方式，但是Java只实现了四种（没有信号驱动I/O模型）。

相关概念如下：

+	异步非阻塞 I/O：很多人喜欢将JDK 1.4提供的NlO框架称为异步非阻塞 I/O， 但是，如果严格按照UNIX网络编程 模型和JDK的实现进行区分， 实际上它只能被称为非阻塞VO, 不能叫异步非阻塞 I/O。由JDKl.7提供的NIO2.0新增了异步的套接字通道， 它是真正的异步 I/O, 在异步 I/O 操作的时候可以传递信号变晁， 当操作完成之后会回调相关的方法，异步I/O也被称为AIO
+	多路复用器 Selector：多路复用的核心就是通过 Selector 来轮询注册在其上的 Channel，当发现某个或者多个 Channel 处于就绪状态后 ，从阻塞状态返回就绪的 Channel 的选择键集合，进行 I/O 操作 。 
+	伪异步I/O：在通信线程和业务线程之间做个缓冲区， 这个缓冲区用千隔离I/O线程和业务线程间的直接访问， 这样业务线程就不会被 I/O线程阻塞。线程接收连接，将连接封装为Task后放到线程池中然后返回去处理其它的请求。

基于NIO的网络框架NIO是一种网络并发解决方案，使用广泛。但是，Netty不是全能的，具体选择什么样的I/O 模型或者NIO框架， 完全基于业务的实际应用场景和性能诉求，如果客户端并发连接数不多，周边对接的网元不多，服务器的负载也不重， 那就完全没必要选择NlO做服务端；如果是相反情况，那就要考虑选择合适的 NIO框架进行开发。


不选择 Java 原生 NIO 编程的原因：
+	NIO 的类库和 API 繁杂，使用麻烦， 你需要熟练掌握 Selector 、 ServerSocketChannel、 SocketChannel、 ByteBuffer
+	需要熟悉 Java 多线程编程等额外技能
+	可靠性差
+	JDK NIO 的 BUG, 例如臭名昭著的 epoll bug, 它会导致 Selector 空轮询， 最终导 致 CPU 100%

如果需要开发一个网络并发组件，那么Netty值得一试。




