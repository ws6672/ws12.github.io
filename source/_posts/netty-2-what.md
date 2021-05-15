---
title: Netty（二）入门
date: 2021-03-21 13:02:35
tags: [Netty]
---


# 一、基础

Netty是由JBOSS提供的一个java开源框架，现为 Github上的独立项目。Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。

也就是说，Netty 是一个基于NIO的客户、服务器端的编程框架，使用Netty 可以确保你快速和简单的开发出一个网络应用，例如实现了某种协议的客户、服务端应用。Netty相当于简化和流线化了网络应用的编程开发过程，例如：基于TCP和UDP的socket服务开发。

Netty和Tomcat的区别：Netty和Tomcat最大的区别就在于通信协议，Tomcat是基于Http协议的，他的实质是一个基于http协议的web容器，但是Netty不一样，他能通过编程自定义各种协议，因为netty能够通过codec自己来编码/解码字节流，完成类似redis访问的功能，这就是netty和tomcat最大的不同。


Netty的优点
+	并发高：底层使用了NIO的网络I/O模型，该模型是非阻塞的，阻塞业务处理但不阻塞数据接收
+	传输快：依赖于零拷贝特性。当需要接收数据的时候，他会在堆内存之外开辟一块内存，数据就直接从IO读到了那块内存中去，在netty里面通过ByteBuf可以直接对这些数据进行直接操作，从而加快了传输速度。
+	封装好

### Netty的线程模型

1. 目前存在的线程模型

+	传统阻塞IO模型：采用阻塞式IO，每个连接都需要独立的线程进行处理；该模型的缺陷在于高并发时占用了大量的系统资源；阻塞模型需要等待资源到位，造成资源浪费。


+	Reactor模式
	+	单Reactor单线程
	+	单Reactor多线程
	+	主从Reactor多线程

![线程模型简图](/image/netty/reactor.png)

2. 单Reactor单线程 AND 单Reactor多线程


![单Reactor单线程与单Reactor多线程](/image/netty/reactor-1.png)

如上图所示，单Reactor单线程有一个分发器，一个处理器。而单Reactor多线程是它的改进版本，相关细节如下：

+	Reactor对象通过select监听客户端请求事件，收到事件后通过 dispatch 进行分发
	+	如果是请求连接事件，由Acceptor通过 accept处理连接请求，它会创建一个Handler对象处理连接完成后的各种事件。
	+	非请求连接事件则由Reactor对象分发到对应的handler处理。
		+	handler只负责响应事件，不负责实际的业务处理，通过read读取数据后，会分发给后面的worker线程池的某个线程处理业务
			+	worker线程池会分配独立线程完成真正的业务，并将结果返回给handler；handler收到响应后，通过send返回结果给client。


3. 主从Reactor多线程

但是，单Reactor多线程模型仍然存在缺陷。由单个Reactor分发线程，还是存在性能瓶颈。主从Reactor多线程就是为了解决单个Reactor的性能瓶颈，相关细节如下：

+	Reactor主线程 MainReactor 对象通过select监听连接事件，收到事件后，通过Acceptor处理连接事件；
	+	当 Acceptor 处理连接事件后，MainReactor 将连接分配给 SubReactor；
		+	SubReactor 将连接加入到连接队列进行监听，并创建Handler进行事件处理；
+	当新事件发生时，SubReactor 就会调用对应的 handler 处理；
	+	handler 通过read读取数据，分发给后面的 worker线程池 处理；
		+	worker线程池 分配独立的 worker线程 进行业务处理，并返回结果；
	+	handler 收到响应结果后，再通过 send 将结果返回给 client。

![主从Reactor多线程](/image/netty/reactor-2.png)

主从Reactor多线程实现了请求连接和处理的分离，将连接事件由主线程响应；而请求的读写事件等由子线程进行响应。


4. Reactor模式的构成基础

而Netty线程模型主要基于主从Reactor多线程模型的改进版本，主从Reactor多线程模型有多个Reactor.

+	构成基础
	+	基于I/O复用模型（Reactor），多个连接共用一个阻塞对象，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接；只要有一个及以上的请求接入，就把它们包装成任务分配给其它线程处理。
	+	基于线程池复用资源（Handlers），通过线程池分配资源执行任务。
+	设计思想
	+	Reactor模式，通过一个或多个输入同时传递给服务器的模式（基于事件驱动）。服务器程序处理传入的多个请求并将它们分配给相应的处理线程，因此它也称之为Dispatcher模式（分发者模式）。
	+	Reactor模式使用IO复用监听事件，手动事件后分给某个线程，这就是网络服务器高并发处理的关键。


如下图所示，Netty 抽象出两组线程池。其中，BossGroup 专门负责接收客户端连接；而 WorkerGroup 专门负责网络的读写。

+	BossGroup 和 WorkerGroup 类型都是 NioEventLoopGroup；NioEventLoopGroup 相当于一个事件循环组，该组含有多个事件循环，每个事件循环都是一个 NioEventLoop。
	+	NioEventLoop 表示一个不断循环的执行处理任务的线程，每个 NioEventLoop 都有一个 selector, 用于监听绑定在其上的socket网络通讯
	+	NioEventLoopGroup 可以拥有多个线程（含有多个 NioEventLoop）
+	BossGroup 循环的步骤
	+	轮询 accept事件
	+	处理 accept事件，与client建立连接，生成 NioSocketChannel，并将其注册到某个 worker NIOEventLoop 上的 selector中
	+	处理任务队列的任务，即 runAllTasks
+	WorkerGroup 循环的步骤
	+	轮询 read，write事件
	+	处理I/O事件，调用对应 NioSocketChannel 处理
	+	处理任务队列的任务，即 runAllTasks


![Netty模型](/image/netty/reactor-3.png)


# 二、入门

访问的官网http://netty.io/， 从【Downloads】标签页选择下载`netty-4.1.60.Final.tar.bz2`。如果你习惯使用包管理器，例如maven等。你可以新建一个项目，然后倒入以下的依赖：
```
<dependencies>
  ...

	<!-- https://mvnrepository.com/artifact/io.netty/netty-all -->
	<dependency>
		<groupId>io.netty</groupId>
		<artifactId>netty-all</artifactId>
		<version>4.1.59.Final</version>
	</dependency>
  ...
</dependencies>
```


通过ServerBootstrap 服务端辅助类来启动服务端代码：
+	定义两个线程组，处理客户端的 Accept和读写事件
+	绑定NIO服务端通道 NioServerSocketChannel
+	为读写事件的线程通道绑定handle，处理具体的业务逻辑
+	绑定监听


### 实例

```

import io.netty.bootstrap.Bootstrap;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

public class NettyTest {
    public static void main(String[] args) throws Exception {


    }
}

class NettySer {
    public static void main(String[] args) throws Exception {
        new NettySer().bind(9132);
    }
    public void bind(int port) throws Exception {
		//            处理处理客户端的 Accept 事件
        EventLoopGroup boss = null;
		//            处理客户端读写事件
        EventLoopGroup worker =  null;
		//            服务端辅助类
        ServerBootstrap b =  null;
        try {
			// 如果不设置线程池的大小，那么默认大小为 CPU核数*2
            boss = new NioEventLoopGroup();
            worker = new NioEventLoopGroup();
            b = new ServerBootstrap();

			//            group：定义两个线程组，bgroup处理客户端的 Accept事件、workerGroup处理读写事件
			//            channel：绑定NIO服务端通道 NioServerSocketChannel
			//            option：backlog参数指定了等待队列的大小
			//            childHandler：绑定handler，处理读写事件，ChannelInitializer是给通道进行初始化
            b.group(boss, worker)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 1024)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettySerHandler());
                        }
                    });
			//          绑定端口
            ChannelFuture f = b.bind(port).sync();
			//          监听服务器关闭监听
            f.channel().closeFuture().sync();
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            boss.shutdownGracefully();
            worker.shutdownGracefully();
        }
    }

}

class NettySerHandler extends ChannelInboundHandlerAdapter {
	// 处理连接事件
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("客户端接入");
    }
	// 处理断开事件
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("客户端断开");
    }
	// 处理读取事件
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf)msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, "UTF-8");
        System.out.println("From client:"+body);

        ByteBuf res = Unpooled.copiedBuffer("to Client: ok".getBytes());
        ctx.writeAndFlush(res);
    }
	// 处理读取完成事件
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }
	
	// 处理异常事件
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}


class NettyCli {
    public static void main(String[] args) {
        new NettyCli().connect("127.0.0.1", 9132);
    }
    public void connect(String host, int port) {
        EventLoopGroup worker =  null;
        Bootstrap bootstrap = null;
        try {
            worker = new NioEventLoopGroup();
            bootstrap = new Bootstrap();
            bootstrap.group(worker)
                .channel(NioSocketChannel.class)
                .option(ChannelOption.TCP_NODELAY, true)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        socketChannel.pipeline().addLast(new NettyCliHandler());
                    }
                });

            ChannelFuture f = bootstrap.connect(host, port).sync();
            f.channel().closeFuture().sync();

        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            worker.shutdownGracefully();
        }

    }
}

class NettyCliHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("to Server: connect".getBytes()));
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf)msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, "UTF-8");
        System.out.println("From Server:"+body);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

### 打包和部署

基于Netty开发的都是非Web的Java应用，它的打包形态非常简单，就是一个普通的jar包，通常情况下，在正式的商业开发中，我们会使用三种打包方式。

(1) 开发软件提供的导出功能。它可以将指定的Java 或者源码包、代码输出成指定的jar包，它基于下工操作， 在项目模块较多时非常不方便，所以一般不使用这种方式。
(2) 使用ant脚本对工程进行打包。将Netty的应用程序打包成指定的`．jar`	包，一般会输出一个软件安装包： `xxxx_install.gz`
(3) 使用Maven进行上程构建。它可以对校块间的依赖进行管理，支持版本的自动化测试、编译和构建，是目前主流的项目管理工具。


# 三、参考

[Scalable IO in Java](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)