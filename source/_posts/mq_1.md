---
title: RabbitMQ（一）入门
date: 2020-01-05 23:08:41
tags: [mq]
---

# 一、简介

RabbitMQ 是一个热门的消息中间件，在分布式环境下性能高。RabbitMQ 是采用 rlang 语言实现 AMQP (Advanced Message Queuing Protocol ，高级消息队列协议）的消息中间件，它最初起源于金融系统，用于在分布式系统中存储转发消息。


### 基础

+	MQ
	+	基础
		+	MQ(Message Queue) 即消息队列， 是在消息的传输过程中保存消息的容器
		+	消息中间件
		+	把数据放到消息队列叫做生产者
		+	从消息队列里边取数据叫做消费者
	+	功能
		+	消息传递
		+	消息生产者和消息消费者的认证功能（即他们能消费和生产哪些具体的topic）。
	+	MQ优点
		+	削峰/限流 将请求写入队列，避免超过负载
		+	异步
		+	解耦 不需要将服务请求写死在代码中
	+	队列
		+	队列是一种先进先出的数据结构。
	+	需求
		+	在Java中已经有HashMap这种键值对可以存储数据，但redis这种中间件还是有存在的必要的，通过中间件可以实现代码的解耦。
		+	webservice 也可以请求数据，但使用MQ可以异步请求
	+	与WebService的区别
		+	Webservice近乎实时通信，而MQ却通常是延时通信
	+	消息传递模式
		+	发布订阅：向订阅的客户端统一推送数据
		+	点对点：独享通信链路，每多一个客户端就需要有一条链路
	+	架构图
		+	![架构图](/image/mq/mq-1-企业级架构图.png)


### 什么是消息中间件

消息，是指在应用间传送的数据，包括字符串、JSON、对象等。消息中间件（MQ），是指利用高效可靠的消息传递机制进行与平台无关的数据交流，并基于数据通信进行分布式系统的集成。常用的消息中间件有RabbitMQ、Kafka、ActiveMQ、RocketMQ等

消息队列中间件，也可以称为消息队列或者消息中间件。它一般有两种传递模式：点对点( P2P, Point-to-Point ）模式和发布／订阅（ Pub/Sub ）模式。点对点模式是基于队列的，消息生产者发送消息到队列，消息消费者从队列中接收消息，队列的存在使得消息的异步传输成为可能发布订阅模式定义了 如何向 个内容节点发布和订阅消息，这个内容节点称为主题（ topic ），主题可以认为是消息传递的中介，消息发布者将消息发布到某个主题，而消息订阅者则从主题中订阅消息。主题使得消息的订阅者与消息的发布者互相保持独立，不需要进行接触即可保证消息的传递，发布／订阅模式在消息的一对多广播时采用。


### AMQP 协议
+	AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。
+	AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。
+	RabbitMQ就是一个基于AMQP协议的中间件

***工作流程***

发布者生产消息经由交换机，交换机根据路由规则分发给与该交换机绑定的队列， AMQP 代理会将消息投递给订阅了该队列的消费者或由消费者自助获取消息。

![AMQP](/image/mq/mq-amqp.png)


***消息确认（Message Acknowledgements）机制***

当一个消息从队列中投递给消费者后，不会立即从队列中删除，直到它收到来自消费者的确认回执（Acknowledgement）后，才完全从队列中删除。

***处理无法路由的消息***

当一个消息无法被成功路由时（无法从交换机分发到队列），消息或许会被返回给发布者并被丢弃。或者，如果消息代理执行了延期操作，消息会被放入一个所谓的死信队列中。此时，消息发布者可以选择某些参数来处理这些特殊情况。

***注意***

+	发布者、交换机、队列、消费者都可以有多个。同时因为 AMQP 是一个网络协议，所以这个过程中的发布者，消费者，消息代理 可以分别存在于不同的设备上。
+	发布者发布消息时可以给消息指定各种消息属性（Message Meta-data）。有些属性有可能会被消息代理（Brokers）使用，然而其他的属性则是完全不透明的，它们只能被接收消息的应用所使用。


### Erlang/OTP 

***Erlang***
	+	Erlang是一种通用的*面向并发*的编程语言，它由瑞典电信设备制造商爱立信所辖的CS-Lab开发，目的是创造一种可以应对大规模并发活动的编程语言和运行环境。
	+	Erlang非常适合于构建分布式，实时软并行计算系统
	
***OTP设计原则***
OTP设计原则用于指导以进程、模块和目录的形式组织Erlang代码的规范。
![OTP设计原则结构图](/image/mq/mq-2-otp.png)

+	监控树：基于 workers（工人）和 supervisors（监工、监程）的进程组织模型
	+	workers：工作进程
	+	supervisors：重置进程，当监控到 workers 发生异常，该进程会对它进行重启
	+	![otp监控树](/image/mq/OTP-worker.png.png)

+	Behaviours（行为模式）：在监控树中，多个supervisors的结果是类似的，差别在于子进程不同。行为模式就是把通用行为抽象成一个模块，即把进程的代码分成通用的部分（behaviour 模块）和专有的部分（callback module 回调模块）.

+	`RabbitMQ`符合OTP设计原则的行为
	+	gen_server 用于实现 C/S 结构中的服务端。
	+	gen_fsm 用于实现有限状态机。
	+	gen_event 用于实现事件处理功能。
	+	supervisor 用于实现监督树中的督程。
	+	gen_statem 新版本中的有限状态机实现 


>	Erlang语言只是一部分，实际上支撑这个生态/技术栈是EVM（ErlangVM/BEAM）/OTP（标准库）。
	RabbitMQ服务端代码是使用并发式语言Erlang编写的。


### RabbitMQ结构

![几个概念的关系图](/image/mq/mq-2-rabbitmq-gn.png)

***Message***
消息，它由消息头和消息体两部分组成。消息体是不透明的，但消息头是由一些属性组成的，其中包括：
+	routing-key（路由键）
+	priority（优先权）
+	delivery-mode（持久存储）。

***Publisher***
生产者，消息生产者，向交换器推送消息

***Exchange***
交换器，用来接收生产者传递过来的消息，然后将这些消息路由至服务器中的队列

***Binding***
绑定，用于消息队列与交换器之间的沟通。也是消息路由的规则，相当于一个路由表。

***Queue***
消息队列，用来保存消息直到发送给消费者。一个消息可以进入一个或多个队列，除消费者取走消息，否则它一直在消息队列里。

***Connection***
网络连接，如：一个TCP连接

***Channel***
信道，多路复用连接中一个独立的双向数据传输通道。无论是发布消息、订阅队列、接收消息都是通过信道来完成。复用信道是为了降低系统资源的消耗。

***Consumer***
消费者，也就是接收生产者发来的消息的客户端应用。

***Virtual Host***
虚拟主机，交换器、消息队列相关的对象。一个VHOST其实可以看成一个rabbitmp服务器，它拥有自己的队列、交换器、绑定与权限机制等。Rabbitmq默认vhost是/。


# 二、安装


*资源地址*
在以下地址按电脑位数安装对应的版本：
[RabbitMQ 的下载地址](https://www.rabbitmq.com/download.html)
[Erlang 的下载地址](https://www.erlang.org/downloads)

### window安装

1.因为RabbitMQ服务端是由 Erlang编写，所以 首先需要安装Erlang ，默认选项即可
![Erlang安装页面](/image/mq/mq-2-erlInstall.png)

2. 安装 RabbitMQ 中间件
![RabbitMQ安装页面](/image/mq/mq-2-rabbitmqctl.png)

3. 配置Erlang编写的环境变量
```
# 添加环境变量
ERLANG_HOME C:\Program Files\erl10.6
path ;%ERLANG_HOME%\bin;

# cmd 测试
C:\Users\wsz>erl
Eshell V10.6  (abort with ^G)

```
4. 安装RabbitMQ-Plugins插件
```
cd C:\Program Files\RabbitMQ Server\rabbitmq_server-3.8.2\sbin
c:
rabbitmq-plugins enable rabbitmq_management

rabbitmqctl status
```



以下表示安装成功
![表示安装成功](/image/mq/mq-2-rabbitmqctl.png)

5. 启动`/sbin/rabbitmq-server.bat`，访问 `http://localhost:15672/` 会显示下面的网站。默认情况下，访问 RabbitMQ 务的用户名和密码都是“guest ”，这个账户有限制，默认只能通过本地网络（如 localhost ）访问，远程网络访问受限，所以在实现生产和消费消息之前，需要另外添加其它用户，并设置相应的访问权限。

![启动成功](/image/mq/mq-2-rbmq.png)

### Linux 安装

1. 安装 Erlang
```
# 解压
tar zxvf otp_src_19.3.tar.gz 
cd otp_src_19.3
# ./configure --prefix=/opt/erlang

# 安装 Erlang
yum install ncurses-devel
make
make install

# 修改／etc/profile 配置文件，添加下面的环境变量
vim /etc/profile
	ERLANG_HOME=/opt/erlang 
	export PATH=$PATH:$ERLANG_HOME/bin 
	export ERLANG_HOME

# 执行如下命令让配置文件生效
source /etc/profile

# 验证 Erlang 是否安装成功
erl


```

2. 安装RabbitMQ

```
[root@hidden ~]# tar zvxf rabbitmq- server- generic-unix-3.6.10 . tar . gz -C /opt 
[root@hidden ~]# cd /opt 
[root@hidden ~ ]# mv rabbitmq_server-3 . 6.10 rabbitmq

vim /etc/profile
	export PATH=$PATH:/opt/rabbitmq/sbin 
	export RABBITMQ_HOME=/opt/rabbitmq

source /etc/profile
```



### 常用命令


```
rabbitmq-plugins.bat list（查看已安装的插件列表）
rabbitmq-plugins.bat enable rabbitmq_management（开启该插件）

# 服务启动与关闭
rabbitmq-service.bat start
rabbitmq-service.bat stop

# 让 RabbitMQ服务以守护进程的方式在后台运行
rabbitmq-server -detached
```
 
### 应用场景

+	异步处理-解耦
+	广播	订阅发布
+	流量削峰 缓存到消息队列
+	`RabbitMQ的默认访问端口为5672，而管理端口为 15672`

### 集群 

Rabbitmq 基于 Erlang语言编写，具有分布式特点，可以进行集群部署。在集群中，一台机器就是一个节点。
Rabbitmq存在一个虚拟主机的逻辑概念，用于划分服务。相对于一个节点来说，节点里可以根据服务的不同开启不同的虚拟主机。一个虚拟主机可以类比为关系数据库中的一个数据库。

***


# 三、Rabbitmq CLI
 
Rabbitmq提供 rabbitmqctl通过命令行进行操作。rabbitmqctl工具是一个系列的工具，运用这个工具可以执行大部分的 RabbitMQ 的管理操作。

### rabbitmqctl 

rabbitmqctl是用于管理RabbitMQ服务器节点的命令行工具。它通过连接到专用CLI工具通信端口上的目标RabbitMQ节点并使用共享密钥（称为cookie文件）进行身份验证来执行所有操作。


### 查询操作

```
rabbitmqctl list_bindings    #查看绑定信息
rabbitmqctl list_exchanges   #查看交换器
rabbitmqctl list_queues   #查看队列
rabbitmqctl list_channels	#查看通道
rabbitmqctl list_consumers	#查看消费者
```

### 虚拟主机配置


***语法***

```
创建新的虚拟主机 
rabbitmqctl add_vhost【主机name】

删除虚拟主机
rabbitmqctl delete_vhost【主机name】

列出所有的虚拟主机
rabbitmqctl list_vhosts 


-- 查看trace功能是否开启
rabbitmqctl list_vhosts name tracing
```

***实例***

```
-- 创建
rabbitmqctl add_vhost test

-- 列出
rabbitmqctl list_vhosts 

	name
	/ --默认虚拟主机
	test
 

```

### 用户配置

***用户类别***

|角色|登陆控制台|查看节点|操作策略|操作用户|
|:--|:--|:--|:--|:--|
|超级管理员(administrator)：拥有所有的权限|Y|Y|Y|Y|
|监控者(monitoring)：管理节点|Y|Y|N|N|
|策略制定者(policymaker)：管理策略|Y|N|Y|N|
|普通管理者(management)：登陆管理台|Y|N|N|N|
|生产者和消费者：无法登陆服务器控制台|N|N|N|N|

***语法***
```
-- 创建用户
rabbitmqctl add_user 【name】【password】

-- 列出所有用户
rabbitmqctl list_users

-- 更改用户密码
rabbitmqctl change_password 【name】【password】

-- 清除用户密码
rabbitmqctl clear_password 【name】

-- 删除用户
rabbitmqctl delete_user 【name】 删除user1用户

-- 赋予用户某个角色
rabbitmqctl set_user_tags 【name】【角色 administrator/monitoring/policymaker/management】 

-- 设置权限（配置权限、发布者权限、消费者权限，正则表达式）
rabbitmqctl set_permissions [-p <vhost>]【用户名】【conf】【write】【 read】 
	conf 队列和交换机的创建和删除
	write 发布消息
	read 有关消息的任何操作，包括清除这个队列 

正则实例：
	"."：匹配任何队列和交换器
	"checks-."/"^checks-."：只匹配checks-开头的队列和交换器
	""：不匹配队列和交换器
	 

➜  ~ rabbitmqctl add_user root root

-- 列出某用户的权限
rabbitmqctl list_user_permissions 【用户名】 

-- 列出指定虚拟主机下所有用户的权限
rabbitmqctl list_permissions -p 【主机名】

-- 清除某用户在指定虚拟机上的授权
rabbitmqctl clear_permissions -p 【主机名】 【用户名】

```


***实例***

```
-- 创建用户
rabbitmqctl add_user mn mn
rabbitmqctl add_user pm pm
rabbitmqctl add_user mm mm
rabbitmqctl add_user test test
rabbitmqctl add_user test2 test2

-- 设置用户角色
rabbitmqctl set_user_tags mn monitoring
rabbitmqctl set_user_tags pm policymaker
rabbitmqctl set_user_tags mm management
rabbitmqctl set_user_tags test none
rabbitmqctl set_user_tags test2 none

-- 列出用户-角色
rabbitmqctl list_users
user    tags
mn      [monitoring]
mm      [management]
guest   [administrator]
pm      [policymaker]

-- 创建虚拟主机
rabbitmqctl add_vhost test

-- 用户赋权
rabbitmqctl set_permissions -p / mn ".*" ".*" ".*"
rabbitmqctl set_permissions -p / mm ".*" ".*" ".*"
rabbitmqctl set_permissions -p / pm ".*" ".*" ".*"
rabbitmqctl set_permissions -p test test ".*" ".*" ".*"
rabbitmqctl set_permissions -p test test2 ".*" ".*" ".*"

-- 列出指定虚拟主机下所有用户的权限
rabbitmqctl list_permissions -p /
```

### 节点配置

rabbitmqctl 工具的标准语法如下 :
`rabbitmqctl [ -n node] [-t timeout] [-q) {command) [command options ... ]`
+	-n 格式是rabbit@servername，默认节点是“rabbit hostname“，此处的 hostname是主机名称。
+	-t 超时时间（秒）
+	-q 宁静(quiet)模式 

***实例***
查看主机为sky，等待10s：
`rabbitmqctl -n rabbit@sky -t 10 list_users`

### RabbitMQ management 插件

RabbitMQ management 插件同样是由 Erlang 语言编写的，并且和 RabbitMQ 服务运行在同一个Erlang 虚拟机中。
RabbitMQ management 插件可以提供 Web 管理界面用来管理如前面所述的虚拟主机、用户等，也可以用来管理队列、交换器、绑定关系、策略、 参数等，还可以用来监控 RabbitMQ服务的状态及一些数据统计类信息，可谓是功能强大，基本上能够涵盖所有 RabbitMQ 管理的
功能。

在最新版本中，该插件默认开启，可以通过[ip地址](http://localhost:15672/#/)访问

![rabbitmq-management](/image/mq/rabbitmq-management.png)

***


# 四、在Java中的使用


**通过包导入配置**

[RabbitMQ Jar包下载地址](http://repo1.maven.org/maven2/com/rabbitmq/amqp-client/)

**maven配置**
```
<!-- https://mvnrepository.com/artifact/com.rabbitmq/amqp-client -->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.8.0</version>
</dependency>
```

***实例***

生产者：
```
package com.wsz.tool.rabbitmq.base;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author zws
 * @date 2020/12/23 14:55
 * @Description TODO
 */
public class RabbitProducer {
    private static final String EXCHANGE_NAME ="exchange_demo";
    private static final String ROUTING_KEY = "routingkey_demo";
    private static final String QUEUE_NAME ="queue_demo";
    private static final String IP_ADDRESS ="127.0.0.1";
    private static final int PORT= 5672;//RabbitMQ 服务端默认端口号为 5672
    public static  void main(String[] args) throws IOException,
            TimeoutException, InterruptedException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(IP_ADDRESS);
        factory.setPort(PORT);
        factory.setUsername("guest");
        factory.setPassword("guest");
        Connection connection = factory.newConnection();//创建连接
        Channel channel = connection.createChannel(); //创建信道

        //创建一个 type direct 、持久化的、非自动删除的交换器
        channel.exchangeDeclare(EXCHANGE_NAME, "direct", true, false, null);
        //创建一个持久化、非排他的、非自动删除的队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        //将交换器与队列通过路由键绑定
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
        //发送一条持久 的消息 hello worl d !
        String message = "Hello World !";
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
                MessageProperties.PERSISTENT_TEXT_PLAIN,
                message.getBytes());
        //关闭资源
        channel.close();
        connection.close();
    }
}


```

消费者：
```
package com.wsz.tool.rabbitmq.base;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

/**
 * @author zws
 * @date 2020/12/23 14:55
 * @Description TODO
 */
public class RabbitConsumer {
    private static final String QUEUE_NAME = "queue_demo";
    private static final String IP_ADDRESS = "127.0.0.1";
    private static final int PORT = 5672;//RabbitMQ 服务端默认端口号为 5672

    public static void main(String[] args) throws IOException,
            TimeoutException, InterruptedException {

        Address[] addresses = new Address[]{
                new Address(IP_ADDRESS, PORT)
        };
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("guest");
        factory.setPassword("guest");
        //这里的连接方式与生产者的 demo 略有不同，注意辨别区别
        Connection connection = factory.newConnection(addresses); //创建连接
        final Channel channel = connection.createChannel(); //创建信道

        channel.basicQos(64); //设置客户端最多接收未被 ack 的消息的个数
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                try {
                    System.out.println("message :" + new String(body));
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                channel.basicAck(envelope.getDeliveryTag(), false);
            }

        };

        channel.basicConsume(QUEUE_NAME, consumer);
        //等待回调函数执行完毕之后 关闭资源
        TimeUnit.SECONDS.sleep(1);
        channel.close();
        connection.close();

    }
}

```



***

# 五、参考文章
> [rabbitMQ和对应的erlang版本匹配](https://www.cnblogs.com/gne-hwz/p/10714013.html)
[windows10环境下的RabbitMQ安装步骤（图文）](https://www.cnblogs.com/saryli/p/9729591.html)
[RabbitMQ windows安装（一 ）](https://www.cnblogs.com/lfalex0831/p/8951955.html)
[rabbitmq基本原理以及常用命令rabbitmqctl](https://blog.csdn.net/qq_28513801/article/details/90641238)
[MQ的几种消息传递方式](https://www.cnblogs.com/panchanggui/p/10265703.html)
[深入理解消息中间件技术之RabbitMQ服务](https://blog.csdn.net/mingongge/article/details/81132632)

