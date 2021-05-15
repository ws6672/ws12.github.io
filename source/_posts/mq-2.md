---
title: RabbitMQ（二）
date: 2020-12-23 15:31:17
tags: [mq]
---

### 基本概念
 
+	Broker： 消息队列服务器的实体，大多数情况下也可以将 `RabbitMQ Broker` 看作消息队列的服务器
+	message（消息）
	+	routing-key（路由键）:路由关键字，exchange根据这个关键字进行消息投递
	+	priority（优先权）
	+	delivery-mode（持久存储）。
+	*Exchange（消息交换机）*：它指定消息按什么规则，路由到哪个队列
	+	交换机模式
		+	fanout 推送所有消息到所有与它绑定的队列
		+	direct 推送到那些路由键完全匹配的队列中
		+	topic（主题匹配）模糊匹配路由键
			+	'\*' 可以代替一个单词。
			+	'\#' 可以替代零个或多个单词。
			+	```
				\*.aa.\*, 可以匹配 b.aa.b,不可以匹配 b.b.aa.b

				\#.aa.\*, 可以匹配 b.b.aa.b,不可以匹配 b.aa.b.b
			```
		+	headers 不依赖于routing,而是根据消息头来匹配
+	*Binding*：绑定，它的作用就是把exchange和queue按照路由规则绑定起来
+	Queue（消息队列）：消息队列载体，每个消息都会被投入到一个或多个队列
+	vhost（虚拟消息队列）：`vhost`是`AMQP` 概念的基础，客户端在连接的时候必须制定一个 vhost RabbitMQ 默认创
建的 vhost 为 '/', 逻辑消息队列
	+	用户权限分离
	+	逻辑隔离
+	producer（消息生产者）：就是投递消息的程序
+	consumer（消息消费者）：，就是接受消息的程序
+	connection（连接）：一个网络连接，例如TCP套接字
	+	channel（消息通道）：在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务

***消息队列的运转过程***

![mq-run-param](/image/mq/mqrun.png)

首先生产者将业务方数据进行可能的包装，之后封装成消息，发送（AMQP 协议里这个动作对应的命令为 Basic.Publish）到 Broker中。
消费者订阅并接收消息 （AMQP 协议里这个动作对应的命令为 Basic.Consume 或者 Basic.Get ），经过可能的解包处理得到原始的数据，之后再进行业务处理逻辑。

这个业务处理逻辑并不一定需要和接收消息的逻辑使用同一个线程，消费者进程可以使用一个线程去接收消息，存入到内存中，比如使用 Java 中的 BlockingQueue.业务处理逻辑使用另一个线程从内存中读取数据，这样可以将应用进一步解稿，提高整个应用
的处理效率。



***信息传递的过程***

![信息传递的过程](/image/mq/mq-2-messgeTo.png)
*prefetchCount（权重）* 
![prefetchCount](/image/mq/mq-2-prefetchCount.png)



