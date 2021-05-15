---
title: redis 布隆过滤器
date: 2020-10-15 22:06:16
tags: [Redis]
---

Redis 4.x 新特性 支持布隆过滤器插件，可以进行统计去重。
> 布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都比一般的算法要好的多，缺点是有一定的误识别率和删除困难。


### 一、入门

#### 1.基础


***特点***

布隆过滤器是一个BIT数组，记录数据，通过hash函数得到数组下标并且标注为1，这样一个数据就只占用1BIT的数据。它可以基于合集判断数据可能存在或数据一定存在。

***准确度***

+	由于是通过hash函数获取的数组下标，所以也可能存在hash碰撞的问题。所以，布隆过滤器检测存在的键，不一定存在；但布隆过滤器检测不存在的键，一定不存在。
+	为了提高准确度，可以通过增加布隆过滤器中数组的长度，来降低hash碰撞几率；还可以通过多个hash函数标记同一个数据，由多个标记位确定同一个数据是否存在。

***删除***

布隆过滤器无法准确判断数据是否存在，只能保证数据【不存在】，所以它不支持删除。




#### 2.使用


相关资源：
> [布隆过滤器插件](https://github.com/RedisBloom/RedisBloom)
[官网](https://oss.redislabs.com/redisbloom/Quick_Start/)
[Redis 模块](https://github.com/RedisLabsModules/)


***在Linux安装***

```
# 1. 安装
# LINUX MAKE
git clone https://github.com/RedisBloom/RedisBloom.git
cd redisbloom
make

# DOCKET
docker run -p 6379:6379 --name redis-redisbloom redislabs/rebloom:latest

# 2. 启动
/path/to/redis-server --loadmodule ./redisbloom.so
$redis-server --loadmodule /usr/local/RedisBloom/redisbloom.so

# 3. 使用
$ redis-cli 
127.0.0.1:6379> bf.add flt a
(integer) 1
127.0.0.1:6379> bf.add flt a
(integer) 0
```

***命令***

Redis中布隆过滤器的命令如下

+	bf.add：添加元素到过滤器中
+	bf.exists：判断元素是否在过滤器中
+	bf.madd：添加多个元素
+	bf.mexists：查询多个元素

测试如下：

```
# bf.add
127.0.0.1:6379> bf.add flt a
(integer) 1
127.0.0.1:6379> bf.add flt b
(integer) 1
127.0.0.1:6379> bf.add flt c
(integer) 1
127.0.0.1:6379> bf.add flt a
(integer) 0

# bf.exists
127.0.0.1:6379> bf.exists flt a
(integer) 1
127.0.0.1:6379> bf.exists flt d
(integer) 0

# bf.madd 批量添加
127.0.0.1:6379> bf.madd flt h i j k
1) (integer) 1
2) (integer) 1
3) (integer) 1
4) (integer) 1

# bf.mexists 批量测试
127.0.0.1:6379> bf.mexists flt h i j z
1) (integer) 1
2) (integer) 1
3) (integer) 1
4) (integer) 0

```


布隆过滤器可以高效的判断出元素是否在集合中，但是由于是通过哈希函数来确定下标的，所以数据量过大会导致哈希碰撞严重，进而大大降低准确率。


### 二、进阶

在上面的例子中，在第一次使用`bf.add`时，就会使用默认参数生成布隆过滤器。因此，为了提高准确度，它也支持自定义布隆过滤器。

#### 1. bf.reserve

`[bf.reserve key error_rate capacity]`
+	用于自定义布隆过滤器
+	key：键
+	error_rate：期望错误率，期望错误率越低，需要的空间就越大。
+	capacity：初始容量，当实际元素的数量超过这个初始化容量时，误判率上升。

```
# 默认的error_rate是 0.01，capacity是 100。

# 错误率为0.00001，初始容量为10000
127.0.0.1:6379> bf.reserve myflt 0.00001 10000
OK

127.0.0.1:6379> bf.madd myflt a b c d e f g h
1) (integer) 1
2) (integer) 1
3) (integer) 1
4) (integer) 1
5) (integer) 1
6) (integer) 1
7) (integer) 1
8) (integer) 1

```

> 注：布隆过滤器的error_rate越小，需要的存储空间就越大，对于不需要过于精确的场景，error_rate设置稍大一点也可以。布隆过滤器的capacity设置的过大，会浪费存储空间，设置的过小，就会影响准确率

***hash函数个数以及布隆过滤器长度的抉择***


#### 2. 原理

Redis中布隆过滤器的底层就是一个位数组还有几个无偏哈希函数（哈希分布均匀）。例如：

```
1. key1 通过不同的哈希函数进行映射
key1--hashA（）--2
key1--hashB（）--7
key1--hashC（）--3

2. 设置为1
arr[2]=1
arr[7]=1
arr[3]=1

```

当通过`bf.add`添加元素的时候，会通过多个哈希函数获取不同的索引，然后将多个索引所在的数组位设置为1。当我们通过`bf.exists`测试元素是否存在的时候，如果有一个索引不为1，那么该元素就不存在。但如果都为1，也无法保证元素就一定存在，因为这些1可能是其他元素的标记位。



#### 3. 应用

+	解决缓存穿透的问题
	+	可以使用布隆过滤器解决缓存穿透的问题，把已存在数据的key存在布隆过滤器中。当有新的请求时，先到布隆过滤器中查询是否存在，如果不存在该条数据直接返回；如果存在该条数据再查询缓存查询数据库。
+	黑名单校验
	+	发现存在黑名单中的，就执行特定操作
+	限制IP
+	统计新增用户


### 四、参考文章
> [详细解析Redis中的布隆过滤器及其应用](https://www.cnblogs.com/heihaozi/p/12174478.html)