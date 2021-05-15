---
title: Java 并发（六）线程工具
date: 2021-01-18 15:29:56
tags: [并发]
---



# 一、ForkJoinPool（分治线程池）

1. 概述
> ForkJoinPool的优势在于，可以充分利用多cpu，多核cpu的优势，把一个任务拆分成多个“小任务”，把多个“小任务”放到多个处理器核心上并行执行；当多个“小任务”执行完成之后，再将这些执行结果合并起来即可。

ForkJoinPool提供了一个并发处理“分而治之”的框架，让我们能以类似于递归的编程方式获得并发执行的能力。所谓“分而治之“是理清思路和解决问题的一个重要的方法。大到系统架构对功能模块的拆分，小到归并排序的实现，无一不在散发着分而治之的思想。在实现分而治之的算法的时候，我们通常使用递归的方法。递归相当于把大的任务拆成多个小的任务，然后大任务等待多个小的子任务执行完成后，合并子任务的结果。



2. 类图

![ForkJoinPool](/image/java/cn/ForkJoinPool.png)

ForkJoinPool是ExecutorService的实现类，因此是一种特殊的线程池。创建了ForkJoinPool实例之后，就可以调用ForkJoinPool的submit(ForkJoinTask<T> task) 或invoke(ForkJoinTask<T> task)方法来执行指定任务了。

其中ForkJoinTask代表一个可以并行、合并的任务。ForkJoinTask是一个抽象类，它还有两个抽象子类：
+	RecusiveTask 代表有返回值的任务
+	RecusiveAction 代表没有返回值的任务。


实例：

```
public class ExecutorsTest {
    public static void main (String[] args) {
        ExecutorsTest.forkJoinPool();
    }
    public static void forkJoinPool(){
        ForkJoinPool forkJoinPool=null;
        try {
            forkJoinPool = new ForkJoinPool();
            Future<Integer> future = forkJoinPool.submit(new MyTask(1,1000));
            System.out.println("result:"+future.get());
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            if(forkJoinPool!=null) {
                forkJoinPool.shutdown();
            }

        }

    }
   
}

class MyTask extends RecursiveTask<Integer> {
    private final static int NUMS = 100;
    private int st;
    private int end;
    public MyTask(int st,int end){
        this.st=st;
        this.end=end;
    }
    @Override
    protected Integer compute() {
        int sum=0;
        if (end-st<=NUMS) {
            for(int i=st; i<=end;i++) {
                sum+=i;
            }
            System.out.println(Thread.currentThread()+":"+sum);
            return sum;
        } else {
            int md=(end+st)/2;
            MyTask lf = new MyTask(st, md);
            MyTask rt = new MyTask(md+1, end);
            lf.fork();
            rt.fork();
            return lf.join()+rt.join();
        }

    }
}
```

ForkJoinPool 主要用来使用分治法(Divide-and-Conquer Algorithm)来解决问题。典型的应用比如快速排序算法。ThreadPoolExecutor虽然也可以执行多个任务，但是它无法在任务中添加子任务，等待子任务完成后再完成该任务。而使用ForkJoinPool时，就能够让其中的线程创建新的任务，并挂起当前的任务，此时线程就能够从队列中选择子任务执行。

ForkJoinPool分治的关键是fork()和join()方法，fork()用于拆分任务，而join()方法用于合并任务。在ForkJoinPool使用的线程中，会使用一个内部队列来对需要执行的任务以及子任务进行操作来保证它们的执行顺序。但是，ForkJoinPool在分治时会创建大量子任务，消耗过多的系统资源。

# 二、线程计数器

线程中存在计数器，用于设置任务的执行顺序。当子任务执行完成后，线程才能执行。countDownLatch与CyclicBarrier是在java1.5被引入，作为线程计数器。

1. CountDownLatch

CountDownLatch这个类使一个线程等待其他线程各自执行完毕后再执行。CountDownLatch是通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。



CountDownLatch相关源码如下。使用过程如下：
+	创建CountDownLatch对象，需要指定线程的数量
	+	`CountDownLatch countDownLatch = new CountDownLatch(3);`
+	每当一个线程完成自己的任务后，计数器的值就会减 1 。
	+	`countDownLatch.countDown();`
+	当计数器的值变为0时，就表示所有的线程均已经完成了任务，然后就可以恢复等待的线程继续执行了。

```
//	线程计数器
public class CountDownLatch {
	。。。
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    private final Sync sync;

	//	使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    //	线程递减
    public void countDown() {
        sync.releaseShared(1);
    }
	
	
	//	获取线程数
    public long getCount() {
        return sync.getCount();
    }

}

```


CountDownLatch 的内部类Sync继承了AQS，通过CAS实现同步锁，通过同步锁实现CountDownLatch的实现类。

```
//	AQS
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
		// CAS
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }
```

相关实例如下：
```
public class ExecutorsTest {
    public static void main (String[] args) {
        ExecutorsTest.countDownLatch();
    }
	/* 
		输出如下：
			等待线程完成
			Thread[pool-1-thread-2,5,main]
			Thread[pool-1-thread-1,5,main]
			主线程完成:Thread[main,5,main]
	*/

    public static void countDownLatch() {
        CountDownLatch latch = new CountDownLatch(2);
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                    System.out.println(Thread.currentThread());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    latch.countDown();
                }
            }
        });
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    latch.countDown();
                }
            }
        });
        executorService.shutdown();
        try {
            System.out.println("等待线程完成");
            latch.await();
            System.out.println("主线程完成:"+Thread.currentThread());
        }  catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```


2. CyclicBarrier

CyclicBarrier意为可循环利用的屏障，作用是让线程等待，所有子线程都完成后才会继续下一步行动。CyclicBarrier可以用于多线程计算数据，最后合并计算结果的场景。

相关方法：
```
// 构造方法
// parties  是线程个数
// barrierAction 最后一个到达线程要执行的任务
public CyclicBarrier(int parties)
public CyclicBarrier(int parties, Runnable barrierAction)

//  await() 表示已到达栅栏
//	BrokenBarrierException  表示栅格被破坏，即其它线程await被中断或延时
public int await() throws InterruptedException, BrokenBarrierException
public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException
```

使用方法如下：
```
public class ExecutorsTest {
    public static void main (String[] args) {
        ExecutorsTest.cyclicBarrier();
    }

    public static void cyclicBarrier(){
        CyclicBarrier barrier = new CyclicBarrier(2, new Runnable() {
            @Override
            public void run() {
                System.out.println("线程全部到位");
            }
        });
        CBThread cbThread = new CBThread(barrier);
        cbThread.start();
        CBThread cbThread1 = new CBThread(barrier);
        cbThread1.start();
    }
}

class CBThread extends Thread {

    CyclicBarrier barrier;
    public CBThread(CyclicBarrier barrier) {
        this.barrier = barrier;
    }
    @Override
    public void run() {
        super.run();
        try {
            Thread.sleep(20);
            System.out.println(getName()+"抵达栅格1");
            barrier.await();	//等待其它线程抵达
            System.out.println(getName()+"冲破栅格1");

            System.out.println(getName()+"抵达栅格2");
            Thread.sleep(100);
            barrier.await();
            System.out.println(getName()+"冲破栅格2");
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}
```

CountDownLatch和CyclicBarrier区别：
+	countDownLatch是一个计数器，线程完成一个记录一个，计数器递减，只能只用一次
+	CyclicBarrier的计数器更像一个阀门，需要所有线程都到达，然后继续执行，计数器递增，提供reset功能，可以多次使用


# 三、Exchanger（线程通讯）

Exchanger 是 JDK 1.5 开始提供的一个用于两个工作线程之间交换数据的封装工具类，简单说就是一个线程在完成一定的事务后想与另一个线程交换数据，则第一个先拿出数据的线程会一直等待第二个线程，直到第二个线程拿着数据到来时才能彼此交换对应数据。

使用方法：
```
public class ExecutorsTest {
    public static void main (String[] args) {
        ExecutorsTest.exchanger();
    }
    public static void exchanger() {
		/*
		输出：
        Thread-1 交换前：38
        Thread-0 交换前：55
        Thread-1 交换后：55
        Thread-0 交换后：38
		*/
        Exchanger exchanger = new Exchanger();
        ExchangerTest ex = new ExchangerTest(exchanger);
        ExchangerTest ex1 = new ExchangerTest(exchanger);
        ex.start();
        ex1.start();
    }
    
}

class ExchangerTest extends Thread {
    private Exchanger exchanger;
    public ExchangerTest(Exchanger exchanger) {
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        int data = (int) (Math.random()*1000);
        try {
            System.out.println(getName()+" 交换前："+data);
            data = (int) exchanger.exchange(data);
            System.out.println(getName()+" 交换后："+data);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```


# 四、Semaphore（信号量）

Semaphore称之为信号量，可以控制访问资源的线程数。Semaphore是一个线程同步的辅助类，可以维护当前访问自身的线程个数，并提供了同步机制。使用Semaphore可以控制同时访问资源的线程个数，例如，实现一个文件允许的并发访问数。


Semaphore的主要方法：
```
acquire()：获取一个令牌，在获取到令牌、或者被其他线程调用中断之前线程一直处于阻塞状态。
acquire(int permits)：获取一个令牌，在获取到令牌、或者被其他线程调用中断、或超时之前线程一直处于阻塞状态。
acquireUninterruptibly()：获取一个令牌，在获取到令牌之前线程一直处于阻塞状态（忽略中断）。
    
tryAcquire()：尝试获得令牌，返回获取令牌成功或失败，不阻塞线程
tryAcquire(long timeout, TimeUnit unit)：尝试获得令牌，在超时时间内循环尝试获取，直到尝试获取成功或超时返回，不阻塞线程。
​
release()：释放一个令牌，唤醒一个获取令牌不成功的阻塞线程。
​
hasQueuedThreads()：等待队列里是否还存在等待线程。
getQueueLength()：获取等待队列里阻塞的线程数。
​
drainPermits()：清空令牌把可用令牌数置为0，返回清空令牌的数量。
​
availablePermits()：返回可用的令牌数量。
```

使用场景：
+	数据库连接池限制
+	选课限流

使用方法如下：
```
public class ExecutorsTest {
    public static void main (String[] args) {
        ExecutorsTest.semaphore();
    }
    public static void semaphore() {
        Semaphore semaphore = new Semaphore(4);
        for (int i=0; i<30; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {

                    try {
						// 剩余令牌为0
                        if (semaphore.availablePermits()==0) {
                            System.out.println(Thread.currentThread() + "进入失败，网站访问量过多");
                        }
						// 两秒内尝试获取令牌
                        semaphore.tryAcquire(2,TimeUnit.SECONDS);
                        System.out.println(Thread.currentThread()+" 进入选课");
                        Thread.sleep(new Random().nextInt(5000));
                        System.out.println(Thread.currentThread()+" 离开选课");
						// 释放令牌
                        semaphore.release();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
            thread.start();
        }
    }
}
```

# 五、参考文档
> [Fork/Join框架基本使用](https://blog.csdn.net/tyrroo/article/details/81390202)
[深入理解CyclicBarrier原理](https://blog.csdn.net/qq_39241239/article/details/87030142)
[Semaphore](https://blog.csdn.net/longgeqiaojie304/article/details/91127730)
[CountDownLatch](https://blog.csdn.net/a303549861/article/details/90666982)
