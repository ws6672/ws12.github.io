---
title: Java 并发（四）线程池
date: 2021-01-15 21:18:02
tags: [并发]
---

线程池通过重复利用已创建的线程降低线程创建和销毁所造成的消耗，通过提前创建线程来提高执行任务的效率。

# 一、Callable/Future接口

1. Callable和Future

自从Java 1.5开始，就提供了Callable和Future接口，通过它们可以在任务执行完毕之后得到任务执行结果。位于java.util.concurrent包，源码如下：

```
// Callable 执行任务，产生结果
@FunctionalInterface
public interface Callable<V> {
	// 
    V call() throws Exception;
}

// Future 获取异步任务结果
public interface Future<V> {
	// 取消任务
    boolean cancel(boolean mayInterruptIfRunning);
	// 测试任务是否取消
    boolean isCancelled();
	// 测试任务是否执行成功
    boolean isDone();
	// 获取任务结果
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

在并发包中，还存在RunnableFuture 接口，该接口支持同步和异步；还存在ScheduledFuture接口，该接口是定时的异步任务。

FutureTask类实现了RunnableFuture接口, 相关源码如下：
```
// FutureTask 异步任务
public class FutureTask<V> implements RunnableFuture<V> {
	public FutureTask(Callable<V> callable) {
	}
	public FutureTask(Runnable runnable, V result) {
	}
}

// RunnableFuture支持同步也支持异步
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

实例：
```
public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CallTask callTask = new CallTask();
        FutureTask futureTask = new FutureTask(callTask);
        ExecutorService executor = Executors.newCachedThreadPool();
        executor.submit(futureTask);
        executor.shutdown();

        while (futureTask.isDone()) {
            Thread.sleep(1000);
        }
        System.out.println(futureTask.get());
    }
}

class CallTask implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        System.out.println("子进程计算");
        Thread.sleep(1000);
        int sum=0;
        for (int i = 0; i < 100; i++) {
            sum+=i;
        }
        return sum;
    }
}
```


2. future相关类图如下：

![future相关类图](/image/java/cn/Future.png)


# 二、ThreadFatory

ThreadFatory是基于工厂模式的线程工程接口，它有一个实现类CustomizableThreadFactory(Spring框架)。

相关源码如下：
```
public interface ThreadFactory {
    Thread newThread(Runnable r);
}
//标准的线程工程
public class CustomizableThreadFactory extends CustomizableThreadCreator implements ThreadFactory {
    public CustomizableThreadFactory() {
    }

    public CustomizableThreadFactory(String threadNamePrefix) {
        super(threadNamePrefix);
    }

    public Thread newThread(Runnable runnable) {
        return this.createThread(runnable);
    }
}
```

实例：
```
ThreadFactory springThreadFactory = new CustomizableThreadFactory("springThread-pool-");

ExecutorService exec = new ThreadPoolExecutor(1, 1,
		0L, TimeUnit.MILLISECONDS,
		new LinkedBlockingQueue<Runnable>(10),springThreadFactory);
exec.submit(() -> {
	logger.info("TEST");
});


```

# 三、线程池

1. ExecutorService

相关源码如下：
```
public interface ExecutorService extends Executor {

//	只是将空闲的线程 interrupt() 了，shutdown（）之前提交的任务可以继续执行直到结束
    void shutdown();
//	interrupt 所有线程， 因此大部分线程将立刻被中断。之所以是大部分，而不是全部，是因为 interrupt()方法能力有限
    List<Runnable> shutdownNow();
//	是否停止
    boolean isShutdown();
//	若关闭后所有任务都已完成，则返回true。注意除非首先调用shutdown或shutdownNow，否则isTerminated永不为true。
    boolean isTerminated();
	
//	awaitTermination 使用之前 必须先手动关闭线程池，否则一直会阻塞到超时为止
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
		
//	提交任务
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
	
//	将调用你在集合中传给 ExecutorService 的所有 Callable 对象。invokeAll() 返回一系列的 Future 对象，通过它们你可以获取每个 Callable 的执行结果
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;
		
//	invokeAny(…)方法要求一系列的 Callable 或者其子接口的实例对象。调用这个方法并不会返回一个 Future，但它返回其中一个 Callable 对象的结果。无法保证返回的是哪个 Callable 的结果 – 只能表明其中一个已执行结束 
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```		

+	ExecutorService的实现
	+	ThreadPoolExecutor
	+	ScheduledThreadPoolExecutor
+	ExecutorService 创建
	+	Executors（不推荐）
	+	ThreadPoolExecutor、ScheduledThreadPoolExecutor



2. Executors


Executors提供了四种创建线程池的方法：
+	固定大小的线程池
	+	固定大小的线程池 newFixedThreadPool 
	+	单任务线程池 newSingleThreadExecutor 
+	可变尺寸的线程池 newCachedThreadPool
+	延迟连接池 newScheduledThreadPool 


```
public class ExecutorsTest {
    public static void main (String[] args) {
        ExecutorService fp = Executors.newFixedThreadPool(3);
        Future future = fp.submit(new Runnable() {
            @Override
            public void run() {
                System.out.println("1. 创建固定大小的线程池");
                System.out.println("submit 执行线程，有返回值");
            }
        });
        System.out.println(future.isCancelled());
        fp.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("execute 执行线程，无返回值");
            }
        });
        fp.shutdown();

        ExecutorService cp = Executors.newCachedThreadPool();
        cp.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("2. 创建可缓存（扩容）线程池");
            }
        });
        cp.shutdown();

        ExecutorService sgp = Executors.newSingleThreadExecutor();
        sgp.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("3. 创建单任务线程池");
            }
        });
        sgp.shutdown();

        ScheduledExecutorService sp = Executors.newScheduledThreadPool(3);
        sp.schedule(new Runnable() {
            @Override
            public void run() {
                System.out.println("4. 创建定时线程池(设置延迟3秒)");
            }
        }, 3, TimeUnit.MILLISECONDS);
        sp.shutdown();
    }
}
```

> 在java 的多线程中，一但线程关闭，就会成为死线程。关闭后死线程就没有办法在启动了。再次启动就会出现异常信息：Exception in thread "main" java.lang.IllegalThreadStateException。那么如何解决这个问题呢？我们这里就可以使用 Executors.newSingleThreadExecutor()来再次启动一个线程.


通过`ExecutorService`异步执行任务：
+	通过`ExecutorService`的`execute`方法调用一个新线程，异步执行任务。一个线程将一个任务委派给一个 ExecutorService 去异步执行。一旦该线程将任务委派给 ExecutorService，该线程将继续它自己的执行，独立于该任务的执行。

```
    public static void main(String[] args) {
//        初始化线程池
        ExecutorService pool = Executors.newCachedThreadPool();

        pool.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("test:"+Thread.currentThread().getName());
            }
        });
//        销毁线程池
        pool.shutdown();
    }
```

3. 不推荐使用 Executors

过度使用Executors创建线程，会导致OOM；底层是通过ThreadPoolExecutor来创建，只是进行了基本配置。

```
// 其中，有new一个阻塞队列 LinkedBlockingQueue
public class Executors {
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
}
// 如下，通过构造器创建了Integer.MAX_VALUE大小的队列，会堆积大量的请求，从而造成OOM（内存溢出）
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
}
```

> 结论：推荐通过`ThreadPoolExecutor`手动创建线程池，参数设置则可以参考`Executors`




