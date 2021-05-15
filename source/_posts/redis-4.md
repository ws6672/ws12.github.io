---
title: redis（四）数据类型相关命令
date: 2020-10-02 22:56:45
tags: [Redis]
---

# 一、字符串

***SET***

1.`SET key value [EX seconds] [PX milliseconds] [NX|XX]`
2. 将字符串值 value 关联到 key；如果 key 已经持有其他值， SET 就覆写旧值， 无视类型。当 SET 命令对一个带有生存时间（TTL）的键进行设置之后， 该键原有的 TTL 将被清除。
3. 返回值：从 Redis 2.6.12 版本开始， SET 命令只在设置操作成功完成时才返回 OK

```
# EX seconds ： 将键的过期时间设置为 seconds 秒
# PX milliseconds ： 将键的过期时间设置为 milliseconds 毫秒
# NX ： 只在键不存在时， 才对键进行设置操作
# XX ： 只在键已经存在时， 才对键进行设置操作

# 假如k不存在，设置k为1，存在60秒
set k 1 ex 60 nx
```

***GET***

1.`GET key`
2. 返回与键 key 相关联的字符串值。
3. 返回值：如果键 key 不存在， 那么返回特殊值 nil ； 否则， 返回键 key 的值。如果键 key 的值并非字符串类型， 那么返回一个错误， 因为 GET 命令只能用于字符串值。


***SETNX***

1.`SETNX key value`
2. 只在键 key 不存在的情况下， 将键 key 的值设置为 value；若键 key 已经存在， 则 SETNX 命令不做任何动作；SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。
3. 返回值：命令在设置成功时返回 1 ， 设置失败时返回 0 

***SETEX***

1.`SETEX key seconds value`

2. 将键 key 的值设置为 value ， 并将键 key 的生存时间设置为 seconds 秒钟。如果键 key 已经存在， 那么 SETEX 命令将覆盖已有的值。SETEX 命令的效果和以下两个命令的效果类似：
```
	SET key value
	EXPIRE key seconds  # 设置生存时间
```
SETEX 和这两个命令的不同之处在于 SETEX 是一个原子（atomic）操作， 它可以在同一时间内完成设置值和设置过期时间这两个操作， 因此 SETEX 命令在储存缓存的时候非常实用。

3. 返回值：命令在设置成功时返回 OK。当 seconds 参数不合法时，命令将返回一个错误。

***PSETEX***

1.```
PSETEX key milliseconds value
psetex str 500 1g
```

2. 这个命令和 SETEX 命令相似， 但它以毫秒为单位设置 key 的生存时间， 而不是像 SETEX 命令那样以秒为单位进行设置

3. 返回值：命令在设置成功时返回 OK


***GETSET***

1.`GETSET key value`
2. 将键 key 的值设为 value ， 并返回键 key 在被设置之前的旧值。
3. 返回值：返回给定键 key 的旧值；如果键 key 没有旧值， 也即是说， 键 key 在被设置之前并不存在， 那么命令返回 nil。当键 key 存在但不是字符串类型时， 命令返回一个错误

***STRLEN***

1.`STRLEN key`
2. 返回键 key 储存的字符串值的长度。
3. 返回值：STRLEN 命令返回字符串值的长度。当键 key 不存在时， 命令返回 0；当 key 储存的不是字符串值时， 返回一个错误。

***APPEND***

1.`APPEND key value`
2. 如果键 key 已经存在并且它的值是一个字符串， APPEND 命令将把 value 追加到键 key 现有值的末尾。如果 key 不存在， APPEND 就简单地将键 key 的值设为 value ， 就像执行 SET key value 一样
3. 返回值：追加 value 之后， 键 key 的值的长度。

***SETRANGE***

1.`SETRANGE key offset value`
2. 从偏移量 offset 开始， 用 value 参数覆写(overwrite)键 key 储存的字符串值。如果键 key 原来储存的字符串长度比偏移量小(比如字符串只有 5 个字符长，但你设置的 offset 是 10 )， 那么原字符和偏移量之间的空白将用零字节(zerobytes, "\x00" )进行填充
3. 返回值：SETRANGE 命令会返回被修改之后， 字符串值的长度。

***GETRANGE***

1.`GETRANGE key start end`
2. 返回键 key 储存的字符串值的指定部分， 字符串的截取范围由 start 和 end 两个偏移量决定 (包括 start 和 end 在内)。负数偏移量表示从字符串的末尾开始计数， -1 表示最后一个字符，以此类推。
3. 返回值：GETRANGE 命令会返回字符串值的指定部分。

***INCR***

1.`INCR key`
2. 为键 key 储存的数字值加上一。如果键 key 不存在， 那么它的值会先被初始化为 0 ， 然后再执行 INCR 命令；如果键 key 储存的值不能被解释为数字， 那么 INCR 命令将返回一个错误。本操作的值限制在 64 位(bit)有符号数字表示之内。
3. 返回值：INCR 命令会返回键 key 在执行加一操作之后的值。

***INCRBY***

1.`INCRBY key increment`
2. 为键 key 储存的数字值加上增量 increment 。如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 INCRBY 命令；如果键 key 储存的值不能被解释为数字， 那么 INCRBY 命令将返回一个错误。本操作的值限制在 64 位(bit)有符号数字表示之内。关于递增(increment) / 递减(decrement)操作的更多信息， 请参见 INCR 命令的文档。
3. 返回值：在加上增量 increment 之后， 键 key 当前的值。

***INCRBYFLOAT***

1.`INCRBYFLOAT key increment`
2. 为键 key 储存的值加上浮点数增量 increment。无论是键 key 的值还是增量 increment ， 都可以使用像 `2.0e7`、`3e5`、 `90e-2` 那样的指数符号(exponentialnotation)来表示， 但是， 执行 INCRBYFLOAT 命令之后的值总是以同样的形式储存， 也即是， 它们总是由一个数字， 一个（可选的）小数点和一个任意长度的小数部分组成
3. 返回值：在加上增量 increment 之后， 键 key 的值。

***DECR***

1.`DECR key`
2. 类似`INCR`，为键 key 储存的数字值减去一，本操作的值限制在64位(bit)有符号数字表示之内。
3. 返回值：DECR 命令会返回键 key 在执行减一操作之后的值。


***DECRBY***

1.`DECRBY key decrement`
2. 将键 key 储存的整数值减去减量 decrement
3. 返回值：DECRBY 命令会返回键在执行减法操作之后的值。

***MSET（原子性操作）***

1.`MSET key value [key value …]`
2. 同时为多个键设置值。如果某个给定键已经存在， 那么 MSET 将使用新值去覆盖旧值， 如果这不是你所希望的效果， 请考虑使用 MSETNX 命令， 这个命令只会在所有给定键都不存在的情况下进行设置。
3. 返回值：MSET 命令总是返回 OK 。

***MSETNX（原子性操作）***

1.`MSETNX key value [key value …]`
2. 当且仅当所有给定键都不存在时， 为所有给定键设置值。即使只有一个给定键已经存在， MSETNX 命令也会拒绝执行对所有键的设置操作。
3. 返回值：当所有给定键都设置成功时， 命令返回 1 ； 如果因为某个给定键已经存在而导致设置未能成功执行， 那么命令返回 0 。

***MGET***

1.`MGET key [key …]`
2. 返回给定的一个或多个字符串键的值。如果给定的字符串键里面， 有某个键不存在， 那么这个键的值将以特殊值 nil 表示。
3. 返回值：MGET 命令将返回一个列表， 列表中包含了所有给定键的值。

***实例***
```
# 设置值
set i 1
set j 1
get i
1

# 递增
incr i
incr i
get i
3

# 递减
decr i
decr i
1

# 获取多个值
mget i j
```


# 二、哈希表

***HSET***

1.`HSET hash[哈希表名] field[键] value[值]` 
2. 将哈希表 hash 中域 field 的值设置为 value。如果给定的哈希表并不存在， 那么一个新的哈希表将被创建并执行 HSET 操作。如果域 field 已经存在于哈希表中， 那么它的旧值将被新值 value 覆盖。
3. 返回值：当 HSET 命令在哈希表中新创建 field 域并成功为它设置值时， 命令返回 1 ； 如果域 field 已经存在于哈希表， 并且 HSET 命令成功使用新值覆盖了它的旧值， 那么命令返回 0 。

***HSETNX***

1.`HSETNX hash field value`
2. 当且仅当域 field 尚未存在于哈希表的情况下，将它的值设置为 value
3. 返回值：HSETNX 命令在设置成功时返回 1 ， 在给定域已经存在而放弃执行设置操作时返回 0 。

***HGET***

1.`HGET hash field` 
2. 返回哈希表中给定域的值。
3. 返回值：HGET 命令在默认情况下返回给定域的值。如果给定域不存在于哈希表中， 又或者给定的哈希表并不存在， 那么命令返回 nil 。

***HEXISTS***

1.`HEXISTS hash field`
2. 检查给定域 field 是否存在于哈希表 hash 当中。
3. 返回值：HEXISTS 命令在给定域存在时返回 1 ， 在给定域不存在时返回 0

***HDEL***

1.`HDEL key field [field …]`
2. 删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。
3. 返回值：被成功移除的域的数量，不包括被忽略的域。

***HLEN***

1.`HLEN key`
2. 返回哈希表 key 中域的数量。
3. 返回值：哈希表中域的数量；当 key 不存在时，返回 0。

***HSTRLEN***

1.`HSTRLEN key field`
2. 返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度（string length）。如果给定的键或者域不存在， 那么命令返回 0 。
3. 返回值：相关哈希表中键对应值的长度

***HINCRBY***

1.`HINCRBY key field increment`
2. 为哈希表 key 中的域 field 的值加上增量 increment，增量也可以为负数，相当于对给定域进行减法操作。如果 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令；如果域 field 不存在，那么在执行命令前，域的值被初始化为 0 。对一个储存字符串值的域 field 执行 HINCRBY 命令将造成一个错误
3. 返回值：执行 HINCRBY 命令之后，哈希表 key 中域 field 的值。

***HINCRBYFLOAT***

1.`HINCRBYFLOAT key field increment`
2. 为哈希表 key 中的域 field 加上浮点数增量 increment 。
3. 返回值：执行加法操作之后 field 域的值。

***HMSET***

1.`HMSET key field value [field value …]`
2. 同时将多个 field-value (域-值)对设置到哈希表 key 中。此命令会覆盖哈希表中已存在的域；如果 key 不存在，一个空哈希表被创建并执行 HMSET 操作。
3. 返回值：如果命令执行成功，返回 OK；当 key 不是哈希表(hash)类型时，返回一个错误。

***HMGET***

1.`HMGET key field [field …]`
2. 返回哈希表 key 中，一个或多个给定域的值
3. 返回值：一个包含多个给定域的关联值的表，表值的排列顺序和给定域参数的请求顺序一样。

***HKEYS***

1.`HKEYS key`
2. 返回哈希表 key 中的所有域。
3. 返回值：一个包含哈希表中所有域的表。当 key 不存在时，返回一个空表。

***HVALS***

1.`HVALS key`
2. 返回哈希表 key 中所有域的值。
3. 返回值：一个包含哈希表中所有值的表；当 key 不存在时，返回一个空表。

***HGETALL***

1.`HGETALL key`
2. 返回哈希表 key 中，所有的域和值。在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。
3. 返回值：以列表形式返回哈希表的域和域的值；若 key 不存在，返回空列表。

***HSCAN***

1.`HSCAN key cursor [MATCH pattern] [COUNT count]`

***实例***


```
# 添加键
HSET website google "www.g.cn"
HSET website gd "www.g.cn"
hgetall website

# 删除键
HDEL website gd google
2

# 获取哈希表website中键gd对应值的长度
hstrlen website gd
10
```

# 三、列表

***LPUSH***

1.`LPUSH key value [value …]`
2. 将一个或多个值 value 插入到列表 key 的表头；多个值插入相当于单个值的原子操作
3. 返回值：执行 LPUSH 命令后，列表的长度。

***LPUSHX***

1.`LPUSHX key value`
2. 将值 value 插入到列表 key 的表头，当且仅当 key 存在并且是一个列表；和 LPUSH 命令相反，当 key 不存在时， LPUSHX 命令什么也不做。
3. 返回值：LPUSHX 命令执行之后，表的长度。

***RPUSH***

1.`RPUSH key value [value …]`
2. 将一个或多个值 value 插入到列表 key 的表尾(最右边)。如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表尾：比如对一个空列表 mylist 执行 RPUSH mylist a b c ，得出的结果列表为 a b c ，等同于执行命令 RPUSH mylist a 、 RPUSH mylist b 、 RPUSH mylist c 。
3. 返回值：执行 RPUSH 操作后，表的长度。

***RPUSHX***

1.`RPUSHX key value`
2. 将值 value 插入到列表 key 的表尾，当且仅当 key 存在并且是一个列表。
3. 返回值：RPUSHX 命令执行之后，表的长度。

***LPOP***

1.`LPOP key`
2. 移除并返回列表 key 的头元素。
3. 返回值：列表的头元素。 当 key 不存在时，返回 nil 。

***RPOP***

1.`RPOP key`
2. 移除并返回列表 key 的尾元素。
3. 返回值：列表的尾元素。 当 key 不存在时，返回 nil  

***RPOPLPUSH***

1.`RPOPLPUSH source destination`
2. 将列表 source 中的最后一个元素(尾元素)弹出，并返回给客户端。将 source 弹出的元素插入到列表 destination ，作为 destination 列表的的头元素；如果 source 不存在，值 nil 被返回，并且不执行其他动作；如果 source 和 destination 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转(rotation)操作。
3. 返回值：被弹出的元素。

***LREM***

1.`LREM key count value`
2. 根据参数 count 的值，移除列表中与参数 value 相等的元素。
count 的值可以是以下几种：
+	count>0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
+	count<0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
+	count=0 : 移除表中所有与 value 相等的值。

3. 返回值：被移除元素的数量。 因为不存在的 key 被视作空表(empty list)，所以当 key 不存在时， LREM 命令总是返回 0 。

***LLEN***

1.`LLEN key`
2. 返回列表 key 的长度。
如果 key 不存在，则 key 被解释为一个空列表，返回 0 .
如果 key 不是列表类型，返回一个错误。
3. 返回值：列表 key 的长度。

***LINDEX***

1.`LINDEX key index`
2. 返回列表 key 中，下标为 index 的元素。
3. 返回值：列表中下标为 index 的元素。 如果 index 参数的值不在列表的区间范围内(out of range)，返回 nil

***LINSERT***

1.`LINSERT key BEFORE|AFTER pivot value`
2. 将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。
3. 返回值：如果命令执行成功，返回插入操作完成之后，列表的长度。 如果没有找到 pivot ，返回 -1 。 如果 key 不存在或为空列表，返回 0 。

***LSET***

1.`LSET key index value`
2. 将列表 key 下标为 index 的元素的值设置为 value 。
3. 返回值：操作成功返回 ok ，否则返回错误信息。

***LRANGE***

1.`LRANGE key start stop`
2. 返回列表 key 中指定区间内的元素，区间以偏移量 start 和 stop 指定。
+	下标(index)参数 start 和 stop 都以 0 为底，也就是说，以 0 表示列表的第一个元素，以 1 表示列表的第二个元素
+	超出范围的下标值不会引起错误。如果 start 下标比列表的最大下标 end ( LLEN list 减去 1 )还要大，那么 LRANGE 返回一个空列表。如果 stop 下标比 end 下标还要大，Redis将 stop 的值设置为 end 。
3. 返回值：一个列表，包含指定区间内的元素。

***LTRIM***

1.`LTRIM key start stop`
2. 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
3. 返回值：命令执行成功时，返回 ok 。

***BLPOP***

1.`BLPOP key [key …] timeout`
2. 它是 LPOP key 命令的阻塞版本, BLPOP 是列表的阻塞式(blocking)弹出原语。
+	超时参数 timeout 接受一个以秒为单位的数字作为值。超时参数设为 0 表示阻塞时间可以无限期延长(block indefinitely) 。
3. 返回值：如果列表为空，返回一个 nil 。 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。


***BRPOP***

1.`BRPOP key [key …] timeout`
2. BRPOP 是列表的阻塞式(blocking)弹出原语.当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素。
3. 返回值：假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。

***BRPOPLPUSH***

1.`BRPOPLPUSH source destination timeout`
2. BRPOPLPUSH 是 RPOPLPUSH 的阻塞版本。超时参数 timeout 接受一个以秒为单位的数字作为值。超时参数设为 0 表示阻塞时间可以无限期延长(block indefinitely)
3. 返回值：假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。

# 四、集合

***SADD***

1.`SADD key member [member …]`
2. 将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。假如 key 不存在，则创建一个只包含 member 元素作成员的集合。当 key 不是集合类型时，返回一个错误。
3. 返回值：被添加到集合中的新元素的数量，不包括被忽略的元素。

***SISMEMBER***

1.`SISMEMBER key member`
2. 判断 member 元素是否集合 key 的成员。
3. 返回值：如果 member 元素是集合的成员，返回 1 。 如果 member 元素不是集合的成员，或 key 不存在，返回 0 。

***SPOP***

1.`SPOP key`
2. 移除并返回集合中的一个随机元素。如果只想获取一个随机元素，但不想该元素从集合中被移除的话，可以使用 SRANDMEMBER
3. 返回值：被移除的随机元素。 当 key 不存在或 key 是空集时，返回 nil 。

***SRANDMEMBER***

1.`SRANDMEMBER key [count]`
2. 如果命令执行时，只提供了 key 参数，那么返回集合中的一个随机元素。
count 参数：
+	如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
+	如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值。

3. 返回值：

***SREM***

1.`SREM key member [member …]`
2. 移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略。当 key 不是集合类型，返回一个错误。
3. 返回值：被成功移除的元素的数量，不包括被忽略的元素。

***SMOVE***

1.`SMOVE source destination member`
2. 原子操作。将 member 元素从 source 集合移动到 destination 集合。
3. 返回值：如果 member 元素被成功移除，返回 1 。 如果 member 元素不是 source 集合的成员，并且没有任何操作对 destination 集合执行，那么返回 0 。

***SCARD***

1.`SCARD key`
2. 返回集合 key 的基数(集合中元素的数量)。
3. 返回值：集合的基数。 当 key 不存在时，返回 0 。

***SMEMBERS***

1.`SMEMBERS key`
2. 返回集合 key 中的所有成员；不存在的 key 被视为空集合。
3. 返回值：集合中的所有成员

***SSCAN***

1.`SSCAN key cursor [MATCH pattern] [COUNT count]`
2. 命令用于迭代集合键中的元素
+	cursor - 游标。
	+	当 SCAN 命令的游标参数被设置为 0 时， 服务器将开始一次新的迭代， 而当服务器向用户返回值为 0 的游标时， 表示迭代已结束
+	pattern - 匹配的模式。
+	count - 指定从数据集里返回多少元素，默认值为 10 。
3. 返回值：数组列表。

***SINTER***

1.`SINTER key [key …]`
2. 返回一个集合的全部成员，该集合是所有给定集合的交集，不存在的 key 被视为空集。当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。
3. 返回值：交集成员的列表

***SINTERSTORE***

1.`SINTERSTORE destination key [key …]`
2. 这个命令类似于 `SINTER key [key …]` 命令，但它将结果保存到 destination 集合，而不是简单地返回结果集。如果 destination 集合已经存在，则将其覆盖。destination 可以是 key 本身。
3. 返回值：结果集中的成员数量。

***SUNION***

1.`SUNION key [key …]`
2. 返回一个集合的全部成员，该集合是所有给定集合的并集。不存在的 key 被视为空集。
3. 返回值：并集成员的列表。

***SUNIONSTORE***

1.`SUNIONSTORE destination key [key …]`
2. 这个命令类似于 SUNION key [key …] 命令，但它将结果保存到 destination 集合，而不是简单地返回结果集。如果 destination 已经存在，则将其覆盖，destination 可以是 key 本身
3. 返回值：结果集中的元素数量

***SDIFF***

1.`SDIFF key [key …]`
2. 返回一个集合的全部成员，该集合是所有给定集合之间的差集，不存在的 key 被视为空集。
3. 返回值：一个包含差集成员的列表

***SDIFFSTORE***

1.`SDIFFSTORE destination key [key …]`
2. 这个命令的作用和 `SDIFF key [key …]` 类似，但它将结果保存到 destination 集合，而不是简单地返回结果集。如果 destination 集合已经存在，则将其覆盖,destination 可以是 key 本身。
3. 返回值：结果集中的元素数量

***实例***

```
sadd myset a b c d e a
5 # 存放非重复元素的个数

sismember myset a
1 #存在
sismember myset f
0 #不存在

spop myset
b # 随机元素

srandmember myset 10 
e # 返回集合中的几个随机元素
a
d
c

srem myset e a 
e # 删除指定元素
a # 剩余元素 d c

smove myset myset2 d
1 # 把 元素d 从 myset 移动到 myset2
  # 剩余元素 c
   
SCARD myset
1 # 返回集合的基数

smembers myset
c # 返回集合中的所有成员

# 设置数据
srem myset a b c d e f
sadd myset aa ab bc bd eef ff
sadd myset2 aa

sscan myset 0 match * count 10
# 0 
# match * 匹配所有数据
# count 10 返回10个数据
1) "0"  # 表示数据迭代结束
2) 1) "ff"
   2) "bc"
   3) "aa"
   4) "bd"
   5) "ab"
   6) "eef"

sinter myset myset2
aa # 求交集

SINTERSTORE myset3 myset myset2
1 # 存入 myset3的交集元素个数
```

# 五、有序集合

***ZADD***

1.`ZADD key score member [[score member] [score member] …]`
2. 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
+	如果某个 member 已经是有序集的成员，那么更新这个 member 的 score 值，并通过重新插入这个 member 元素，来保证该 member 在正确的位置上。score 值可以是整数值或双精度浮点数。
+	如果 key 不存在，则创建一个空的有序集并执行 ZADD 操作；当 key 存在但不是有序集类型时，返回一个错误。
3. 返回值：被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

***ZSCORE***

1.`ZSCORE key member`
2. 返回有序集 key 中，成员 member 的 score 值。如果 member 元素不是有序集 key 的成员，或 key 不存在，返回 nil 。
3. 返回值：member 成员的 score 值，以字符串形式表示。

***ZINCRBY***

1.`ZINCRBY key increment member`
2. 为有序集 key 的成员 member 的 score 值加上增量 increment。
+	可以通过传递一个负数值 increment ，让 score 减去相应的值，比如 ZINCRBY key -5 member ，就是让 member 的 score 值减去 5 。
+	当 key 不存在，或 member 不是 key 的成员时， ZINCRBY key increment member 等同于 ZADD key increment member；当 key 不是有序集类型时，返回一个错误。
+	score 值可以是整数值或双精度浮点数。
3. 返回值：member 成员的新 score 值，以字符串形式表示。

***ZCARD***

1.`ZCARD key`
2. 返回有序集 key 的基数。
3. 返回值：当 key 存在且是有序集类型时，返回有序集的基数。 当 key 不存在时，返回 0

***ZCOUNT***

1.`ZCOUNT key min max`
2. 返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的成员的数量。
3. 返回值：score 值在 min 和 max 之间的成员的数量。

***ZRANGE***

1.`ZRANGE key start stop [WITHSCORES]`
2. 返回有序集 key 中，指定区间（位置）内的成员
+	其中成员的位置按 score 值递增(从小到大)来排序；下标参数 start 和 stop 都以 0 为底，输出第一个参数
+	start>stop,返回空集合
+	使用 WITHSCORES 选项，来让成员和它的 score 值一并返回
3. 返回值：指定区间内，带有 score 值(可选)的有序集成员的列表

***ZREVRANGE***

1.`ZREVRANGE key start stop [WITHSCORES]`
2. 返回有序集 key 中，指定区间内的成员，其中成员的位置按 score 值递减(从大到小)来排列。
3. 返回值：指定区间内，带有 score 值(可选)的有序集成员的列表。

***ZRANGEBYSCORE***

1.`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`
2. 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。
+	min 和 max 可以是 -inf 和 +inf ，这样一来，你就可以在不知道有序集的最低和最高 score 值的情况下，使用 ZRANGEBYSCORE 这类命令。默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 `(`符号来使用可选的开区间 (小于或大于)。
+	WITHSCORES 带分数输出
+	LIMIT offset count 支持分页
3. 返回值：指定区间内，带有 score 值(可选)的有序集成员的列表。

***ZREVRANGEBYSCORE***

1.`ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]`
2. 返回有序集 key 中， score 值介于 max 和 min 之间(默认包括等于 max 或 min )的所有的成员。有序集成员按 score 值递减(从大到小)的次序排列。
3. 返回值：指定区间内，带有 score 值(可选)的有序集成员的列表。

***ZRANK***

1.`ZRANK key member`
2. 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列。
3. 返回值：如果 member 是有序集 key 的成员，返回 member 的排名。 如果 member 不是有序集 key 的成员，返回 nil 。

***ZREVRANK***

1.`ZREVRANK key member`
2. 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减(从大到小)排序。排名以 0 为底，也就是说， score 值最大的成员排名为 0 。
3. 返回值：如果 member 是有序集 key 的成员，返回 member 的排名。 如果 member 不是有序集 key 的成员，返回 nil 。

***ZREM***

1.`ZREM key member [member …]`
2. 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。
3. 返回值：被成功移除的成员的数量，不包括被忽略的成员。

***ZREMRANGEBYRANK***

1.`ZREMRANGEBYRANK key start stop`
2. 移除有序集 key 中，指定排名(rank)区间内的所有成员，区间分别以下标参数 start 和 stop 指出，包含 start 和 stop 在内，从0开始。
3. 返回值：被移除成员的数量。



***ZREMRANGEBYSCORE***

1.`ZREMRANGEBYSCORE key min max`
2. 移除有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。
3. 返回值：被移除成员的数量。

***ZRANGEBYLEX***

1.`ZRANGEBYLEX key min max [LIMIT offset count]`
2. 解析字符串，按字典排序为集合中的字符串排序，按区间输出。
3. 返回值：数组回复：一个列表，列表里面包含了有序集合在指定范围内的成员。

***ZLEXCOUNT***

1.`ZLEXCOUNT key min max`
2. 对于一个所有成员的分值都相同的有序集合键 key 来说， 这个命令会返回该集合中， 成员介于 min 和 max 范围内的元素数量。
3. 返回值：整数回复：指定范围内的元素数量。

***ZREMRANGEBYLEX***

1.`ZREMRANGEBYLEX key min max`
2. 对于一个所有成员的分值都相同的有序集合键 key 来说， 这个命令会移除该集合中， 成员介于 min 和 max 范围内的所有元素。
3. 返回值：整数回复：被移除的元素数量。

***ZSCAN***

1.`ZSCAN key cursor [MATCH pattern] [COUNT count]`
2. 元素遍历
3. 返回值：数组列表

***ZUNIONSTORE***

1.`ZUNIONSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]`
2. 计算给定的一个或多个有序集的并集，其中给定 key 的数量必须以 numkeys 参数指定，并将该并集(结果集)储存到 destination。默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之 和 。
+	使用 WEIGHTS 选项，你可以为 每个 给定有序集 分别 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 score 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。
+	使用 AGGREGATE 选项，你可以指定并集的结果集的聚合方式。
3. 返回值：保存到 destination 的结果集的基数。

***ZINTERSTORE***

1.`ZINTERSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]`
2. 计算给定的一个或多个有序集的交集，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集(结果集)储存到 destination 。默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之和.
3. 返回值：保存到 destination 的结果集的基数。

***实例***

```
zadd myzset 1 aa 2 ab 3 cc 4 ddc 5 fff 6 efe
6 # 成功添加元素的个数

zscore myzset cc
3 # 返回相对应分数

zincrby myzset 10 aa
11 # 返回新的分数
zincrby myzset -10 aa
1

zcard myzset
6 # 元素个数

zcount myzset 2 6
5 # 返回分数在 2-6 的元素个数（ab cc ddc fff efe）

zrange myzset 1 3
ab # 数字越大，排序越小
cc
ddc

zrevrange myzset 1 3 withscores
fff # 逆序输出区间内的元素
5
ddc
4
cc
3

zrangebyscore myzset 1 6 withscores # 输出分数在1-6的元素
zrangebyscore myzset -inf +inf withscores # 不知最大最小分数输出有序表
zrangebyscore myzset (1 (6 withscores # 输出大于1小于6的所有元素
zrangebyscore myzset (1 (6 withscores limit 1 3 # 分页，输出大于1小于6的所有元素

zrank myzset ab
1 # 获取元素ab的排序，从0开始递增

zrevrank myset aa
5 # 从0开始递减

```

# 六、HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。
在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。
但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素


***PFADD***

1.`PFADD key element [element …]`
2. 将任意数量的元素添加到指定的 HyperLogLog 里面。
调用 PFADD key element [element …] 命令时可以只给定键名而不给定元素：
+	如果给定键已经是一个 HyperLogLog ， 那么这种调用不会产生任何效果；
+	但如果给定的键不存在， 那么命令会创建一个空的 HyperLogLog ， 并向客户端返回 1 
3. 返回值：整数回复： 如果 HyperLogLog 的内部储存被修改了， 那么返回 1 ， 否则返回 0 。




***PFCOUNT***

1.`PFCOUNT key [key …]`
2. 当 PFCOUNT key [key …] 命令作用于单个键时， 返回储存在给定键的 HyperLogLog 的近似基数， 如果键不存在， 那么返回 0 。
+	举个例子， 为了记录一天会执行多少次各不相同的搜索查询， 一个程序可以在每次执行搜索查询时调用一次 PFADD key element [element …] ， 并通过调用 PFCOUNT key [key …] 命令来获取这个记录的近似结果。
3. 返回值：整数回复： 给定 HyperLogLog 包含的唯一元素的近似数量。


***PFMERGE***

1.`PFMERGE destkey sourcekey [sourcekey …]`
2. 将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。合并得出的 HyperLogLog 会被储存在 destkey 键里面， 如果该键并不存在， 那么命令在执行之前， 会先为该键创建一个空的 HyperLogLog。
3. 返回值：字符串回复：返回 OK 。


# 七、地理位置


***GEOADD***

1.`GEOADD key longitude latitude member [longitude latitude member …]`
2. 将给定的空间元素（纬度、经度、名字）添加到指定的键里面。 这些数据会以有序集合的形式被储存在键里面， 从而使得像 GEORADIUS 和 GEORADIUSBYMEMBER 这样的命令可以在之后通过位置查询取得这些元素。
GEOADD 命令以标准的 x,y 格式接受参数， 所以用户必须先输入经度， 然后再输入纬度。 GEOADD 能够记录的坐标是有限的： 非常接近两极的区域是无法被索引的。 精确的坐标限制由 EPSG:900913 / EPSG:3785 / OSGEO:41001 等坐标系统定义， 具体如下：

+	有效的经度介于 -180 度至 180 度之间。
+	有效的纬度介于 -85.05112878 度至 85.05112878 度之间。
3. 返回值：新添加到键里面的空间元素数量， 不包括那些已经存在但是被更新的元素。

***GEOPOS***

1.`GEOPOS key member [member …]`
2. 从键里面返回所有给定位置元素的位置（经度和纬度）。因为 GEOPOS 命令接受可变数量的位置元素作为输入， 所以即使用户只给定了一个位置元素， 命令也会返回数组回复。
3. 返回值：GEOPOS 命令返回一个数组， 数组中的每个项都由两个元素组成： 第一个元素为给定位置元素的经度， 而第二个元素则为给定位置元素的纬度。 当给定的位置元素不存在时， 对应的数组项为空值。

***GEODIST***

1.`GEODIST key member1 member2 [unit]`
2. 返回两个给定位置之间的距离。如果两个位置之间的其中一个不存在， 那么命令返回空值。

指定单位的参数 unit 必须是以下单位的其中一个：

+	m 表示单位为米(默认)
+	km 表示单位为千米
+	mi 表示单位为英里
+	ft 表示单位为英尺

3. 返回值：计算出的距离会以双精度浮点数的形式被返回。 如果给定的位置元素不存在， 那么命令返回空值。

***GEORADIUS***

1.`GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]`
2. 以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。
longitude 经度
latitude 纬度 
radius m|km|ft|mi 半径
WITHDIST：在返回位置元素的同时，将位置元素与中心之间的距离也一并返回。 距离的单位和用户给定的范围单位保持一致。
WITHCOORD：将位置元素的经度和维度也一并返回。
WITHHASH：以 52 位有符号整数的形式，返回位置元素经过原始 geohash 编码的有序集合分值。这个选项主要用于底层应用或者调试， 实际中的作用并不大。
ASC|DESC：按从近到远|从远到近的方式返回位置元素
COUNT：限制返回元素个数

3. 返回值：GEORADIUS 命令返回一个数组

***GEORADIUSBYMEMBER***

1.`GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]`
2. 这个命令和 GEORADIUS 命令一样， 都可以找出位于指定范围内的元素， 但是 GEORADIUSBYMEMBER 的中心点是由给定的位置元素决定的， 而不是像 GEORADIUS 那样， 使用输入的经度和纬度来决定中心点。
3. 返回值：一个数组， 数组中的每个项表示一个范围之内的位置元素。

***GEOHASH***

1.`GEOHASH key member [member …]`
2. 返回一个或多个位置元素的 Geohash 表示。
3. 返回值：一个数组， 数组的每个项都是一个 geohash 。 命令返回的 geohash 的位置与用户给定的位置元素的位置一一对应。

# 八、位图

***SETBIT***

1.`SETBIT key offset value`
2. 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。位的设置或清除
+	取决于 value 参数，可以是 0 也可以是 1 。
+	当 key 不存在时，自动生成一个新的字符串值。字符串会进行伸展(grown)以确保它可以将 value 保存在指定的偏移量上。
+	当字符串值进行伸展时，空白位置以 0 填充。
+	offset 参数必须大于或等于 0 ，小于 2^32 (bit 映射被限制在 512 MB 之内)。
3. 返回值：指定偏移量原来储存的位。

***GETBIT***

1.`GETBIT key offset`
2. 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。当 offset 比字符串值的长度大，或者 key 不存在时，返回 0 。
3. 返回值：字符串值指定偏移量上的位(bit)。

***BITCOUNT***

1.`BITCOUNT key [start] [end]`
2. 计算给定字符串中，被设置为 1 的比特位的数量。一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。
3. 返回值：被设置为 1 的位的数量。

***BITPOS***

1.`BITPOS key bit [start] [end]`
2. 返回位图中第一个值为 bit 的二进制位的位置。在默认情况下， 命令将检测整个位图， 但用户也可以通过可选的 start 参数和 end 参数指定要检测的范围。
3. 返回值：整数回复。

***BITOP***

1.`BITOP operation destkey key [key …]`
2. 对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。
operation 可以是 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种：
+	BITOP AND destkey key [key ...] ，对一个或多个 key 求逻辑并，并将结果保存到 destkey 。
+	BITOP OR destkey key [key ...] ，对一个或多个 key 求逻辑或，并将结果保存到 destkey 。
+	BITOP XOR destkey key [key ...] ，对一个或多个 key 求逻辑异或，并将结果保存到 destkey 。
+	BITOP NOT destkey key ，对给定 key 求逻辑非，并将结果保存到 destkey 。

3. 返回值：保存到 destkey 的字符串的长度，和输入 key 中最长的字符串长度相等。

***BITFIELD***

1.`BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]`
2. BITFIELD 命令可以将一个 Redis 字符串看作是一个由二进制位组成的数组， 并对这个数组中储存的长度不同的整数进行访问 （被储存的整数无需进行对齐）。 换句话说， 通过这个命令， 用户可以执行诸如 “对偏移量 1234 上的 5 位长有符号整数进行设置”、 “获取偏移量 4567 上的 31 位长无符号整数”等操作。 此外， BITFIELD 命令还可以对指定的整数执行加法操作和减法操作， 并且这些操作可以通过设置妥善地处理计算时出现的溢出情况。
3. 返回值：BITFIELD 命令的返回值是一个数组， 数组中的每个元素对应一个被执行的子命令。 需要注意的是， OVERFLOW 子命令本身并不产生任何回复。

