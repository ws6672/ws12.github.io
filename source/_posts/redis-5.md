---
title: redis（五）集群
date: 2020-10-04 13:29:19
tags: [Redis]
---

# 基础

***集群、分布式、SOA、微服务***

集群：同一个模块部署在多个服务器上，在不同的服务器中会运行相同代码，通过负载均衡提供服务
分布式：一个业务分拆多个子业务，不同服务器运行不同模块，解决高并发问题；服务层和Web层 是同一个工程，耦合严重
SOA：服务层和表现层 分成两个子项目，没有耦合关系；表现层可以用任意一个服务层，相互独立

单机、集群、分布式、SOA的区别如下：
```
存在模块A、B、C；模块A可以分为表现层A1、服务层A2

+	单机模式
	+	A+B+C
+	集群（每个服务器运行相同的代码）
	+	服务器1（A+B+C）
	+	服务器2（A+B+C）
	+	服务器3（A+B+C）
+	分布式（每个服务器运行不同的模块）
	+	服务器1（A）
	+	服务器2（B）
	+	服务器3（C）
+	SOA
	+	表现层
		+	服务器1（A1）
		+	服务器2（B1）
		+	服务器3（C1）
	+	服务层
		+	服务器4（A2）
		+	服务器5（B2）
		+	服务器6（C2）
```
		
SOA和微服务的主要区别：
+	SOA强调按水平架构划分为：前、后端、数据库、测试等；微服务强调按垂直架构划分，按业务能力划分，每个服务完成一种特定的功能，服务即产品（前端、后端、数据库、测试都是产品）
+	微服务剔除SOA中复杂的ESB企业服务总线，所有的业务智能逻辑在服务内部处理，使用Http（Rest API）进行轻量化通讯
+	SOA架构强调的是异构系统之间的通信和解耦合；微服务架构强调的是系统按业务边界做细粒度的拆分和部署


***什么是 Redis 集群***
+	Redis 集群是一个分布式（distributed）、容错（fault-tolerant）的 Redis 实现， 集群可以使用的功能是普通单机 Redis 所能使用的功能的一个子集（subset）。

+	Redis 集群中不存在中心（central）节点或者代理（proxy）节点， 集群的其中一个主要设计目标是达到线性可扩展性（linear scalability）。

+	Redis 集群为了保证一致性（consistency）而牺牲了一部分容错性： 系统会在保证对网络断线（net split）和节点失效（node failure）具有有限（limited）抵抗力的前提下， 尽可能地保持数据的一致性。

集群的容错功能是通过使用主节点（master）和从节点（slave）两种角色（role）的节点（node）来实现的：

+	主节点和从节点使用完全相同的服务器实现， 它们的功能（functionally）也完全一样， 但从节点通常仅用于替换失效的主节点。
+	如果不需要保证“先写入，后读取”操作的一致性（read-after-write consistency）， 那么可以使用从节点来执行只读查询。

***功能子集***
Redis 集群实现了单机 Redis 中， 所有处理单个数据库键的命令。

用户也许可以通过 MIGRATE COPY 命令， 在集群的计算节点（computation node）中执行针对多个数据库键的只读操作， 但集群本身不会去实现那些需要将多个数据库键在多个节点中移来移去的复杂多键命令。

Redis 集群不像单机 Redis 那样支持多数据库功能， 集群只使用默认的 0 号数据库， 并且不能使用 SELECT index 命令。


***Redis 集群协议中的客户端和服务器***

节点的职责：
+	保存键值对数据
+	记录集群的状态
+	自动发现其他节点，识别工作不正常的节点，并在有需要时，在从节点中选举出新的主节点。

为了执行以上列出的任务， 集群中的每个节点都与其他节点建立起了“集群连接（cluster bus）”， 该连接是一个 TCP 连接， 使用二进制协议进行通讯。

节点之间使用 Gossip 协议 来进行以下工作：
+	传播（propagate）关于集群的信息，以此来发现新的节点。
+	向其他节点发送 PING 数据包，以此来检查目标节点是否正常运作。
+	在特定事件发生时，发送集群信息。
+	在集群中发布或订阅信息

***键分布模型***

Redis 集群的键空间被分割为 16384 个槽（slot）， 集群的最大节点数量也是 16384 个。每个主节点都负责处理 16384 个哈希槽的其中一部分。当集群处于“稳定”（stable）状态时，指的是集群没有在执行重配置（reconfiguration）操作， 每个哈希槽都只由一个节点进行处理

键映射到slot的算法如下：
```
HASH_SLOT = CRC16(key) mod 16384
```

> 推荐的最大节点数量为 1000 个左右。
重配置指的是将某个/某些槽从一个节点移动到另一个节点。
一个主节点可以有任意多个从节点， 这些从节点用于在主节点发生网络断线或者节点失效时， 对主节点进行替换（主从复制）。



### conf

```
proto-max-bulk-len 512mb


# 包含其它配置文件
include /path/to/local.conf
# 配置其它模块
loadmodule /path/to/my_module.so

# 绑定端口
bind 127.0.0.1
# redis 端口
port 6379

# Linux内核为每个TCP服务器程序维护两条backlog队列，一条是TCP层的未连接队列，一条是应用层的已连接队列，分别对应net.ipv4.tcp_max_syn_backlog 和net.core.somaxconn 两个内核参数。
# 此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。建议值：2048
tcp-backlog 511

# Redis 通过 unixsocket 连接
# 指定redis监听的unix socket 路径
unixsocket /tmp/redis.sock
# unixsocket 文件权限
unixsocketperm 700

# Redis服务端的连接超时时间
timeout 0

# redis服务器与客户端保活参数，用来定时向client发送tcp_ack包来探测client是否存活（单位秒）
tcp-keepalive 300


```


> [Redis 集群规范](http://redisdoc.com/topic/cluster-spec.html)