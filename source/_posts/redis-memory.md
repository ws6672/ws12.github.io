---
title: Redis 内存命令
date: 2020-10-17 19:19:06
tags: [Redis]
---


### 一、MEMORY DOCTOR（内存诊断）

1. 简述
命令MEMORY DOCTOR 会列出 Redis 服务器遇到的不同类型的内存相关问题，并提供相应的解决建议


2. 返回值
bulk-string-reply，类似是以下几个类型

|维度|触发条件|建议|
|:--|:--|:--|
|Instance is empty or almost empty|节点内存空间使用小于5M|无|
|Memory peak is much larger|节点在过去使用内存的峰值超过150％的当前内存|导致内存碎片率较大,但这它是无害的，可以手动执行MEMORY PURGE命令或者重启节点|
|High fragmentation|memory_fragmentation 和 RSS memory之和大于1.4|通常是由于工作负载导致分配器大量分段内存造成的，请确保使用的内存分配器是Jemalloc|
|High allocator fragmentation|节点的allocator external fragmentation大于1.1|建议尝试启用"activedefrag"配置选项开启自动内存碎片整理|
|High rss overhead|节点的non-allocator RSS memory开销大于1.1|意味着Redis进程的驻留集大小远大于分配器保存的RSS，通常可能是由于Lua脚本或模块化功能引起的|
|High process rss overhead|节点的RSS memory大于1.1|此通常是由于峰值内存较大,可以尝试使用MEMORY PURGE命令来回收它|
|Slave buffers are too big|节点的output buffers对于每个replica(平均)大于10MB|一些replica正在努力接收数据,要么是因为它太慢,要么是因为网络问题.可以使用INFO输出来检查副本延迟.并使用CLIENT LIST命令检查每个副本的输出缓冲区|
|Client buffers are too big|节点的 clients output buffers大于每个客户端200K(平均)|可能是由于不同的原因造成的,使用CLIENT LIST命令调查问题|
|Script cache has too many|节点中缓存脚超过1000个|定期调用SCRIPT FLUSH命令|



3. 使用

```
127.0.0.1:6379> memory doctor
Hi Sam, this instance is empty or is using very little memory, my issues detector can't be used in these conditions. Please, leave for your mission on Earth and fill it with some data. The new Sam and I will be back to our programming as soon as I finished rebooting.
```


### 二、info memory(内存信息)
会显示 Redis 系统关于存储的统计信息。

例如：
```
127.0.0.1:6379> info memory
# Memory

# 分配的内存(byte)
used_memory:865696
# 分配的内存（Kb）
used_memory_human:845.41K

# 向操作系统申请的内存大小(byte)
used_memory_rss:8892416
# 向操作系统申请的内存大小(MB)
used_memory_rss_human:8.48M

# redis的内存消耗峰值(byte)
used_memory_peak:865696
# redis的内存消耗峰值(KB)
used_memory_peak_human:845.41K
# redis的内存消耗峰值(百分比)
used_memory_peak_perc:100.01%

# Redis维护数据集的内部机制所需的内存开销,
used_memory_overhead:820170
# Redis启动使用的内存
used_memory_startup:802960

# 数据占用的内存
used_memory_dataset:45526
# 数据占用的内存（百分比）
used_memory_dataset_perc:72.57%

# 分配器分配的内存
allocator_allocated:1006000
# 分配器活跃的内存
allocator_active:1228800
# 分配器常驻的内存
allocator_resident:3461120

# 主机内存总量(byte)
total_system_memory:3953950720
# 主机内存总量(GB)
total_system_memory_human:3.68G

# Lua引擎存储占用的内存(byte)
used_memory_lua:37888
# Lua引擎存储占用的内存(KB)
used_memory_lua_human:37.00K
# script脚本占用内存(byte)
used_memory_scripts:0
# script脚本占用内存(B)
used_memory_scripts_human:0B


number_of_cached_scripts:0

# 最大可用内存（byte）
maxmemory:0
# 最大可用内存（B）
maxmemory_human:0B
# 当达到maxmemory时的淘汰策略
maxmemory_policy:noeviction

# 分配器的碎片率
allocator_frag_ratio:1.22
# 分配器的碎片大小
allocator_frag_bytes:222800
# 分配器常驻内存比例
allocator_rss_ratio:2.82
# 分配器的常驻内存大小
allocator_rss_bytes:2232320

# 常驻内存开销比例
rss_overhead_ratio:2.57
# 常驻内存开销大小
rss_overhead_bytes:5431296

# 碎片率(used_memory_rss / used_memory)
mem_fragmentation_ratio:10.78
# 内存碎片大小
mem_fragmentation_bytes:8067736

# 被驱逐的内存
mem_not_counted_for_evict:0

# redis复制积压缓冲区内存
mem_replication_backlog:0

# Redis节点客户端消耗内存
mem_clients_slaves:0
# Redis所有常规客户端消耗内存
mem_clients_normal:16986

# AOF使用内存		
mem_aof_buffer:0

# 内存分配器
mem_allocator:jemalloc-5.1.0
# 活动碎片整理是否处于活动状态(0没有,1正在运行)
active_defrag_running:0
# 0-不存在延迟释放的挂起对象
lazyfree_pending_objects:0
```

### 三、SLOWLOG（慢查询日志）

此命令用于读取和重置Redis慢查询日志。慢日志就是记录了执行速度特别慢的SQL语句。

Redis Slow Log是一个用于记录超过指定执行时间的查询的系统。执行时间不包括与客户端交谈，发送回复等I / O操作，而仅包括实际执行命令所需的时间（这是命令执行的唯一阶段，在该阶段线程被阻塞并且不能同时满足其他要求）。


***配置***

可通过`redis.conf`配置：
```
# 告诉Redis执行命令的时间（以微秒为单位）要超过多少秒才能被记录下来
slowlog-log-slower-than 10000

# 设置慢日志的长度,最小值为零
# 当记录新命令并且慢速日志已经达到最大长度时，最旧的日志将从已记录命令队列中删除，以腾出空间
slowlog-max-len 128
```

可以通过编辑redis.conf或在服务器运行时使用CONFIG GET和CONFIG SET命令来完成配置。

***语法***

```
SLOWLOG subcommand [argument]

# 查看日志
slowlog get COUNT
# 查看日志长度
SLOWLOG LEN
```

1. 查看日志
```

redis 127.0.0.1:6379> slowlog get 2
1) 1) (integer) 14
   2) (integer) 1309448221
   3) (integer) 15
   4) 1) "ping"
2) 1) (integer) 13
   2) (integer) 1309448128
   3) (integer) 30
   4) 1) "slowlog"
      2) "get"
      3) "100"

每个条目都由四个（或从Redis 4.0开始的六个）字段组成：
	每个慢日志条目的唯一累进标识符。
	处理已记录命令的Unix时间戳。
	它执行所需的时间（以微秒为单位）。
	组成命令参数的数组。
	客户端IP地址和端口（仅4.0）。
	客户端名称（如果通过CLIENT SETNAME命令设置）（仅4.0）
```

2. 查看日志长度
```	
127.0.0.1:6379> slowlog len
0
```

