---
title: jvm探索（零）从常见的异常入手
date: 2019-10-28 17:05:48
tags: [java]
---

### 【java.lang.OutOfMemoryError: GC overhead】  && 【  GC overhead limit exceeded】
【 java.lang.OutOfMemoryError: GC overhead】是指java栈空间溢出，这里有多个可能
+	存在严重的内存泄漏，而这里其实也有两种可能
	+	代码编写不当导致的异常，例如不经意间编写了死循环。这需要用专门的软件进行测试。
	+	在程序运行中，强行关闭，导致有许多空间收不回来。这里，通常重启电脑就可以解决了。
+	程序编译时间过长，分配空间过小，导致堆栈空间溢出。修改配置即可。
+	也有可能是一些比较少见的错误

我使用的编译软件是IDEA，在如下类似目录【IntelliJ IDEA XXX\bin】打开配置文件【idea.exe.vmoptions/idea64.exe.vmoptions】，具体看电脑系统的位数。

```
-Xmx1024m    修改堆栈空间分配
-XX:-UseGCOverheadLimit  增加参数
```


【GC overhead limt exceed】是Hotspot VM 1.6定义的一个策略，通过统计GC时间来预测是否要OOM了，提前抛出异常，防止OOM发生。这个功能还是有一定用处的，在预计到你的程序要挂掉前拯救一波，保存数据，避免强行关闭导致的数据丢失。
> 并行/并发回收器在GC回收时间过长时会抛出OutOfMemroyError。过长的定义是，超过98%的时间用来做GC并且回收了不到2%的堆内存。用来避免内存过小造成应用不能正常工作。
OOM,内存溢出。


### IDEA 出现【 GC overhead limit exceeded】
在 IDEA中修改Complier配置项。

![Complier配置项](/image/jvm/idea-c.png)
![maven配置项](/image/jvm/idea-maven-dz.png)
