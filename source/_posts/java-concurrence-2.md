---
title: java 并发（三）数据的并发
date: 2021-01-14 16:39:12
tags: [并发]
---

# 一、Atomic

1. 简介

Atomic包是Java.util.concurrent下的另一个专门为线程安全设计的Java包，包含多个原子操作类。这个包里面提供了一组原子变量类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性。

2. 自旋锁

自旋锁是一种非阻塞锁,抢到执行权的线程并不会自旋,自旋的精髓在于没抢到执行权的线程,它们会空转cpu,一直循环,这就是自旋,并非把线程改为阻塞状态.它们还是在运行的,自旋重试想获取锁。

在JDK1.4.2的时候就引入了自旋锁，到了JDK1.6以后，就已经是默认开启。自定义自旋锁如下：
```
public class AtomicTest {
    private AtomicReference<Thread> atomicReference = new AtomicReference<>();
    public void lock() {
        Thread thread = Thread.currentThread();
        while (!atomicReference.compareAndSet(null,thread)) {
            //当ref为null的时候compareAndSet返回true，反之为false
//            通过循环的自旋锁判断是否是其它线程持有的
        }
    }
    public void unlock() {
        Thread thread = Thread.currentThread();
        if (atomicReference.get()!=null) {

        }
        atomicReference.set(null);
    }
    static int count =0;
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(20);
        AtomicTest atomicTest = new AtomicTest();
        for (int i=0; i<100; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    atomicTest.lock();
                    System.out.println(count++);
                    atomicTest.unlock();
                }
            });
        }

    }

}
// 输出按序
```

自适应自旋锁：随着JDK的更新，在1.6的时候，又出现了一个叫做“自适应自旋锁”的玩意。它的出现使得自旋操作变得聪明起来，不再跟之前一样死板。所谓的“自适应”意味着对于同一个锁对象，线程的自旋时间是根据上一个持有该锁的线程的自旋时间以及状态来确定的。

3. CAS机制
CAS 是实现自旋锁的基础，CAS 利用 CPU 指令保证了操作的原子性，以达到锁的效果。加锁或使用 synchronized 关键字带来的性能损耗较大，而用 CAS 可以实现乐观锁，它实际上是直接利用了 CPU 层面的指令，所以性能很高。自旋是指循环，如果有多个线程进入自旋，那么性能会大大降低。

CAS,compare and swap的缩写，中文翻译成比较并交换。CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。 `如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值 。否则，处理器不做任何操作。`无论哪种情况，它都会在 CAS 指令之前返回该位置的值。


通常将 CAS 用于同步的方式是从地址 V 读取值 A，执行多步计算来获得新 值 B，然后使用 CAS 将 V 的值从 A 改为 B。如果 V 处的值尚未同时更改，则 CAS 操作成功。Java的CAS会使用现代处理器上提供的高效机器级别原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键。

递归程序使用自旋锁应该遵循以下原则：
+	递归程序决不能在持有自旋锁时调用它自己
+	决不能在递归调用时试图获得相同的自旋锁。

4. compareAndSet

```
	// compareAndSet这个方法主要调用unsafe.compareAndSwapInt这个方法，这个方法有四个参数，其中第一个参数为需要改变的对象，第二个为偏移量(即之前求出来的valueOffset的值)，第三个参数为期待的值，第四个为更新后的值。
    public final boolean compareAndSet(V expect, V update) {
        return unsafe.compareAndSwapObject(this, valueOffset, expect, update);
    }
```

5. Atomic 相关原子类
atomic类是基于CAS机制操作volatile遍历实现的，可用于保证基本数据类型的原子性。Atomic中包含了多个基本类型的原子类，具体如下：
+	基本类型
	+	AtomicInteger：整型原子类
	+	AtomicLong：长整型原子类
	+	AtomicBoolean ：布尔型原子类
+	引用类型
	+	AtomicReference：引用类型原子类
	+	AtomicMarkableReference：原子更新带有标记的引用类型
	+	AtomicStampedReference ：原子更新带有版本号的引用类型
+	数组类型
	+	AtomicIntegerArray：整型数组原子类
	+	AtomicLongArray：长整型数组原子类
	+	AtomicReferenceArray ：引用类型数组原子类
+	原子字段更新器
	+	AtomicIntegerFieldUpdater：原子更新整型字段的更新器
	+	AtomicLongFieldUpdater： 原子更新长整型字段的更新器
	+	AtomicReferenceFieldUpdater：原子更新引用类型里的字段的更新器

在JDK1.6之前，synchroized是重量级锁，即操作被锁的变量前就对对象加锁，不管此对象会不会产生资源竞争。这属于悲观锁的一种实现方式。　而CAS会比较内存中对象和当前对象的值是否相同，相同的话才会更新内存中的值，不同的话便会返回失败。这是乐观锁的一中实现方式。这种方式就避免了直接使用内核状态的重量级锁。



# 二、并发集合

1. 同步集与并发集合

分类：

+	同步集合类
	+	Hashtable
	+	Vector
	+	同步集合包装类，Collections.synchronizedMap()和Collections.synchronizedList() 
+	并发集合类
	+	ConcurrentHashMap
	+	CopyOnWriteArrayList
	+	CopyOnWriteHashSet
	

区别：
+	同步集合比并发集合会慢得多，因为是通过锁（sychronized）实现的。

2. ConcurrentHashMap

主要就是为了应对hashmap在并发环境下不安全而诞生的，ConcurrentHashMap的设计与实现非常精巧，大量的利用了volatile，final，CAS等lock-free技术来减少锁竞争对于性能的影响。对于重要部分，ConcurrentHashMap进行了加锁；对于其它数据，支持多线程访问。


+	JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8采用CAS+Synchronized保证线程安全。
+	锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）。
+	链表转化为红黑树：定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。
+	查询时间复杂度：从原来的遍历链表O(n)，变成遍历红黑树O(logN)。


3. CopyOnWriteArrayList 与 CopyOnWriteHashSet

Copy-On-Write简称COW，是一种用于程序设计中的优化策略。其基本思路是，从一开始大家都在共享同一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后再改，这是一种延时懒惰策略。Java并发包离提供了两个基于COW机制的并发容器。

+	读写分离
	+	COW即写时复制的容器，当向容器中添加新元素时，不会直接操作容器，而是会对容器复制。在向新容器中添加元素后，会重新将原容器的引用引向新的容器。通过COW机制，我们可以进行并发的读而不需要加锁，因为当前容器不会添加元素。所以，COW实际上是基于读写分离构建的。
+	场景
	+	CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景。
+	特点
	+	内存占用问题。COW机制是写时复制机制，用空间换取时间，实现并发，但是会占用一倍的内存空间。
	+	数据一致性问题。COW机制只能保证数据的最终一致性，无法保证数据的实时一致性。

# 三、锁

锁相关类图如下：
![Lock](/image/java/cn/lock.png)
![Lock](/image/java/cn/lock2.png)

1. 锁的相关概念

+	可重入锁：如果锁具备可重入性，则称作为 可重入锁 。像 synchronized和ReentrantLock都是可重入锁，而可重入锁是基于线程的分配，而不是基于方法调用的分配。在获取锁之后，同一个线程不需要重新申请锁。
+	可中断锁：可中断锁就是可以响应中断的锁。在Java中，synchronized就不是可中断锁，而Lock是可中断锁。
+	公平锁：公平锁即尽量按请求锁的顺序来获取锁，线程等待的时间越长越快获得锁；而非公平锁则无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。而对于ReentrantLock 和 ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为公平锁。



2. AbstractQueuedSynchronizer（AQS 抽象队列同步器）

AQS使用一个整型的volatile变量（命名为state）来维护同步状态，这是接下来实现大部分同步需求的基础，还提供了一个基于FIFO队列，可以用于构建锁或者其他相关同步装置的基础框架。AQS（AbstractQueuedSynchronizer）是一个用于构建锁和同步器的框架，许多同步器都可以通过AQS很容易并且高效地构造出来。不仅ReentrantLock和Semaphore是基于AQS构建的，还包括CountDownLatch、ReentrantReadWriteLock、SynchronousQueue和FutureTask。

```
public abstract class AbstractQueuedSynchronizer
	extends AbstractOwnableSynchronizer
    implements java.io.Serializable 
```

该同步器利用了一个int来表示状态，期望它能够成为实现大部分同步需求的基础。使用的方法是继承，子类通过继承同步器并需要实现它的方法来管理其状态，管理的方式就是通过类似acquire和release的方式来操纵状态。


***等待队列***

同步器核心之一便是等待队列，结构如下：
![compareAndSetTail](/image/current/compareAndSetTail.png)

该队列的节点定义如下：

```
static final class Node {
	//首节点
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;
    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;
	
    //表示节点的状态。默认为0，表示当前节点在sync队列中，等待着获取锁。
    //其它几个状态为：CANCELLED、SIGNAL、CONDITION、PROPAGATE
    volatile int waitStatus;
	
    //前驱节点
    volatile Node prev;
    //后继节点
    volatile Node next;
    //获取锁的线程
    volatile Thread thread;
    //存储condition队列中的后继节点。
    Node nextWaiter;
    ......
}

waitStatus的几个状态

CANCELLED(1) : 由于 timeout/interrupt, 线程被取消
0            : 其他状态
SIGNAL(-1)   : 线程成功被阻塞,可以调用当前结点的后续结点
CONDITION(-2): 位于条件队列中，等待condition
PROPAGATE(-3): 表示当前场景下后续的acquireShared能够得以执行

```



该等待队列是“CLH（自旋锁）”锁队列，用于阻塞同步器，使用了相同的策略来控制当前线程的上一个线程的信息。每个节点包含一个状态字段，用于区分该线程是否会被阻塞，而不保证线程获得锁。CLH队列需要一个虚拟头节点才能启动。

***几个核心方法***

```
java.util.concurrent.locks.AbstractQueuedSynchronizer.getState()
java.util.concurrent.locks.AbstractQueuedSynchronizer.setState(int)
java.util.concurrent.locks.AbstractQueuedSynchronizer.compareAndSetState(int, int)

//compareAndSetState是一个基于CAS原则定义的方法，用于设置队列的尾节点
private final boolean compareAndSetTail(Node expect, Node update) {
	return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
```



3. Lock（接口）和 ReentrantLock（可重入锁）

Lock（接口）相关方法如下：
```
package java.util.concurrent.locks;
import java.util.concurrent.TimeUnit;

public interface Lock {
// 获取锁
    void lock();
// 如果当前线程未被中断，则获取锁，可以响应中断
    void lockInterruptibly() throws InterruptedException;
// 仅在调用时锁为空闲状态才获取该锁，可以响应中断
    boolean tryLock();
// 如果锁在给定的等待时间内空闲，并且当前线程未被中断，则获取锁
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
// 释放锁
    void unlock();
// 返回绑定到此 Lock 实例的新 Condition 实例
    Condition newCondition();
}
```

ReentrantLock 意为可重入锁，是Lock接口的实现类。ReentrantLock是唯一实现了Lock接口的类，并且提供了更多的方法。实例如下：

```
public class LockTest {

    public void testLock(String value) {
        Lock lock = new ReentrantLock();
        lock.lock();
        try {
            //处理任务
            System.out.println(value);
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
//            释放锁，需要放在第一行
            lock.unlock();
        }
    }

    public void testTryLock(String value) {
        Lock lock = new ReentrantLock();

        if (lock.tryLock()) {
            try {
                //处理任务
                System.out.println(value);
            }  catch (Exception e) {
                e.printStackTrace();
            }  finally {
//            释放锁，需要放在第一行
                lock.unlock();
            }
        } else {
            System.out.println("don't have lock");
        }
    }

    public static void main(String[] args) {
        LockTest lockTest = new LockTest();
        new Thread(()->lockTest.testLock("A")).start();
        new Thread(()->lockTest.testLock("B")).start();
    }
}
```

3. ReadWriteLock 和 ReentrantReadWriteLock

ReadWriteLock是读写锁接口，源码如下。ReadWriteLock管理一组锁，一个是只读的锁，一个是写锁；ReentrantLock是一个排他锁，同一时间只允许一个线程访问。

```
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```

***ReetrantReadWriteLock***

Java并发库中ReetrantReadWriteLock实现了ReadWriteLock接口并添加了可重入的特性。

而ReentrantReadWriteLock允许多个读线程同时访问，但不允许写线程和读线程、写线程和写线程同时访问。相对于排他锁，提高了并发性。在实际应用中，大部分情况下对共享数据（如缓存）的访问都是读操作远多于写操作，这时ReentrantReadWriteLock能够提供比排他锁更好的并发性和吞吐量。

ReentrantReadWriteLock支持以下功能
+	支持公平和非公平的获取锁的方式
+	支持可重入
+	允许从写入锁降级为读取锁，获取写锁后可以获取读锁，最后释放写锁；如果先获取读锁，则不允许升级为写锁。
+	支持锁中断
+	Condition支持

相关实例如下：
```
public class RRWTest {
    Map<String,Integer> map = new HashMap<>();
    ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    Lock rl = readWriteLock.readLock();
    Lock wl = readWriteLock.writeLock();

    public static void main (String[] args) {

    }
    public Integer get(String key) {
        Integer res = null;
        rl.lock();
        try {
            res = map.get(key);
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            rl.unlock();
        }
        return res;
    }
    public void put(String key, Integer value) {
        wl.lock();
        try {
            map.put(key, value);
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            wl.unlock();
        }
    }
    public void clear() {
        wl.lock();
        try {
            map.clear();
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            wl.unlock();
        }
    }

}
```

4. Condition

Condition类似于条件队列，当一个线程在调用了await方法以后，直到线程等待的某个条件为真的时候才会被唤醒。这种方式为线程提供了更加简单的等待/通知模式。Condition必须要配合锁一起使用，因为对共享状态变量的访问发生在多线程环境下。一个Condition的实例必须与一个Lock绑定，因此Condition一般都是作为Lock的内部实现。

相关源码如下：
```
public interface Condition {
//	在接到信号或被中断之前一直处于等待状态
    void await() throws InterruptedException;
    boolean await(long time, TimeUnit unit) throws InterruptedException;
	
//	造成当前线程在接到信号之前一直处于等待状态
    void awaitUninterruptibly();
//	造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态
    long awaitNanos(long nanosTimeout) throws InterruptedException;
//	造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false。
    boolean awaitUntil(Date deadline) throws InterruptedException;
//	唤醒等待线程
    void signal();
    void signalAll();
}

```

Condition是AQS的内部类。每个Condition对象都包含一个队列(等待队列)。等待队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，该线程就是在Condition对象上等待的线程，如果一个线程调用了Condition.await()方法，那么该线程将会释放锁、构造成节点加入等待队列并进入等待状态。


5. LockSupport

LockSupport是JDK中比较底层的类，用来创建锁和其他同步工具类的基本线程阻塞原语。LockSupport很类似于二元信号量(只有1个许可证可供使用)，如果这个许可还没有被占用，当前线程获取许可并继续执行；如果许可已经被占用，当前线程阻塞，等待获取许可。

