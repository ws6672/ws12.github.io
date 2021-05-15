---
title: redis（六） 进阶命令
date: 2020-10-11 23:23:42
tags: [Redis]
---

### 一、数据库命令（database）

***EXISTS***

1. `EXISTS key`
2. 检查给定 key 是否存在。
3. 返回值：若 key 存在，返回 1 ，否则返回 0 。

```
> set i 1
> exists i
(integer) 1
```

***TYPE***

1. `TYPE key`
2. 返回 key 所储存的值的类型
3. 返回值：
+	none (key不存在)
+	string (字符串)
+	list (列表)
+	set (集合)
+	zset (有序集)
+	hash (哈希表)
+	stream （流）

```
> set name petter
OK
> type name
"string"
> type tt
"none"
```

***RENAME***

1. `RENAME key newkey`
2. 将 key 改名为 newkey；当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。当 newkey 已经存在时， RENAME 命令将覆盖旧值。
3. 返回值：改名成功时提示 OK ，失败时候返回一个错误。

```
> get y
(nil)
> set t 100
OK
> rename t y
OK
> get y
"100"
```

***RENAMENX***

1. `RENAMENX key newkey`
2. 当且仅当 newkey 不存在时，将 key 改名为 newkey。当 key 不存在时，返回一个错误
3. 返回值：修改成功时，返回 1 ； 如果 newkey 已经存在，返回 0 。

```
> set i 100
OK
> set j 200
OK
> renamenx i j
(integer) 0 # 新值j存在，
> renamenx i k
(integer) 1 # 不存在，修改成功返回1
```

***MOVE***

1. `MOVE key db`
2. 将当前数据库的 key 移动到给定的数据库 db 当中。如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。
3. 返回值：移动成功返回 1 ，失败则返回 0 。

```
set i 1
move i 1 # 把i移动到数据库1
```

***DEL***

1. `DEL key [key …]`
2. 删除给定的一个或多个 key 。不存在的 key 会被忽略。
3. 返回值：被删除 key 的数量。

```
> set i 100
OK
> set j 200
OK
> del i j
(integer) 2
```

***RANDOMKEY***

1. `RANDOMKEY`
2. 从当前数据库中随机返回(不删除)一个 key
3. 返回值：当数据库不为空时，返回一个 key 。 当数据库为空时，返回 nil

```
set j 100
set i 100
RANDOMKEY
i
```

***DBSIZE***

1. `DBSIZE`
2. 返回当前数据库的 key 的数量。
3. 返回值：当前数据库的 key 的数量。

```
redis> DBSIZE
(integer) 5

redis> SET new_key "hello_moto"     # 增加一个 key 试试
OK

redis> DBSIZE
(integer) 6
```

***KEYS***

1. `KEYS pattern`
2. 查找所有符合给定模式 pattern 的 key，例如：
+	`KEYS * 匹配数据库中所有 key `
+	`KEYS h?llo 匹配 hello ， hallo 和 hxllo 等`
+	`KEYS h*llo 匹配 hllo 和 heeeeello 等`
+	`KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo`
3. 返回值：符合给定模式的 key 列表。

```
> keys *
1) "k"
2) "j"
3) "name"
4) "y"
5) "i"
```

***SCAN***

1. `SCAN cursor [MATCH pattern] [COUNT count]`
2. SCAN 命令及其相关的 SSCAN 命令、 HSCAN 命令和 ZSCAN 命令都用于增量地迭代（incrementally iterate）一集元素（a collection of elements）：
+	SCAN 命令用于迭代当前数据库中的数据库键。
+	SSCAN 命令用于迭代集合键中的元素。
+	HSCAN 命令用于迭代哈希键中的键值对。
+	ZSCAN 命令用于迭代有序集合中的元素（包括元素成员和元素分值）。
3. 返回值：
SCAN 命令、 SSCAN 命令、 HSCAN 命令和 ZSCAN 命令都返回一个包含两个元素的 multi-bulk 回复： 回复的第一个元素是字符串表示的无符号 64 位整数（游标）， 回复的第二个元素是另一个 multi-bulk 回复， 这个 multi-bulk 回复包含了本次被迭代的元素。


***SORT***

1. `SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern …]] [ASC | DESC] [ALPHA] [STORE destination]`
2. 返回或保存给定列表、集合、有序集合 key 中经过排序的元素。排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。
+	`[BY pattern]`：通过其它键的值进行排序
+	`[LIMIT offset count]`：使用 LIMIT 修饰符限制返回结果
+	`[GET pattern [GET pattern …]]`：使用 GET 选项， 可以根据排序的结果来取出相应的键值。
+	`[ASC | DESC]`：逆序、正序
+	`[ALPHA]`：使用 ALPHA 修饰符对字符串进行排序
+	`[STORE destination]`：通过给 STORE 选项指定一个 key 参数，可以将排序结果保存到给定的键上。

3. 返回值：没有使用 STORE 参数，返回列表形式的排序结果。 使用 STORE 参数，返回排序结果的元素数量。

```
sadd ms1 1 55 6 77 3 22

# 1. [ASC | DESC]
# 排序(默认为 asc，从小到大)
SORT ms1
# desc（从大到小）
SORT ms1 desc

# 2. [LIMIT offset count]
# 从第一个开始，输出5个
sort ms1 limit 1 5

# 3. [BY pattern]
# 通过其它键的值进行排序
set uid_1 1
set uid_55 2
set uid_6 3
set uid_77 4
set uid_3 5
set uid_22 6

sort ms1 by uid_*
1
55
6
77
322


# 4. [GET pattern [GET pattern …]]
# 输出排序后关联键的值
sort ms1 get uid_*
1
5
3
6
2
4

# 5. [ALPHA]

lpush myList aa ddf cce bbf
sort myList alpha

	aa
	bbf
	cce
	ddf

# 6. [STORE destination]
# 排序ms1，存放在ml中
sort ms1 store ml
lrange ml 0 10

```

***FLUSHDB***

1. `FLUSHDB`
2. 清空当前数据库中的所有 key。
3. 返回值：总是返回 OK

```
dbsize
19
flushdb
OK
dbsize
0
```

***FLUSHALL***

1. `FLUSHALL`
2. 清空整个 Redis 服务器的数据(删除所有数据库的所有 key ).此命令从不失败。
3. 返回值：总是返回 OK

```
redis> DBSIZE            # 0 号数据库的 key 数量
(integer) 9

redis> SELECT 1          # 切换到 1 号数据库
OK

redis[1]> DBSIZE         # 1 号数据库的 key 数量
(integer) 6

redis[1]> flushall       # 清空所有数据库的所有 key
OK

redis[1]> DBSIZE         # 不但 1 号数据库被清空了
(integer) 0

redis[1]> SELECT 0       # 0 号数据库(以及其他所有数据库)也一样
OK

redis> DBSIZE
(integer) 0
```


***SELECT***

1. `SELECT index`
2. 切换到指定的数据库，数据库索引号 index 用数字值指定，以 0 作为起始索引值。默认使用 0 号数据库。
3. 返回值：返回OK

```
select 0
"OK"
select 99
"DB index is out of range"
```

***SWAPDB***

1. `SWAPDB db1 db2`
2. 对换指定的两个数据库， 使得两个数据库的数据立即互换。
3. 返回值：OK

```
# 对换数据库 0 和数据库 1
redis> SWAPDB 0 1
OK
```

### 二、自动过期

+	EXPIRE
+	EXPIREAT
+	TTL
+	PERSIST
+	PEXPIRE
+	PEXPIREAT
+	PTTL


***EXPIRE***

1. `EXPIRE key seconds`
2. 为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。

+ 生存时间可以通过使用 DEL 命令来删除整个 key 来移除，或者被 SET 和 GETSET 命令覆写(overwrite)，这意味着，如果一个命令只是修改(alter)一个带生存时间的 key 的值而不是用一个新的 key 值来代替(replace)它的话，那么生存时间不会被改变
+ 可以对一个已经带有生存时间的 key 执行 EXPIRE 命令，新指定的生存时间会取代旧的生存时间。


3. 返回值：设置成功返回 1 。 当 key 不存在或者不能为 key 设置生存时间时(比如在低于 2.1.3 版本的 Redis 中你尝试更新 key 的生存时间)，返回 0 。

```
set i 1
expire i 30 # 生存周期，两秒
```

***EXPIREAT***

1. `EXPIREAT key timestamp`
2. EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置生存时间。不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
3. 返回值：如果生存时间设置成功，返回 1 ； 当 key 不存在或没办法设置生存时间，返回 0 。

```
redis> SET cache www.google.com
OK

redis> EXPIREAT cache 1355292000     # 这个 key 将在 2012.12.12 过期
(integer) 1

redis> TTL cache
(integer) 45081860
```

***TTL***

1. `TTL key`
2. 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)
3. 返回值：整数
+	当 key 不存在时，返回-2
+	当 key 存在但没有设置剩余生存时间时，返回-1
+	否则，以秒为单位，返回 key 的剩余生存时间。

```
set i 2
	OK
expire i 60
	1
ttl i
	55
```

***PERSIST***

1. `PERSIST key`
2. 移除给定 key 的生存时间，将这个 key 从“易失的”(带生存时间 key )转换成“持久的”(一个不带生存时间、永不过期的 key )。
3. 返回值：当生存时间移除成功时，返回 1 . 如果 key 不存在或 key 没有设置生存时间，返回 0 。

```
redis> SET mykey "Hello"
OK

redis> EXPIRE mykey 10  # 为 key 设置生存时间
(integer) 1

redis> TTL mykey
(integer) 10

redis> PERSIST mykey    # 移除 key 的生存时间
(integer) 1

redis> TTL mykey
(integer) -1
```

***PEXPIRE***

1. `PEXPIRE key milliseconds`
2. 这个命令和 EXPIRE 命令的作用类似，但是它以毫秒为单位设置 key 的生存时间，而不像 EXPIRE 命令那样，以秒为单位。
3. 返回值：设置成功，返回 1 key 不存在或设置失败，返回 0

```
redis> SET mykey "Hello"
OK

redis> PEXPIRE mykey 1500
(integer) 1

redis> TTL mykey    # TTL 的返回值以秒为单位
(integer) 2

redis> PTTL mykey   # PTTL 可以给出准确的毫秒数
(integer) 1499
```

***PEXPIREAT***

1. `PEXPIREAT key milliseconds-timestamp`
2. 这个命令和 expireat 命令类似，但它以毫秒为单位设置 key 的过期 unix 时间戳，而不是像 expireat 那样，以秒为单位。
3. 返回值：如果生存时间设置成功，返回 1 。 当 key 不存在或没办法设置生存时间时，返回0

```
redis> SET mykey "Hello"
OK

redis> PEXPIREAT mykey 1555555555005
(integer) 1

redis> TTL mykey           # TTL 返回秒
(integer) 223157079

redis> PTTL mykey          # PTTL 返回毫秒
(integer) 223157079318
```

***PTTL***

1. `PTTL key`
2. 这个命令类似于 TTL 命令，但它以毫秒为单位返回 key 的剩余生存时间，而不是像 TTL 命令那样，以秒为单位。
3. 返回值
+	当 key 不存在时，返回 -2 。
+	当 key 存在但没有设置剩余生存时间时，返回 -1 。
+	否则，以毫秒为单位，返回 key 的剩余生存时间。

```
# 不存在的 key

redis> FLUSHDB
OK

redis> PTTL key
(integer) -2


# key 存在，但没有设置剩余生存时间

redis> SET key value
OK

redis> PTTL key
(integer) -1


# 有剩余生存时间的 key

redis> PEXPIRE key 10086
(integer) 1

redis> PTTL key
(integer) 6179
```


### 三、事务

+	MULTI
+	EXEC
+	DISCARD
+	WATCH
+	UNWATCH

***MULTI***

1. `MULTI`
2. 标记一个事务块的开始。事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 EXEC 命令原子性(atomic)地执行。
3. 返回值：总是返回 OK 。

```
redis> MULTI            # 标记事务开始
OK

redis> INCR user_id     # 多条命令按顺序入队
QUEUED

redis> INCR user_id
QUEUED

redis> INCR user_id
QUEUED

redis> PING
QUEUED

redis> EXEC             # 执行
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) PONG
```


***EXEC***

1. `EXEC`
2. 执行所有事务块内的命令。

+	假如某个(或某些) key 正处于 WATCH 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令，那么 EXEC 命令只在这个(或这些) key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。可以避免多客户端导致的冲突

3. 返回值：事务块内所有命令的返回值，按命令执行的先后顺序排列。当操作被打断时，返回空值 nil 。

```
# 监视 key ，且事务成功执行

redis> WATCH lock lock_times
OK

redis> MULTI
OK

redis> SET lock "huangz"
QUEUED

redis> INCR lock_times
QUEUED

redis> EXEC
1) OK
2) (integer) 1


# 监视 key ，且事务被打断

redis> WATCH lock lock_times
OK

redis> MULTI
OK

redis> SET lock "joe"        # 就在这时，另一个客户端修改了 lock_times 的值
QUEUED

redis> INCR lock_times
QUEUED

redis> EXEC                  # 因为 lock_times 被修改， joe 的事务执行失败
(nil)
```

***DISCARD***

1. `DISCARD`
2. 取消事务，放弃执行事务块内的所有命令。如果正在使用 WATCH 命令监视某个(或某些) key，那么取消所有监视，等同于执行命令 UNWATCH 。
3. 返回值：总是返回 OK 。

```
redis> MULTI
OK

redis> PING
QUEUED

redis> SET greeting "hello"
QUEUED

redis> DISCARD
OK
```

***WATCH***

1. `WATCH key [key …]`
2. 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
3. 返回值：总是返回 OK 。

```
redis> WATCH lock lock_times
OK
```

***UNWATCH***

1. `UNWATCH`
2. 取消 WATCH 命令对所有 key 的监视。如果在执行 WATCH 命令之后， EXEC 命令或 DISCARD 命令先被执行了的话，那么就不需要再执行 UNWATCH 了。
3. 返回值：总是 OK

```
redis> WATCH lock lock_times
OK

redis> UNWATCH
OK
```


### 四、持久化

+	SAVE
+	BGSAVE
+	BGREWRITEAOF
+	LASTSAVE

***SAVE（RDB）***

1. `SAVE`
2. SAVE 命令执行一个同步保存操作，将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘。
3. 返回值：保存成功时返回 OK 。

```
redis> SAVE
OK
```

***BGSAVE***

1. `BGSAVE`
2. 在后台异步(Asynchronously)保存当前数据库的数据到磁盘。
+	BGSAVE 命令执行之后立即返回 OK ，然后 Redis fork 出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。
+	客户端可以通过 LASTSAVE 命令查看相关信息，判断 BGSAVE 命令是否执行成功。

3. 返回值：反馈信息。

```
redis> BGSAVE
Background saving started
```


***LASTSAVE***

1. `LASTSAVE`
2. 返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示。
3. 返回值：一个 UNIX 时间戳。

```
redis> LASTSAVE
(integer) 1602391934
```


***BGREWRITEAOF***

1. `BGREWRITEAOF`
2. 执行一个 AOF文件 重写操作。重写会创建一个当前 AOF 文件的体积优化版本。即使 BGREWRITEAOF 执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 BGREWRITEAOF 成功之前不会被修改。重写操作只会在没有其他持久化工作在后台执行时被触发，也就是说：

+	如果 Redis 的子进程正在执行快照的保存工作，那么 AOF 重写的操作会被预定(scheduled)，等到保存工作完成之后再执行 AOF 重写
+	如果已经有别的 AOF 文件重写在执行，那么 BGREWRITEAOF 返回一个错误，并且这个新的 BGREWRITEAOF 请求也不会被预定到下次执行。
+	从 Redis 2.4 开始， AOF 重写由 Redis 自行触发， BGREWRITEAOF 仅仅用于手动触发重写操作。


3. 返回值：

```
redis> BGREWRITEAOF
Background append only file rewriting started
```

### 五、发布与订阅

+	PUBLISH
+	SUBSCRIBE
+	PSUBSCRIBE
+	UNSUBSCRIBE
+	PUNSUBSCRIBE
+	PUBSUB

***PUBLISH***

1. `PUBLISH channel message`
2. 将信息 message 发送到指定的频道 channel 。时间复杂度为 O(N+M)，其中 N 是频道 channel 的订阅者数量，而 M 则是使用模式订阅(subscribed patterns)的客户端的数量。
3. 返回值：接收到信息 message 的订阅者数量。

```
publish mychannel "haha"
```

***SUBSCRIBE***

1. `SUBSCRIBE channel [channel …]`
2. 订阅给定的一个或多个频道的信息。
3. 返回值：接收到的信息

```
redis> subscribe msg chat_room
Reading messages... (press Ctrl-C to quit)
1) "subscribe"       # 返回值的类型：显示订阅成功
2) "msg"             # 订阅的频道名字
3) (integer) 1       # 目前已订阅的频道数量

1) "subscribe"
2) "chat_room"
3) (integer) 2

1) "message"         # 返回值的类型：信息
2) "msg"             # 来源(从那个频道发送过来)
3) "hello moto"      # 信息内容

1) "message"
2) "chat_room"
3) "testing...haha"
```

> 订阅和发布的不能是同一个客户端

***PSUBSCRIBE***

1. `PSUBSCRIBE pattern [pattern …]`
2. 订阅一个或多个符合给定模式的频道。
+	`每个模式以 * 作为匹配符，比如 it* 匹配所有以 it 开头的频道( it.news 、 it.blog 、 it.tweets 等等)， news.* 匹配所有以 news. 开头的频道( news.it 、 news.global.today 等等)，诸如此类。`
3. 返回值：接收到的信息

```
redis> psubscribe news.* tweet.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"                  # 返回值的类型：显示订阅成功
2) "news.*"                      # 订阅的模式
3) (integer) 1                   # 目前已订阅的模式的数量

1) "psubscribe"
2) "tweet.*"
3) (integer) 2

1) "pmessage"                    # 返回值的类型：信息
2) "news.*"                      # 信息匹配的模式
3) "news.it"                     # 信息本身的目标频道
4) "Google buy Motorola"         # 信息的内容

1) "pmessage"
2) "tweet.*"
3) "tweet.huangz"
4) "hello"

1) "pmessage"
2) "tweet.*"
3) "tweet.joe"
4) "@huangz morning"

1) "pmessage"
2) "news.*"
3) "news.life"
4) "An apple a day, keep doctors away"
```

***UNSUBSCRIBE***

1. `UNSUBSCRIBE [channel [channel …]]	`
2. 指示客户端退订给定的频道。一个无参数的 UNSUBSCRIBE 调用被执行，那么客户端使用 SUBSCRIBE 命令订阅的所有频道都会被退订
3. 返回值：

```
unsubscribe
0
```

***PUNSUBSCRIBE***

1. `PUNSUBSCRIBE [pattern [pattern …]]`
2. 指示客户端退订所有给定模式。
3. 返回值：这个命令在不同的客户端中有不同的表现。


***PUBSUB***

1. `PUBSUB <subcommand> [argument [argument …]]`
2. PUBSUB 是一个查看订阅与发布系统状态的内省命令.例如：
+	`PUBSUB CHANNELS [pattern]`：列出当前的活跃频道。活跃频道指的是那些至少有一个订阅者的频道， 订阅模式的客户端不计算在内。pattern 参数是可选
+	`PUBSUB NUMSUB [channel-1 … channel-N]`：返回给定频道的订阅者数量， 订阅模式的客户端不计算在内
+	`PUBSUB NUMPAT`：返回订阅模式的数量。

3. 返回值：这个命令在不同的客户端中有不同的表现

```
# 获取频道数量
PUBSUB CHANNELS
# 返回给定频道的订阅者数量， 订阅模式的客户端不计算在内
pubsub numsub channels 

# 
PUBSUB NUMPAT
```

### 六、复制

+	SLAVEOF
+	ROLE

***SLAVEOF***

1. `SLAVEOF host port`
2. SLAVEOF 命令用于在 Redis 运行时动态地修改复制(replication)功能的行为。

+	通过执行 SLAVEOF host port 命令，可以将当前服务器转变为指定服务器的从属服务器(slave server)。
+	如果当前服务器已经是某个主服务器(master server)的从属服务器，那么执行 SLAVEOF host port 将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。
+	另外，对一个从属服务器执行命令 SLAVEOF NO ONE 将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集不会被丢弃。
+	利用“SLAVEOF NO ONE 不会丢弃同步所得数据集”这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。
3. 返回值：总是返回 OK 。

```
redis> SLAVEOF 127.0.0.1 6379
OK

redis> SLAVEOF NO ONE
OK
```

***ROLE***

1. `ROLE`
2. 	返回实例在复制中担任的角色， 这个角色可以是 master 、 slave 或者 sentinel 。 除了角色之外， 命令还会返回与该角色相关的其他信息， 其中：
+	主服务器将返回属下从服务器的 IP 地址和端口。
+	从服务器将返回自己正在复制的主服务器的 IP 地址、端口、连接状态以及复制偏移量。
+	Sentinel 将返回自己正在监视的主服务器列表。

3. 返回值：ROLE 命令将返回一个数组。

```
#  主服务器
1) "master"
2) (integer) 3129659
3) 1) 1) "127.0.0.1"
      2) "9001"
      3) "3129242"
   2) 1) "127.0.0.1"
      2) "9002"
      3) "3129543"
	  
#  从服务器
1) "slave"
2) "127.0.0.1"
3) (integer) 9000
4) "connected"
5) (integer) 3167038

#  Sentinel
1) "sentinel"
2) 1) "resque-master"
   2) "html-fragments-master"
   3) "stats-master"
   4) "metadata-master"
   
```

> 哨兵（Sentinel）模式：哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

### 七、其它

***客户端与服务器命令***

+	AUTH
	+	AUTH password 密码验证
+	QUIT
	+	请求服务器关闭与当前客户端的连接。
+	INFO
	+	查看服务器配置
+	SHUTDOWN
	+	关闭服务器
+	TIME
	+	返回当前服务器时间
+	CLIENT_GETNAME/CLIENT_SETNAME
	+	返回 CLIENT SETNAME 命令为连接设置的名字。
	+	`client setname tt`
	+	`client getname`
+	CLIENT_KILL
	+	关闭地址为 ip:port 的客户端。
	+	`CLIENT KILL ip:port`
+	CLIENT_LIST
	+	以人类可读的格式，返回所有连接到服务器的客户端信息和统计数据。
	+	`CLIENT LIST`


***配置选项命令***

+	CONFIG_SET：`CONFIG SET parameter value`
	+	返回值：当设置成功时返回 OK ，否则返回一个错误。
+	CONFIG_GET：`CONFIG GET parameter`
	+	返回值：给定配置参数的值。
+	CONFIG_RESETSTAT：`CONFIG RESETSTAT`
	+	重置 INFO 命令中的某些统计数据
		+	Keyspace hits (键空间命中次数)
		+	Keyspace misses (键空间不命中次数)
		+	Number of commands processed (执行命令的次数)
		+	Number of connections received (连接服务器的次数)
		+	Number of expired keys (过期key的数量)
		+	Number of rejected connections (被拒绝的连接数量)
		+	Latest fork(2) time(最后执行 fork(2) 的时间)
		+	The aof_delayed_fsync counter(aof_delayed_fsync 计数器的值)
	+	返回值：总是返回 OK 。
+	CONFIG_REWRITE：`CONFIG REWRITE`
	+	原子性重写：对 redis.conf 文件的重写是原子性的， 并且是一致的： 如果重写出错或重写期间服务器崩溃， 那么重写失败， 原有 redis.conf 文件不会被修改。 如果重写成功， 那么 redis.conf 文件为重写后的新文件。
	+	返回值：一个状态值：如果配置重写成功则返回 OK ，失败则返回一个错误

***调试***

+	PING
+	ECHO：`ECHO message`
	+	打印一个特定的信息 message ，测试时使用。
+	OBJECT：`OBJECT subcommand [arguments [arguments]]`
+	SLOWLOG：`SLOWLOG subcommand [argument]`
	+	Slow log 是 Redis 用来记录查询执行时间的日志系统。
+	MONITOR：`MONITOR`
	+	实时打印出 Redis 服务器接收到的命令，调试用
+	DEBUG_OBJECT：`DEBUG OBJECT key`
	+	DEBUG OBJECT 是一个调试命令，它不应被客户端所使用。查看 OBJECT 命令获取更多信息。
+	DEBUG_SEGFAULT：`DEBUG SEGFAULT`
	+	执行一个不合法的内存访问从而让 Redis 崩溃，仅在开发时用于 BUG 模拟。

***内部命令***

+	MIGRATE：`MIGRATE host port key destination-db timeout [COPY] [REPLACE]`
	+	将 key 原子性地从当前实例传送到目标实例的指定数据库上，一旦传送成功， key 保证会出现在目标实例上，而当前实例上的 key 会被删除。
+	DUMP：`DUMP key`
	+	序列化给定 key ，并返回被序列化的值，使用 RESTORE 命令可以将这个值反序列化为 Redis 键
+	RESTORE：`RESTORE key ttl serialized-value [REPLACE]`
	+	反序列化给定的序列化值
+	SYNC
	+	用于复制功能(replication)的内部命令。
+	PSYNC：`PSYNC master_run_id offset`
	+	用于复制功能(replication)的内部命令。



***Lua 脚本***

+	EVAL：`EVAL script numkeys key [key …] arg [arg …]`
	+	使用 EVAL 命令对 Lua 脚本进行求值
+	EVALSHA：`EVALSHA sha1 numkeys key [key …] arg [arg …]`
	+	根据给定的 sha1 校验码，对缓存在服务器中的脚本进行求值
+	SCRIPT_LOAD：`SCRIPT LOAD script`
	+	加载脚本
+	SCRIPT_EXISTS：`SCRIPT EXISTS sha1 [sha1 …]`
	+	给定一个或多个脚本的 SHA1 校验和，返回一个包含 0 和 1 的列表，表示校验和所指定的脚本是否已经被保存在缓存当中。
+	SCRIPT FLUSH
	+	清除所有 Lua 脚本缓存。
+	SCRIPT KILL
	+	杀死当前正在运行的 Lua 脚本
	
### 八、相关文档

> [Redis 命令参考](http://redisdoc.com/)
