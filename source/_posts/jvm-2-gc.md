---
title: jvm探索（二）垃圾回收机制
date: 2020-03-30 16:18:12
tags: [java]
---

### 一、JVM 堆内存空间
`JVM Heap`分为两个不同的分代：新生代以及旧生代：

![JVM Heap](/image/jvm/jvm-heap.png)


+	堆内存分为 年轻代（Young Generation）、老年代（Old Generation）；~非堆内存就一个永久代（Permanent Generation），即方法区。~垃圾收集器还为年轻代和老年代提供了虚拟空间，垃圾收集器使用它们来调整其他区域的大小，主要是为了满足不同的GC目标。
	+	年轻代又分为`Eden`和`Survivor`区。`Eden`区占大容量，`Survivor`两个区占小容量，默认比例是8:1:1。
		+	`Eden`区：新创建对象存放位置。
		+	`Survivor`区：堆回收时，存活对象的存放位置。由 `FromSpace` 和`ToSpace`组成。
			+	步骤一：第一次Minor GC后，幸存的对象从`Eden`区复制到`Survivor0`，即`From`区；`Eden`区清空。
			+	步骤二：第二次Minor GC后，`Eden`和`Survivor0`中的存活对象又会被复制送入第二块幸存空间，即`To`区；`Eden`和`Survivor0`清空。
			+	然后下一轮S0与S1交换角色，如此循环往复（重复步骤二）。*如果对象的复制次数达到16次，该对象就会被送到老年代中*。如此操作，永远有一块`Survivor`区是空的，而另外一块`Survivor`区是无碎片的。
	
	+	年老代主要存放JVM认为生命周期比较长的对象（经过几次的Young Gen的垃圾回收后仍然存在），采用压缩的方式来避免内存碎片
	
![jvm-heap-handle](/image/jvm/jvm-heap-handle.png)	
	
> 在JDK1.8版本废弃了永久代，替代的是元空间（MetaSpace），元空间与永久代上类似，都是方法区的实现，他们最大区别是：元空间并不在JVM中，而是使用本地内存。



### 二、垃圾回收算法

***1.引用计数***
比较古老的回收算法。原理是此对象有一个引用，即增加一个计数，删除一个引用则减少一个计数。垃圾回收时，只用收集计数为0的对象。此算法最致命的是无法处理循环引用的问题。

***2.复制算法***
此算法把内存空间划为两个相等的区域，每次只使用其中一个区域。垃圾回收时，遍历当前使用区域，把正在使用中的对象复制到另外一个区域中。此算法每次只处理正在使用中的对象，因此复制成本比较小，同时复制过去以后还能进行相应的内存整理，不会出现“碎片”问题。当然，此算法的缺点也是很明显的，就是需要两倍内存空间。

***3.标记-清除算法***
此算法执行分两阶段。第一阶段从引用根节点开始标记所有被引用的对象，第二阶段遍历整个堆，把未标记的对象清除。此算法需要暂停整个应用，同时，会产生内存碎片。


***4.标记-整理算法***
此算法结合了“标记-清除”和“复制”两个算法的优点。也是分两阶段，第一阶段从根节点开始标记所有被引用对象，第二阶段遍历整个堆，把清除未标记对象并且把存活对象“压缩”到堆的其中一块，按顺序排放。此算法避免了“标记-清除”的碎片问题，同时也避免了“复制”算法的空间问题。


### 三、垃圾收集器（Garbage Collectors）

***次收集器（Minor Gc）***
+	从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor GC。
+	当JVM无法为新对象分配空间时（生成区），总是会触发次收集器。因此，分配率越高，次要GC的执行频率就越高。相对于全收集而言，收集间隔较短。
+	只要空间已满，就将复制其全部内容，并且指针可以再次从零开始跟踪可用内存，使用`Mark and Copy`算法来清理Eden和Survivor空间。

***主收集器（Major GC）***
Major GC 用于清理`Tenured space`,即`老年代`所在的空间。

***全收集器（Full GC）***
+	Full GC 是清理整个堆空间—包括年轻代和永久代。
+	Full GC，指发生在老年代的GC，`出现了Full GC一般会伴随着至少一次的Minor GC`（老年代的对象大部分是Scavenge GC过程中从新生代进入老年代），比如：分配担保失败。Full GC的速度一般会比Scavenge GC慢10倍以上。当老年代内存不足或者显式调用System.gc()方法时，会触发Full GC。
+	当老年代或者持久代堆空间满了，会触发全收集操作。可以使用System.gc()方法来显式的启动全收集，全收集一般根据堆大小的不同，需要的时间不尽相同，但一般会比较长。

> 注：在实际应用中，不必过分关注GC类型，而是着眼于GC是停止了所有应用的线程还是可以与应用线程同时进行。

### 四、分代垃圾收集器

***(1)串行收集器（Serial GC）***
Serial收集器是Hotspot运行在Client模式下的默认新生代收集器
+	单线程收集器。
+	通常适用于数据量少的小型应用程序。
+	可以通过指定命令行选项来启用:
	+	`-XX:+UseSerialGC`



***(2)并行收集器（Parallel GC）***

+	串行和并行之间的区别在于并行GC使用多个线程来执行垃圾收集过程。
+	此GC类型也称为吞吐量收集器。
+	可以通过显式指定以下选项来启用它：
	+	`-XX:+UseParallelGC`

***(3)Parallel Scavenge收集器***
与ParNew类似, Parallel Scavenge也是使用复制算法, 也是并行多线程收集器. 但与其他收集器关注尽可能缩短垃圾收集时间不同, Parallel Scavenge更关注系统吞吐量:
+	`系统吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间)`

***(4)Serial Old收集器***

Serial Old是Serial收集器的老年代版本, 同样是单线程收集器,使用“标记-整理”算法。

***(5)Parallel Old***
Parallel Old是Parallel Scavenge收集器的老年代版本, 使用多线程和“标记－整理”算法, 吞吐量优先, 主要与Parallel Scavenge配合在注重吞吐量及CPU资源敏感系统内使用.

***(6)标记-扫描 并发收集器(Concurrent Mark Sweep)***
CMS(Concurrent Mark Sweep)收集器是一款具有划时代意义的收集器, 一款真正意义上的并发收集器。
CMS是一种以获取最短回收停顿时间为目标的收集器(CMS又称多并发低暂停的收集器), 基于”标记-清除”算法实现。

整个GC过程分为以下4个步骤:

+	初始标记(CMS initial mark)
+	并发标记(CMS concurrent mark: GC Roots Tracing过程)
+	重新标记(CMS remark)
+	并发清除(CMS concurrent sweep: 已死对象将会就地释放, 注意:此处没有压缩)

通过以下选项启动：
	`-XX:+UseConcMarkSweepGC`

> 特点：应用程序暂停时间保持最短

***(7)G1收集器（Garbage-First）***
G1(Garbage-First)是一款面向服务端应用的收集器, 主要目标用于配备多颗CPU的服务器治理大内存。

与其他基于分代的收集器不同, G1将整个Java堆划分为多个大小相等的独立区域(Region), 虽然还保留有新生代和老年代的概念, 但新生代和老年代不再是物理隔离的了, 它们都是一部分Region(不需要连续)的集合

通过以下选项启动：
	`-XX:+UseG1GC`

> 特点：高吞吐量和合理的应用程序暂停时间。


### 四、参考文章
>	[Yet Another Post About Java Heap Space](https://dzone.com/articles/gc-explained-heap)
[JVM垃圾回收算法及分代垃圾收集器](https://www.cnblogs.com/guanghe/p/10524807.html)
[Minor GC vs Major GC vs Full GC](https://dzone.com/articles/minor-gc-vs-major-gc-vs-full)
[Java Memory Management](https://dzone.com/articles/java-memory-management)