---
title: java 并发（二）线程安全策略
date: 2020-04-12 10:11:08
tags: [并发]
---


## 一、几个概念

+	同步策略：定义了对象如何协调对其状态的访问， 并且不会违背它的不变约束或后验条件。通过使用final可以缩小需要判断的变量数量，以确定是否为一个状态。
+	
+	不变约束：是指变量间关系的不变约束，例如存在`b=a*a`,那么在多次计算中，都需要成立。
+	先验条件：针对方法，规定了在条用方法之前必须为真的条件 if,while
+	后验条件：针对方法，规定了在条用方法之后必须为真的条件 do-while
+	原子操作：任务在执行过程中不能被打断的一序列操作
+	复合操作：任务在执行过程中可以被打断的一序列操作
+	一致性（Consistency）：是指多副本（Replications）问题中的数据一致性。可以分为强一致性、顺序一致性与弱一致性。
	+	强一致性（Strict Consistency）：也称为原子一致性（Atomic Consistency）或线性一致性（Linearizable Consistency）。要求：任何一次读都能读到某个数据的最近一次写的数据；系统中的所有进程，看到的操作顺序，都和全局时钟下的顺序一致。
	+	顺序一致性（Sequential Consistency）：任何一次读都能读到某个数据的最近一次写的数据；系统的所有进程的顺序一致，而且是合理的。即不需要和全局时钟下的顺序一致，错的话一起错，对的话一起对。
	+	弱一致性：数据更新后，如果能容忍后续的访问只能访问到部分或者全部访问不到，则是弱一致性。



### 二、组合对象

***设计线程安全类***

![设计线程安全类](/image/java-bf/bfzhdx.png)

类的不变约束和方法的后验条件约束了对象的合法状态和合法状态转换。

我们可以通过`wait`和`notify`为操作设置先验条件，但更提倡使用封装好的类进行处理。例如，阻塞队列（blocking queue）或信号量（semaphore）以及其他同步工具（Synchronizer）。

![封装权、所有权](/image/java-bf/fzqysyq.png)

> 把数据封装在对象内部，把对数据的访问限制在对象的方法上（getter、setter），更易确保线程在访问数据时总能获得正确的锁。即只能通过对象获取数据，只要保证它在对象内部是线程安全的，那么同理在系统中也是安全的。


***java监视器模式***

在JAVA虚拟机中，每个对象(Object和class)通过某种逻辑关联监视器，为了实现监视器的互斥功能，每个对象(Object和class)都关联着一个锁(有时也叫“互斥量”)，这个锁在操作系统书籍中称为“信号量”，互斥(“mutex “)是一个二进制的信号量。

```
// 私有锁-监视器模式
public class PrivateLock {
//    信号量
    private final Object myLock = new Object();
    void visite() {
        synchronized (myLock) {
            //业务代码
        }
    }
}
```


***委托线程安全***

如下所示，方法中关联着`ConcurrentHashMap`，这是一个线程安全的类，委托给支持线程安全的内部组件实现线程安全

```
public class WebTimer {
    ConcurrentHashMap<String,Date> hashMap = new ConcurrentHashMap<String, Date>();
    public boolean add (String name, Date date) {
        if (date == null) {
            return false;
        }
        hashMap.replace(name, date);
        return true;
    }
}
```

![委托线程安全](/image/java-bf/aqwt.png)

如果一个状态变量是线程安全的，没有任何不变约束限制它的值，并且没有任何状态转换限制它的操作，那么他可以安全发布。


***客户端加锁***
我们可以通过线程安全类实现线程安全，但如果我们想在这些类中进行拓展，增加新的功能，那么就需要保证子类与对象拥有相同的锁。但有些源码中的状态是很难拿到的。那么，我们可以通过第三方类来拓展功能，然后通过客户端加锁实现线程安全。

![客户端加锁](/image/java-bf/khdjs.png)

更加健硕的方法，是通过组合实现。我们不再直接使用List，而是通过一个新的类进行访问，并在类上加锁。虽然类内部的list不一定是线程安全的，但我们只能通过该类访问List。

![组合](/image/java-bf/zhxcaq.png)

> 线程安全策略建议写入文档，以便维护与更新


### 三、构建块

***同步容器的线程安全问题***

同步容器类常见的有 Vector 和HashTable，在jdk1.2添加了同步包装类（Wrapper）,这些类都是由Collections.synchronizedXxx工厂建立，保证了原子操作的线程安全。

源码如下，可以看出 add方法与 size方法都有加方法锁，在原子操作中是安全的，但是复合操作就不是线程安全的。
```
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
	...
	public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
    public synchronized int size() {
        return elementCount;
    }
	...
```

线程安全测试,同时使用`add`和`size`方法，结果并非线程安全的。
```
public class VectorTest extends Thread {
    Vector<Integer> vector;
    public VectorTest(Vector<Integer> vector) {
        this.vector = vector;
    }
    @Override
    public void run() {
        while (vector.size() != 50) {
            try {
                vector.add(0);
                Thread.sleep((long) (3000*Math.random()));
                System.out.println(this.getName()+":" + vector.size());
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        Vector<Integer> vec=new Vector<Integer>();
        for (int i=0;i<10;i++) {
            new VectorTest(vec).start();
        }
    }
}

```

同步容器也会有线程安全问题，这种容器对于原子操作来说，是线程安全的。但是，对于复合操作来说，是非线程安全的。例如，如果要确保`Vector`在复合操作的时候线程安全，那么就需要在客户端对`Vector`加锁。


![tbfz](/image/java-bf/tbfz.png)


> `Vector`没有解决线程的安全问题。
对集合的遍历一般使用迭代器，但在多线程的环境下，迭代期间被修改，会抛出`CurrentModificationException`的异常。`CurrentModificationException`也可能出现在单线程中，当用户没有通过iterator.remove()删除元素的时候。


***并发容器***

并发容器是为了多线程并发访问而设计的，jdk5提供了几种并发容器来改进同步容器。
+	可以使用 ConcurrentHashMap 代替同步的哈希实现
+	当多数为读取操作的时候，可以使用 CopyOnWriteArrayList 代替List的同步实现
+	当多数为读取操作的时候，可以使用 CopyOnWriteArraySet 代替Set的同步实现
+	Queue-先进先出
	+	ConcurrentLinkedQueue 
	+  	BlockingQueue -阻塞队列（可阻塞插入、读取操作）
+	使用ConcurrentSkipListMap 代替 SortedMap
+	使用ConcurrentSkipListSet 代替 SortedSet

`ConcurrentHashMap`使用分离锁作为锁机制，改进了同步容器类，它返回的迭代器具有弱一致性，无需添加锁。弱一致性的迭代器可以并发修改，迭代器创建后遍历时可以但不保证感应到容器修改。所以，即使客户端加锁，ConcurrentHashMap也无法保证复合操作是原子性。

`CopyOnWriteArrayList`是同步List的替代品，会在写入时复制一个不可变对象，由于是不可变的,不会有线程安全问题；而当需要修改时，会复制一个新的容器拷贝，以实现可变性。由于修改会触发容器拷贝，在容器过大的时候，开销也会过大。


***生产者-消费者 模式***

生产者消费者模式并不是GOF提出的23种设计模式之一，23种设计模式都是建立在面向对象的基础之上的，但其实面向过程的编程中也有很多高效的编程模式，生产者消费者模式便是其中之一，它是我们编程过程中最常用的一种设计模式。

在软件实际开发中，有时候会存在某个模块负责产生数据，另一个模块负责处理数据。生产数据的可以称之为生产者，而对数据进行处理的可以称为消费者；而在生产者、消费者之间一般存在一个缓冲区，用于临时存储数据，提高数据吞吐量。

缓冲区的优点如下：
+	解耦
+	支持并发
+	调节忙闲不均

```

```



***阻塞队列***

![zsdl](/image/java-bf/zsdl.png)

 阻塞队列支持生产者-消费者模式，该模式分离了“识别需要完成的工作”和“执行工作”
 
![有界队列](/image/java-bf/yjdl.png)

***双端队列和窃取工作***
JDK6添加了Deque和BlockingDeque, 双端队列，支持头尾读取、添加；支持窃取工作模式，在消费者处理完自己的双端任务，可以去其它消费者的双端尾部处理数据。


***中断***
![中断](/image/java-bf/zd.png)

当代码中抛出`InterruptedException`时，该方法也会成为中断方法。需要做好响应中断的准备：
+	传递 将异常传递给调用者
+	恢复中断 当代码是进程的一部分，需要捕获异常，调用`interrupt`从中断中恢复。如果既不捕获也不抛出，会导致中断发生时难以找到问题所在。


实例（恢复中断）：
```
@Override
public void run() {
	while (vector.size() != 50) {
		try {
			vector.add(0);
			Thread.sleep((long) (3000*Math.random()));
			System.out.println(this.getName()+":" + vector.size());
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			// 恢复中断状态
		  Thread.currentThread().interrupt();
		}
	}
}
```

### Synchronizer

![Synchronizer](/image/java-bf/Synchronizer.png)


***闭锁***

`闭锁`是一种`Synchronizer`，用于延迟线程进度知道线程终止，用于确保特定获得等到其它活动完成后才发生：
+	确保计算不会执行
+	确保服务不会开始
+	等待，确保相关联资源到位

`闭锁`是一次性使用的对象；一旦进入到最终状态，就不能被重置了。

***FutureTask***

FutureTask 可以作为闭锁，等价于可携带结果的`Runnable`,包含三个状态：
+	等待
+	运行
+	完成
	+	正常结束
	+	取消
	+	异常

 

***信号量（semaphore）***

![信号量](/image/java-bf/xhl.png)


***关卡***
`关卡` 类似于闭锁，能阻塞一组进程，但所有进程必须同时到达关卡点，才能继续处理。而闭锁等待的是事件，而关卡等待的是其它线程。

`CyclicBarrier` 就是一个与关卡概念相关的类。


