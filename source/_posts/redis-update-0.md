---
title: Redis更新了什么（零）总览
date: 2020-10-12 16:00:13
tags: [Redis]
---

Redis 官网文档的版本是Redis2，之后的几个版本进行了不小的修改，具体细节如下。

Redis 2.8 新特性

+	加入了 set 指令的扩展参数，使得 setnx 和 expire 指令可以一起执行，为了解决分布式锁指令原子性的问题
+	scan指令：scan 参数提供了三个参数，第一个是 cursor 整数值，第二个是 key 的正则模式，第三个是遍历的 limit hint。主要用于key扫描，如大key查找。
+	无盘复制（Redis 2.8.18）：用来解决主节点在快照同步时大量IO影响效率的问题。

Redis 3.x 新特性

+	功能更新
	+	新增了地理位置 GEO 模块：GEOHash。主要用于对位置地理信息的操作
+	内部优化
	+	INCR 性能提升
	+	Redis Cluster —— 一个分布式的 Redis 实现
	+	全新的 “embedded string” 对象编码结果，更少的缓存丢失，在特定的工作负载下速度的大幅提升
	+	AOF child -> parent 最终数据传输最小化延迟，通过在 AOF 重写过程中的 “last write”
	+	大幅提升 LRU 近似算法用于键的擦除
	+	WAIT 命令堵塞等待写操作传输到指定数量的从节点
	+	MIGRATE 连接缓存，大幅提升键移植的速度
	+	MIGARTE 新的参数 COPY 和 REPLACE
	+	CLIENT PAUSE 命令：在指定时间内停止处理客户端请求
	+	BITCOUNT 性能提升
	+	CONFIG SET 接受不同单位的内存值，例如 “CONFIG SET maxmemory 1gb”.
	+	Redis 日志格式小调整用于反应实例的角色 (master/slave)


Redis 4.x 新特性
+	支持了 布隆过滤器 以插件的形式加载到 Redis Server 中。主要用于统计去重
+	redis-cell：一个基于漏斗算法的原子性限流模块。主要用于限流
+	MEMORY DOCTOR
+	慢日志记录客户端来源IP地址
+	线程DEL / FLUSH
+	混合RDB + AOF格式
+	更好的复制（PSYNC2）
+	SWAPDB 
+	新的管理命令（活动内存碎片整理）
+	内存使用和性能改进


Redis 5.x 新特性

+	数据类型Stream
+	Timer and Cluster API
+	RDB存储LFU和LRU信息
+	集群管理器从ruby移植到C
+	Sorted Set命令ZPOPMIN/MAX和阻塞变种
+	主动碎片整理方式
+	增强式HyperLogLog实现
+	内存统计报告
+	help子命令
+	断开与连接性能变化
+	错误修复和改进
+	Jemalloc新版本

Redis 6.x 新特性

+	功能更新
	+	ACL 用户权限控制功能
	+	RESP3 新的Redis通信协议
	+	Cluster 管理工具
	+	SSL支持
+	内部优化
	+	IO多线程支持
	+	新的Module API
	+	新的Expire算法
+	外部工具
	+	Redis Cluster Proxy
	+	Disque

