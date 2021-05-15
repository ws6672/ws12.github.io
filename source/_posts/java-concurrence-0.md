---
title: java 并发（二） 线程安全与同步
date: 2021-01-11 18:51:14
tags: [并发]
---


# 一、基础

1. 概念

+	进程：进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位
+	线程：是轻量级进程，是大多数操作系统中时序调度的基本单位。多个线程可以同时共享进程的内存空间，可以方便的进行数据共享，优于进程间的通讯。
+	并发：多个任务交替使用CPU
+	并行：多任务同时运行在多个CPU上
+	异步与同步：着眼于消息通信机制，指请求。同步是请求得到响应前不继续运行；而异步可以在响应返回前继续执行其它任务。
+	阻塞与非阻塞：等待调用结果（消息，返回值）时的状态，指调用者本身。阻塞会等待请求，非阻塞的如果请求没有结果，不会阻塞程序。
 
2. 并发导致的问题


***竞争条件***

竞争条件指多个线程或者进程在读写一个共享数据时结果依赖于它们执行的相对时间的情形。竞争条件发生在当多个进程或者线程在读写数据时，其最终的的结果依赖于多个进程的指令执行顺序。访问的顺序决定了变量的结果，这称之为`竞争条件`。


***递增计数器***

![不安全的序列生成器](/image/java-bf/java-fxcaqxl.png)

如上图所示，是一个不安全的序列生成器；在i可能同时被两个线程得到数据，最后导致拿到一样的序号。Java中可以使用`AtomicLong`(原子变量)声明变量避免这种情况。

所以，递增计数器（i++ -> 读写改）也包含竞争条件，因为不是原子操作。


***检查再运行***
检查再运行是竞争条件之一，在某个判断true/false的语句中，可能有多个线程得出相同的判断结果，然后会运行相同的代码，该代码段如果会导致判断发生变化，那么就可能会引发错误。

```
	if(flag) {
		// 创建指定文件
		...
		flag=false;
	}
	//在多线程中，会有多个线程创建文件，导致资源异常
```

单例模式中，懒汉式的代码就是检查再运行，所以存在竞争条件。

***活跃度危险***
线程A在等待线程B的资源，线程B在等待线程A的资源导致死锁。

***性能危险***
多线程中，线程时常需要挂起，这会带来巨大的系统开销。

***非争取同步修复***
+	不共享变量
+	变量设置为不可变
+	使用 synchronized


***原子性***
原子操作是不会导致线程危机的。如果将`i++`加入代码中，那么它会是非线程安全的。因为，`i++`包含了读写存三个操作。

![原子性](/image/java-bf/yzcz.png)


> 在并发编程中，需要关注如何在不可控的并发中保护数据、保持运行。
无状态对象永远是线程安全；无状态是指没有可变的全局变量。


# 二、线程安全

1. this引用构造时逸出

对象引用在当前范围之外的代码使用称之为发布；对象定义后没有被初始化就被`发布`称之为逸出。this引用构造时逸出，构造器代码没有运行结束，即便部分属性已经生成，也是不允许、不提倡的。

```
class Memento {
    private String state;
    public Memento(String state)
    {
        System.out.println(state);
        this.state=state;
    }
}
```

2. 线程安全的几个方法

可以同通过使用线程封闭技术实现线程安全，即不共享变量
+	Ad-hoc 线程限制，是一种规范，由开发者实现，约束差
+	栈封闭 使用Thread Local（非ThreadLocal类），全部使用局部变量
+	使用ThreadLocal类
+	使用不可变对象(final)，对于不变的变量应当尽可能使用它修饰


3. 发布模式

安全发布：
+	![安全发布模式](/image/java-bf/aqfbms.png)

高效不可变对象：
+	对象发布后在逻辑上不需要进行修改，但是可变的。

发布模式：
+	![发布模式](/image/java-bf/fbms.png)

4. 安全共享

![安全共享](/image/java-bf/aqgx.png)




### synchronized

使用synchronized避免多线程同时间访问同一数据

synchronized 是JAVA同步机制中的关键字，是独占锁、互斥锁。可以用于备注方法、代码段。

原子操作中的lock和unlock操作，对应JVM中的monitorenter和monitorexit指令。monitorenter和monitorexit指令是进入和退出同步块执行的指令，synchronized关键字底层的字节码指令中就包含了这两个命令，所以它可以锁住代码块。

***特点***
+	原子锁
+	内存可见性
```
// 方法锁
public synchronized Memento getMemento()
{
  return memento;
}


//代码段锁
synchronized (MementoPattern.class) {
  System.out.println("初始状态:"+or.getState());
  cr.setMemento(or.createMemento()); //保存状态
  or.setState("S1");
}

```

***重进入机制***

![重进入机制](/image/java-bf/xccjr.png)

***用锁保护状态***

并非*写入*共享变量的时候才需要同步。我们可以在对象内部为共享变量添加锁。而对于涉及多个状态的不变约束，需要用同一个锁进行保护。

![锁保护](/image/java-bf/sbh.png)


### 共享对象（volatile）

***重排序问题***

重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段。只要重排序保证了不会影响本线程的结果，就不能保证它的操作按照程序的既定顺序运行——即便重排序会影响其它线程的结果。指令重排序会导致内存的可见性问题。

特点：
+	数据依赖性（针对单个处理器而已） 带写操作的数据具有依赖性，不会随意调整代码顺序。
+	as-if-serial 不管怎么重排序，（单线程）程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。
+	happens-before   如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。



***非原子的64位***

![非原子的64位](/image/java-bf/fyz64.png)

> 访问共享变量式，要求多线程由同一个锁进行同步，也是为了避免读取到过期数据。



***volatile的使用***

![volatile](/image/java-bf/volatile.png)

`volatile`的典型应用之一就是声明状态标记，例如

```
    volatile boolean flag = true;
    @Test
    public void test() {
        if (flag) {
            System.out.println(flag);
            flag = false;
            // 业务代码
        }
    }
```

![volatile](/image/java-bf/volatile2.png)


> `volatile`不可以过度使用。例如，它不可以用于标注递增数（i++），因为它不能使自增操作原子化。
*加锁可以保证原子性与可见性，而使用 volatile只能保证可见性，无法使操作原子化。*


### final

final关键字意为不可改变，类不可继承、方法不可重写、属性不可修改。对于集合对象声明为final指的是引用不能被更改，但是你可以向其中增加，删除或者改变内容。


例如：
```
//初始化后不能被修改
final String str ="STR"; 

//方法不可重写
public final void finalMethod() {
    System.out.println("finalMathod");
}
//类不可继承
final class FinalClass {
}
```

对于final域，编译器和处理器要遵守两个重排序规则：

+	在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序（编译器会在final域的写之后，插入一个StoreStore屏障）。在对象引用为任意线程可见之前，对象的final域已经被正确的初始化过了。
+	初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序（编译器会在读final域操作的前面插入一个LoadLoad屏障）。只有得到了包含final域对象的引用，才能后读到final域的值


### 其它

1. wait、notify

wait()、notify/notifyAll() 方法是Object的本地final方法，无法被重写。wait()使当前线程阻塞，前提是 必须先获得锁，一般配合synchronized 关键字使用，即，一般在synchronized 同步代码块里使用 wait()、notify/notifyAll() 方法。

+	wait()方法。当线程执行wait()方法时候，会释放当前的锁，然后让出CPU，进入等待状态。
+	notify/notifyAll()方法。只有当 notify/notifyAll() 被执行时候，才会唤醒一个或多个正处于等待状态的线程，然后继续往下执行，直到执行完synchronized 代码块的代码或是中途遇到wait() ，再次释放锁。

2. Lock、Atomic、ThreadLocal

Lock接口提供了与synchronized相似的同步功能，和synchronized（隐式的获取和释放锁，主要体现在线程进入同步代码块之前需要获取锁退出同步代码块需要释放锁）不同的是，Lock在使用的时候是显示的获取和释放锁。虽然Lock接口缺少了synchronized隐式获取释放锁的便捷性，但是对于锁的操作具有更强的可操作性、可控制性以及提供可中断操作和超时获取锁等机制。Lock接口如果不释放锁，那么会触发异常。

Atomic包是Java.util.concurrent下的另一个专门为线程安全设计的Java包，包含多个原子操作类。这个包里面提供了一组原子变量类。

ThreadLocal是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于 各个线程依赖不通的变量值完成操作的场景。

# 三、多线程实现方法


1. 同步线程（在执行完任务之后无法获取执行结果）

+	通过继承Thread类，重写Thread的run()方法，将线程运行的逻辑放在其中
+	通过实现Runnable接口，实例化Thread类

2. 异步

Callable位于java.util.concurrent包下，它也是一个接口，在它里面也只声明了一个方法，只不过这个方法叫做call()：
```
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

Callable 意为可回调的，一般是和ExecutorService配合来使用的。

2. Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

Future类位于java.util.concurrent包下，它是一个接口。

3. ExecutorService 是一个线程池的一个接口类，用于创建线程池。


# 四、参考
> [JAVA并发理解之重排序问题](https://blog.csdn.net/ym123456677/article/details/79700623)
