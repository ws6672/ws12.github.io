---
title: Redis 中 Hyperloglog 的使用
date: 2020-10-12 18:03:06
tags: [Redis]
---

### 一、什么是Hyperloglog

Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素


***基数***

```
基数（cardinal number）在数学上，是集合论中刻画任意集合大小的一个概念。有限集合的基数，其意义与日常用语中的“基数”相同，例如{{a,b,c}的基数是3，不包含重复元素。
```

***原理***

HyperLogLog实际上不会存储每个元素的值，它使用的是概率算法，通过存储元素的hash值的第一个1的位置，来计算元素数量。这个统计准确度不是100%的，可以通过调节参数提高准确度。

1. 伯努利实验
概率学存在伯努利实验，假设你抛了很多次硬币，你告诉在这次抛硬币的过程中最多只有两次扔出连续的反面，让我猜你总共抛了多少次硬币。已知的是最大的k值，可以用kmax表示，由于每次抛硬币的结果只有0和1两种情况，因此，kmax在任意回合出现的概率即为`(1/2)^k*max`。

> 伯努利试验是数学概率论中的一部分内容，它的典故来源于抛硬币。

伯努利试验容易得出有以下结论：

+	n 次伯努利过程的投掷次数都不大于 k_max。
+	n 次伯努利过程，至少有一次投掷次数等于 k_max
+	n = 2^(k_max)

这种局部信息预估整体数据量，用概率和统计的计算公式可以得到一个高准确度的结果。


2. 分桶

redis 中存在 16384 个桶来记录各自桶的元素数量，当一个元素过来，它会散列到其中一个桶。当元素到来时，通过 hash 算法将这个元素分派到其中的一个小集合存储，同样的元素总是会散列到同样的小集合。这样总的计数就是所有小集合大小的总和。使用这种方式精确计数除了可以增加元素外，还可以减少元素。

HyperLogLog占用的空间 为 `16384 *6bit/8/1024=12k`。在计数比较小的时候，多数桶的计数值是零，会采用稀疏存储，稀疏存储的空间远远小于12k字节。而密集存储会恒定占用12K字节。


### 常用方法


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


[Redis HyperLogLog](https://www.runoob.com/redis/redis-hyperloglog.html)
[探索HyperLogLog算法（含Java实现）](https://www.jianshu.com/p/55defda6dcd2)