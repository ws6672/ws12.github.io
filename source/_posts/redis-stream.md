---
title: redis Stream（消息队列）
date: 2020-10-16 23:43:51
tags: [Redis]
---

Redis Stream 是 Redis 5.0 版本新增加的数据结构, 主要用于构建消息队列（MQ，Message Queue）。

### 一、入门

在旧版的Redis中，可以通过发布订阅 (pub/sub) 实现消息队列的功能。但是，通过发布订阅功能构建的消息队列无法持久化数据，如果遇到宕机之类的情况消息就会被丢弃。而 Redis Stream 提供了消息的持久化和主备复制功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。


***消费者组***

消费者组最早是由名为Kafka（TM）的流行消息系统引入的。Redis用完全不同的术语重新实现了一个相似的概念，但目标是相同的：允许一组客户端相互配合来消费同一个Stream的不同部分的消息。

如果没有消费者组，仅使用XREAD，所有客户端都将获得所有到达流的条目。在消费者组中，给定的消费者（即从流中消费消息的客户端）必须使用唯一的消费者名称进行标识。该名称只是一个字符串。消费者组的保证之一是，给定的消费者只能看到发送给它的历史消息，因此每条消息只有一个所有者。

然而，还有一个特殊的特性叫做消息认领，其允许其他消费者在某些消费者无法恢复时认领消息。为了实现这样的语义，消费者组要求消费者使用XACK命令显式确认已成功处理的消息。


#### 数据结构

Stream 是基于 RadixTree 数据结构实现的。基数树也叫基数特里树 radix trie 或压缩前缀树 compact prefix tree，是一种更节省空间的 Trie Tree 前缀树。若节点为唯一子节点则与其父节点合并。结构如下：
![Stream的数据结构](/image/redis/redis-stream-struct.png)

每个 Stream 都有唯一的名称，它就是 Redis 的 key，在我们首次使用 xadd 指令追加消息时自动创建。
+	Consumer Group ：消费组，使用 XGROUP CREATE 命令创建，一个消费组有多个消费者(Consumer)。
+	last_delivered_id ：游标，每个消费组会有个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。
+	pending_ids ：消费者(Consumer)的状态变量，作用是维护消费者的未确认的 id。 pending_ids 记录了当前已经被客户端读取的消息，但是还没有 ack (Acknowledge character：确认字符）


### 二、使用

***消息队列命令***

+	XADD - 添加消息到末尾
+	XTRIM - 对流进行修剪，限制长度
+	XDEL - 删除消息
+	XLEN - 获取流包含的元素数量，即消息长度
+	XRANGE - 获取消息列表，会自动过滤已经删除的消息
+	XREVRANGE - 反向获取消息列表，ID 从大到小
+	XREAD - 以阻塞或非阻塞方式获取消息列表

***消费者组命令***

消费者组：
+	XGROUP CREATE - 创建消费者组
+	XGROUP SETID - 为消费者组设置新的最后递送消息ID
+	XGROUP DELCONSUMER - 删除消费者
+	XGROUP DESTROY - 删除消费者组

消息信息：
+	XINFO - 查看流和消费者组的相关信息；
+	XINFO GROUPS - 打印消费者组的信息；
+	XINFO STREAM - 打印流信息

消费处理流程：
+	XREADGROUP GROUP - 读取消费者组中的消息
+	XPENDING - 显示待处理消息的相关信息
+	XCLAIM - 转移消息的归属权
+	XACK - 将消息标记为"已处理"


#### 消息队列命令

1. XADD
用于向队列添加消息，如果指定队列不存在，则创建一个队列。语法格式：
```
XADD key ID field value [field value ...] 
	key：队列名称
	ID：消息id，使用`*`表示由系统自动生成
	field value：记录

```

2. XLEN
获取消息长度。语法格式：
```
XLEN key
```

3. XTRIM
使用 XTRIM 对流进行修剪，限制长度， 语法格式：
```
XTRIM key MAXLEN [~] count
key ：队列名称
MAXLEN ：长度
[~]: 可选项，表示修剪后数量不少于count个即可
count ：数量
```

4. XDEL
从指定流中移除指定的条目，并返回成功删除的条目的数量，在传递的ID不存在的情况下， 返回的数量可能与传递的ID数量不同。语法格式：
```
XDEL key ID [ID ...]

Redis流以一种使其内存高效的方式表示：使用基数树来索引包含线性数十个Stream条目的宏节点。当从Stream中删除一个条目的时候，条目并没有真正被驱逐，只是被标记为删除。
```

5. XRANGE
此命令返回流中满足给定ID范围的条目。范围由最小和最大ID指定。所有ID在指定的两个ID之间或与其中一个ID相等（闭合区间）的条目将会被返回。语法格式：
```
XRANGE key start end [COUNT count]
XRANGE命令有许多用途：
	返回特定时间范围的项目。这是可能的，因为流的ID与时间相关。
	增量迭代流，每次迭代只返回几个项目。但它在语义上比SCAN函数族强大很多。
	从流中获取单个条目，提供要获取两次的条目的ID：作为查询间隔的开始和结束。

该命令还有一个倒序命令，以相反的顺序返回项目，叫做 XREVRANGE，除了返回顺序相反以外，它们是完全相同的。特殊ID-和+分别表示流中可能的最小ID和最大ID
```

6. XREAD

从一个或者多个流中读取数据，仅返回ID大于调用者报告的最后接收ID的条目。此命令有一个阻塞选项，用于等待可用的项目，类似于BRPOP或者BZPOPMIN等等。使用 XREAD 以阻塞或非阻塞方式获取消息列表 ，语法格式：

```
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]
	count ：数量
	milliseconds ：可选，阻塞毫秒数，没有设置就是非阻塞模式
	key ：队列名
	id ：消息 ID
```

该命令返回一个结果数组：返回数组的每个元素都是一个由两个元素组成的数组（键名和为该键报告的条目）。报告的条目是完整的流条目，具有ID以及所有字段和值的列表。返回的条目及其字段和值的顺序与使用XADD添加它们的顺序完全一致。

当使用BLOCK时，超时时将返回一个空回复（nil）。

***实例***

1. 消息ID的序列化生成
```
127.0.0.1:6379> xadd ms * name mk age 23
"1602855570665-0"
127.0.0.1:6379> xadd ms * name bb age 2
"1602855577557-0"
127.0.0.1:6379> xadd ms * name ct age 43
"1602855589097-0"
127.0.0.1:6379> xadd ms * name yy age 88
"1602855597238-0"
```

XADD生成的 （1602763951440-0），就是Redis生成的消息ID，由两部分组成:时间戳-序号。时间戳是毫秒级单位，是生成消息的Redis服务器时间，它是个64位整型（int64）。序号是在这个毫秒时间点内的消息序号，它也是个64位整型。

2. 消息遍历

```
127.0.0.1:6379> XRANGE mystream - +
1) 1) "1602763789606-0"
   2) 1) "name"
      2) "pt"
      3) "age"
      4) "12"
      5) "sex"
      6) "boy"
2) 1) "1602763898101-0"
   2) 1) "name"
      2) "marry"
      3) "age"
      4) "21"
      5) "sex"
      6) "girl"
```



3. 消息的长度
```
127.0.0.1:6379> xlen ms
(integer) 4
```

4. 消息裁剪
```
127.0.0.1:6379> xtrim ms MAXLEN 1
(integer) 1
```


5. 读取消息

```
# ID（0-0）后的消息，即消息队列中第一个消息
127.0.0.1:6379> xread count 1 streams ms 0-0
1) 1) "ms"
   2) 1) 1) "1602855570665-0"
         2) 1) "name"
            2) "mk"
            3) "age"
            4) "23"
			
# ID（1602855570665-0）下一个消息
127.0.0.1:6379> xread count 1 streams ms 1602855570665-0
1) 1) "ms"
   2) 1) 1) "1602855577557-0"
         2) 1) "name"
            2) "bb"
            3) "age"
            4) "2"

# 阻塞消息
127.0.0.1:6379> xread block 1000 count 1 streams ms 0-0
1) 1) "ms"
   2) 1) 1) "1602855570665-0"
         2) 1) "name"
            2) "mk"
            3) "age"
            4) "23"

# $符号用于获取最新的消息，应该仅在第一次调用XREAD时使用。后面的ID你应该使用前一次报告的项目中最后一项的ID，否则你将会丢失所有添加到这中间的条目。
xread block 1000 count 1 streams ms $
```

#### 消费者组命令

1. XGROUP
消费者组相关命令，语法格式：
```
# 创建命令
XGROUP [CREATE key groupname id-or-$]
# 将消费者组的最后交付ID设置为其他内容
XGROUP [SETID key groupname id-or-$]
# 销毁消费者组
XGROUP [DESTROY key groupname]
# 移除指定的消费者
XGROUP [DELCONSUMER key groupname consumername]
```


2. XINFO
这是一个内省命令，用于检索关于流和关联的消费者组的不同的信息

```
XINFO [CONSUMERS key groupname] 
XINFO [GROUPS key] 
XINFO [STREAM key] 
XINFO [HELP]

消费组中消费者列表
XINFO CONSUMERS <key> <group>

获得与流关联的所有消费者组的输出
XINFO GROUPS <key>

存储在特定键的流的一般信息
XINFO STREAM <key>
```

3. XREADGROUP
XREADGROUP命令是XREAD命令的特殊版本，支持消费者组，用于消费者从消息队列获取信息。通过消费者组从流中获取数据，而不是确认这些数据，具有创建待处理条目的效果。

```
XREADGROUP GROUP groupname consumername [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]
	groupname：消费者组
	consumername：消费者，在第一次使用时会自动的为消费者组添加消费者
	COUNT：消息数量
	BLOCK：显示定义，则是阻塞的；参数是毫秒
	STREAMS key [key ...]：流名称
	ID [ID ...]：消息ID
```

3. XPENDING

显示待处理消息的相关信息
```
XPENDING key groupname [start end count] [consumer]
key：流名称
groupname：消费者组
[start end count]：消息范围，例如“- + 10”
[consumer]：消费者名称
```

4. XCLAIM
将待处理消息的所有人转移.

> 请注意，消息只有在其空闲时间大于我们通过XCLAIM指定的空闲时间的时才会被认领。 因为作为一个副作用，XCLAIM也会重置消息的空闲时间（因为这是处理消息的一次新尝试）， 两个试图同时认领消息的消费者将永远不会成功：只有一个消费者能成功认领消息。 这避免了我们用微不足道的.

```
XCLAIM key group consumer min-idle-time ID [ID ...] [IDLE ms] [TIME ms-unix-time] [RETRYCOUNT count] [FORCE] [JUSTID]
	IDLE <ms>: 设置消息的空闲时间
	TIME <ms-unix-time>: 这个命令与IDLE相同，但它不是设置相对的毫秒数，而是将空闲时间设置为一个指定的Unix时间
	RETRYCOUNT <count>: 将重试计数器设置为指定的值
	FORCE: 在待处理条目列表（PEL）中创建待处理消息条目
	JUSTID: 只返回成功认领的消息ID数组，不返回实际的消息。
```


5. XACK
会立即从待处理条目列表（PEL）中移除待处理条目，因为一旦消息被成功处理，消费者组就不再需要跟踪它并记住消息的当前所有者。
XACK命令用于从流的消费者组的待处理条目列表（简称PEL）中删除一条或多条消息。一旦消费者成功地处理完一条消息，它应该调用XACK，这样这个消息就不会被再次处理.
```
XACK key group ID [ID ...]
返回值:
	integer-reply：
	该命令返回成功确认的消息数。 某些消息ID可能不再是PEL的一部分（例如因为它们已经被确认）， 而且XACK不会把他们算到成功确认的数量中。
```

***实例***

1. 消息的分组消费(创建组)
```
127.0.0.1:6379> xgroup create ms mg1 $
OK
```

`$`这表示流中最后一项的ID.

1.2 重新设置流ID
```
127.0.0.1:6379> xgroup setid ms mg1 0
OK
```
1.3 销毁消费者组
```
127.0.0.1:6379> xgroup destroy ms mg1
(integer) 1
```

2. 获取流、消费者组、消费者的信息（消息队列监控）
```
127.0.0.1:6379> xinfo stream ms
 1) "length"
 2) (integer) 4
 3) "radix-tree-keys"
 4) (integer) 1
 5) "radix-tree-nodes"
 6) (integer) 2
 7) "last-generated-id"
 8) "1602855597238-0"
 9) "groups"
10) (integer) 1
11) "first-entry"
12) 1) "1602855570665-0"
    2) 1) "name"
       2) "mk"
       3) "age"
       4) "23"
13) "last-entry"
14) 1) "1602855597238-0"
    2) 1) "name"
       2) "yy"
       3) "age"
       4) "88"
127.0.0.1:6379> xinfo groups ms
1) 1) "name"
   2) "mg1"
   3) "consumers"
   4) (integer) 0
   5) "pending"
   6) (integer) 0
   7) "last-delivered-id"
   8) "0-0"
127.0.0.1:6379> xinfo consumers ms mg1
(empty array)
```

3. 通过消费者组从流读取消息
```
127.0.0.1:6379> xreadgroup group  mg1 com1  count 2 streams ms >
1) 1) "ms"
   2) 1) 1) "1602855570665-0"
         2) 1) "name"
            2) "mk"
            3) "age"
            4) "23"
      2) 1) "1602855577557-0"
         2) 1) "name"
            2) "bb"
            3) "age"
            4) "2"
```

4. 显示待处理消息
```
127.0.0.1:6379> xpending ms mg1 - + 10 com1
1) 1) "1602855570665-0"
   2) "com1"
   3) (integer) 255408
   4) (integer) 1
2) 1) "1602855577557-0"
   2) "com1"
   3) (integer) 255408
   4) (integer) 1
```

5. 未完成消息的处理(转移所有者)
```
127.0.0.1:6379> xclaim ms mg1 com2 30 1602855570665-0
1) 1) "1602855570665-0"
   2) 1) "name"
      2) "mk"
      3) "age"
      4) "23"
127.0.0.1:6379> xpending ms mg1 - + 10 com1
1) 1) "1602855577557-0"
   2) "com1"
   3) (integer) 801202
   4) (integer) 1
```

6. 确认消息
```
127.0.0.1:6379> xack ms mg1 1602855577557-0
(integer) 1
```

***死信问题***

死信，对应的单词为“Dead Letter”。如果某个消息，不能被消费者处理，也就是不能被XACK，这是要长时间处于Pending列表中，即使被反复的转移给各个消费者也是如此。此时该消息的delivery counter就会累加（pending）,当到临界时，使用`XDEL`删除即可。

但是，并没有删除Pending中的消息因此你查看Pending，消息还会在。可以执行XACK标识其处理完毕！


