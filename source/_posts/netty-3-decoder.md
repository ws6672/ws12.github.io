---
title: Netty（三）TCP粘包／拆包
date: 2021-03-15 13:09:30
tags: [Netty]
---

TCP是个 “流 ” 协议，所谓流，就是没有界限的一串数据。在业务上认为，一个完整的包可能会被TCP拆分成多个包进行发送， 也有可能把多个小的包封装成一个大的数据包发送，这就是所谓的TCP粘包和拆包问题。


# 一、基础

假设客户端分别发送了两个数据包D1和D2给服务端，由于服务端一次读取到的字节数是不确定的，故可能存在以下 4 种情况：

+	服务端分两次读取到了两个独立的数据包，分别是 D1和D2, 没有粘包和拆包
+	服务端一次接收到了两个数据包，D1和D2 粘合在一起，被称为TCP粘包
+	服务端分两次读取到了两个数据包，第一次读取到了完整的D1和D2包的部分内容，第二次读取到了D2包的剩余内容，这被称为TCP拆包
+	服务端分两次读取到了两个数据包，第一次读取到了部分的D1包，第二次读取到了D1包的剩余内容和D2包，这被称为TCP拆包

如果此时服务端TCP接收滑窗非常小，而数据包 D1和D2 比较大，很有可能会发生第5种可能，即服务端分多次才能将DI和02包接收完全，期间发生多次拆包。



1. TCP 粘包／拆包发生的原因：

+	应用程序write写入的字节大小大于套接口发送缓冲区大小
+	进行MSS 大小的TCP分段
+	以太网帧的payload大千MTU进行IP分片


2. MTU和MSS

+	MTU: Maxitum Transmission Unit 最大传输单元，一般是 1500 字节
+	MSS: Maxitum Segment Size 最大分段大小，用MTU代替（MTU - 数据包包头的大小20Bytes- TCP数据段的包头20Bytes = 1460Bytes）

在连接建立的时候，即在发送SYN段的时候，同时会将MSS发送给对方（MSS选项只能出现在SYN段中！！！），告诉对端他期望接收的TCP报文段数据部分最大长度。网络传输数据时，数据是最终是要交付到链路层协议上的，也就是说最后要封装成“帧”。二型以太网（Ethernet Type 2）中规定，帧的大小不能超过 1518 个字节（14 字节的帧头 + 4 字节帧校验和 + 最多 1500 字节数据）。所以 IP 数据报的大小如果超过了 1500 字节，要想交付给链路层就必须进行“分片”, 这个值我们就把它称之为MTU。


3. 粘包问题的解决策略

由于底层的TCP无法理解上层的业务数据，所以在底层是无法保证数据包不被拆分和重组的，这个问题只能通过上层的应用协议栈设计来解决， 根据业界的主流协议的解决方案， 可以归纳如下：

+	消息定长，例如每个报文的大小为固定长度200字节，如果不够，空位补空格
+	在包尾增加回车换行符进行分割， 例如FTP协议
+	将消息分为消息头和消息体， 消息头中包含表示消息总长度（或者消息体长度）的字段， 通常设计思路为消息头的第一个字段使用int32来表示消息的总长度
+	更复杂的应用层协议


4. Netty的粘包解决策略

TCP以流的方式进行数据传输， 上层的应用协议为了对消息进行区分， 往往采用如下4种方式：
+	消息长度固定（FixedLengthFrameDecoder）
+	将回车换行符作为消息结束符（LineBasedFrameDecoder）
+	将特殊的分隔符作为泭息的结束标志（DelimiterBasedFrameDecoder）
+	通过在消息头中定义长度字段来标识消息的总长度（LengthFieldBasedFrameDecoder ）

Netty对上面4种应用做了统一的抽象，提供了4种解码器来解决对应的问题，使用起来非常方便。有了这些解码器， 用户不需要自己对读取的报文进行人工解码，也不需要考虑TCP的粘包和拆包。



# 二、四种解码器 

为了解决TCP粘包／拆包导致的半包读写问题，Netty默认提供了多种编解码器用千处理半包，只要能熟练掌握这类库的使用， TCP粘包问题从此会变得非常容易，你甚至不需要关心它们，这也是其他NIO框架和JDK原生的NIO API所无法匹敌的。

LineBasedFrameDecoder  的工作原理是“它依次遍历ByteBuf中的可读字节，判断看是否有‘\n’或者‘\r\n’，如果有，就在此位置为结束位置，从可读索引到结束位置区间的字节就组成了一行 ”。LineBasedFrameDecoder实例如下：

```

serverBootstrap.group(boss, worker)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 1024)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));
                            socketChannel.pipeline().addLast(new StringDecoder());
                            socketChannel.pipeline().addLast(new NettySerHandler());
                        }
                    });

```

LineBasedFrameDecoder 的工作原理是它依次遍历ByteBuf 中的可读字节，判断看是否有“ \n,, 或者” \r\n"， 如果有， 就以此位置为结束位置，从可读索引到结束位置区间的字节就组成了一行。它是以换行符为结束标志的解码器， 支持携带结束符或者不携带结束符两种解码方式，同时支持配置单行的最大长度。如果连续读取到最大长度后仍然没有发现换行符，就会抛出异常，同时忽略掉之前读到的异常码流。

StringDecoder 的功能非常简单，就是将接收到的对象转换成字符串，然后继续调用后面的Handler。LineBasedFrameDecoder + StringDecoder 组合就是按行切换的文本解码器，它被设计用来支持TCP 的粘包和拆包。



还有另外两种解码器：DelirnitcrBasedFrameDecoder和 FixedLengthFrameDecoder,前者可以自动完成以分隔符做结束标志的消息的解码， 后者可以自动完成对定长消息的解码，它们都能解决TCP粘包／拆包导致的读半包问题。

DelimiterBasedFrameDecoder实例如下，可以自定义分隔符

```
serverBootstrap.group(boss, worker)
	.channel(NioServerSocketChannel.class)
	.option(ChannelOption.SO_BACKLOG, 1024)
	.childHandler(new ChannelInitializer<SocketChannel>() {
		@Override
		protected void initChannel(SocketChannel socketChannel) throws Exception {
			socketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));
			socketChannel.pipeline().addLast(new StringDecoder());
			socketChannel.pipeline().addLast(new NettySerHandler());
		}
	});

class NettySerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("客户端接入");
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("客户端断开");
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("From client:"+msg);
        ByteBuf res = Unpooled.copiedBuffer("to Client: ok$_$".getBytes());
        ctx.writeAndFlush(res);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}

```


FixedLengthFrameDecoder 固定长度解码器，它能够按照指定的长度对消息进行自动解码，开发者不需要考虑 TCP 的粘包与拆包问题，非常实用。无论一次接收到多少数据报，它都会按照构造器中设置的固定长度进行解码，如果是半包消息，FixedLengthFrameDecoder  会缓存半包消息并等待下个包到达之后进行拼包合并，直到读取一个完整的消息包。

如果消息长度不够，则使用空位填补空缺，这样读取到了之后，只需要 trim 去掉空格即可。如果消息不够长，那么就会存储起来。

FixedLengthFrameDecoder 实例如下：

```
bootstrap.group(worker)
	.channel(NioSocketChannel.class)
	.option(ChannelOption.TCP_NODELAY, true)
	.handler(new ChannelInitializer<SocketChannel>() {
		@Override
		protected void initChannel(SocketChannel socketChannel) throws Exception {
			socketChannel.pipeline().addLast(new FixedLengthFrameDecoder(10));
			socketChannel.pipeline().addLast(new StringDecoder());
			socketChannel.pipeline().addLast(new NettyCliHandler());
		}
	});
```


