---
title: 面试（二）微服务
date: 2020-05-31 20:37:43
tags: ms
---


注册中心(eureka)：用于微服务的注册
配置中心(Spring Cloud Config分布式配置中心)：用于统一管理微服务
网关（Zuul）：请求路由，转发
业务微服务：实际微服务、基于swagger2的侵入式框架；
Hystrix，熔断，用于底层服务异常时阻止故障传播
Ribbon，负载均衡



***Zuul网关发展***
在单体应用中，网关模块是和应用部署到同一个jvm进程里面的，例如ssm中的springmvc框架。

> Zuul是Netflix开源的一个网关组件，在Netflix内部系统中Zuul被用来作为内部系统的门面

***分布式系统有CP和AP两种模式，请问如何理解这两种模式的优缺点？***
C-Consistency,强一致性；各个节点实时保存同步，新服务必须保证在每一个节点都可以被获取，否则无法使用。
A-Available,可用性。新服务可以先在一个节点使用，直到和其它节点恢复通信，再进行同步。

***Eureka和ZooKeeper都可以提供服务注册与发现的功能,请说说两个的区别***

1.ZooKeeper保证的是CP,Eureka保证的是AP

ZooKeeper在选举期间注册服务瘫痪,虽然服务最终会恢复,但是选举期间不可用的
Eureka各个节点是平等关系,只要有一台Eureka就可以保证服务可用,而查询到的数据并不是最新的

自我保护机制会导致：

    Eureka不再从注册列表移除因长时间没收到心跳而应该过期的服务
    Eureka仍然能够接受新服务的注册和查询请求,但是不会被同步到其他节点(高可用)
    当网络稳定时,当前实例新的注册信息会被同步到其他节点中(最终一致性)

Eureka可以很好的应对因网络故障导致部分节点失去联系的情况,而不会像ZooKeeper一样使得整个注册系统瘫痪

2.ZooKeeper有Leader和Follower角色,Eureka各个节点平等
3.ZooKeeper采用过半数存活原则,Eureka采用自我保护机制解决分区问题
4.Eureka本质上是一个工程,而ZooKeeper只是一个进程

***微服务通信方式***

同步：RPC，REST等.关注下游执行执行结果，用RPC/REST
异步：消息队列。不关注下游执行结果，用MQ，不用RPC/REST


***使用RabbitMQ有什么好处***

解耦 
异步 
削峰

***RabbitMQ 中的 broker 是指什么？cluster 又是指什么？***
broker 是指一个或多个 erlang node 的逻辑分组，且 node 上运行着 RabbitMQ 应用程序。cluster 是在 broker 的基础之上，增加了 node 之间共享元数据的约束。

***RabbitMQ 消息基于什么传输？***
由于TCP连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ使用信道的方式来传输数据。信道是建立在真实的TCP连接内的虚拟连接，且每条TCP连接上的信道数量没有限制。

