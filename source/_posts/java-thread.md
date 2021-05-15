---
title: java面试（一）多线程
date: 2020-04-12 17:27:42
tags: [ms]
---

### 多线程有几个周期

线程的生命周期包含5个阶段，包括：新建、就绪、运行、阻塞、销毁。

+	新建：就是刚使用new方法，new出来的线程；
+	就绪：就是调用的线程的start()方法后，这时候线程处于等待CPU分配资源阶段，谁先抢的CPU资源，谁开始执行;
+	运行：当就绪的线程被调度并获得CPU资源时，便进入运行状态，run方法定义了线程的操作和功能;
+	阻塞：在运行状态的时候，可能因为某些原因导致运行状态的线程变成了阻塞状态，比如sleep()、wait()之后线程就处于了阻塞状态，这个时候需要其他机制将处于阻塞状态的线程唤醒，比如调用notify或者notifyAll()方法。唤醒的线程不会立刻执行run方法，它们要再次等待CPU分配资源进入运行状态;
+	销毁：如果线程正常执行完毕后或线程被提前强制性的终止或出现异常导致结束，那么线程就要被销毁，释放资源;

![线程生命周期](/image/java/ms/xcsmzq.png)


### 新建线程有几种方法
+	继承 Thread类
+	实现 Runnable 接口

Java不支持类的多重继承，但允许你调用多个接口。所以如果你要继承其他类，当然是调用Runnable接口好 了。

### Runnable中 start和run的区别
start会启动一个线程，而run只是表示运行一个普通方法。

### 线程中属于Object的方法

```
  public final native void notify();
  public final native void wait(long timeout) throws InterruptedException;
  public final native void notifyAll();
```


### 什么是volatile
volatile是一个特殊的修饰符，只有成员变量才能使用它。在Java并发程序缺少同步类的情况下，多线程对成员变量的操作对其它线程是透明的。volatile变量可以保证下一个读取操作会在前一个写操作之后发生。

### 什么是竞态条件
是指多个线程同时修改一个数据，数据的结果由线程的运行时间决定。


### 一个线程运行时发生异常会怎样？
如果异常没有被捕获该线程将会停止执行。


###  如何在两个线程间共享数据
通过共享对象

###	Java中notify 和 notifyAll有什么区别？
notify()方法不能唤醒某个具体的线程，所以只有一个线程在等 待的时候它才有用武之地。而notifyAll()唤醒所有线程并允许他们争夺锁确保了至少有一个线程能继续运行。


### 为什么wait和notify方法要在同步块中调用？
主要是因为Java API强制要求这样做，如果你不这么做，你的代码会抛出IllegalMonitorStateException异常。还有一个原因是为了避免wait和notify之间产生竞态条件。