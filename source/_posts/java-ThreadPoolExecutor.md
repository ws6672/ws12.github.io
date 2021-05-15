---
title: Java 并发（五）自定义线程池
date: 2021-01-16 16:05:24
tags: [并发]
---

在Java并发包中提供了线程池工厂类Executors，它提供了不同类型的线程池，底层都是通过ThreadPoolExecutor类创建的。所以，我们可以通过ThreadPoolExecutor来创建线程。

# 一、线程池状态

+	workerCount，指示工作线程个数；工作线程是指已被允许启动且未被允许停止的线程；workerCount限制为（2 ^ 29）-1（约5亿个）线程，而不是（2 ^ 31）-1（20亿个）可表示的线程。
+	runState，指示是否正在运行，正在关闭等。
+	在ctl中，runState由高3位表示；workerCount由ctl的低29位表示。如果代码中会有超过5百多万线程（536870911）加入池中并允许开始执行，就要注意溢出的风险。

相关源码如下：
```
	//runState相关参数如下
	private static final int COUNT_BITS = Integer.SIZE - 3;
	private static final int RUNNING    = -1 << COUNT_BITS;		//接受新任务并处理排队的任务
	private static final int SHUTDOWN   =  0 << COUNT_BITS;		//不接受新任务，但处理排队的任务
	private static final int STOP       =  1 << COUNT_BITS;		//不接受新任务，不接受处理排队的任务
	private static final int TIDYING    =  2 <<COUNT_BITS;		//任务均已终止，workerCount=0，线程转换为TIDYING状态,将运行Terminate（）
	private static final int TERMINATED =  3 << COUNT_BITS;		//Terminate（）已完成

	// 获取runState，即保留ctl的高3位，后29位置0
    private static int runStateOf(int c)     {
      return c & ~CAPACITY;
    }

    //获取workerCount，即保留ctl的低29位，高3位置0
    private static int workerCountOf(int c)  { 
      return c & CAPACITY;
    }
   
    //设置ctl,或操作
    private static int ctlOf(int rs, int wc) {
      return rs | wc;
    }
```

runState 状态转换如下：
+	调用shutdown（）时，RUNNING -> SHUTDOWN
+	在调用shutdownNow（）时，SHUTDOWN -> TIDYING
+	队列和线程池为空，STOP -> TIDYING
+	当Terminate（）挂钩方法完成时，TIDYING -> TERMINATED


# 二、线程参数

1. 相关参数

+	corePoolSize：核心线程池大小。
+	maximumPoolSize：最大线程池大小；线程池中的当前线程数目不会超过该值。如果正在运行的线程个数多于核心线程数，但小于最大线程数，则仅当队列已满时才创建新线程。
+	keepAliveTime：线程最大空闲时间；如果一个线程处在空闲状态的时间超过了该属性值，就会因为超时而退出。
+	unit：这个是描述存活时间的时间单位。可以使用TimeUnit里边的枚举值。
+	workQueue：代表阻塞队列，存储所有等待执行的任务。
+	threadFactory：代表用来创建线程的工厂。可以自定义一个工厂，传参进来。如果不指定的话，就会使用默认工厂（Executors类里边的 DefaultThreadFactory）。
+	handler：参数代表拒绝策略。当阻塞队列和线程池都满了，即达到了最大线程数，决定了会用什么策略来处理。
	+	AbortPolicy（默认）：直接拒绝，并抛出异常，这也是默认的策略。
	+	DiscardPolicy：直接丢弃当前任务，但是不抛异常。
	+	CallerRunsPolicy：直接让调用execute方法的线程去执行此任务。
	+	DiscardOldestPolicy：丢弃最老的未处理的任务，然后重新尝试执行当前的新任务。
+	拒绝任务的场景如下：
	+	当线程数已经达到maxPoolSize，阻塞队列已满，会拒绝新任务
	+	当线程池被调用shutdown()后，会等待线程池里的任务执行完毕，再shutdown。如果在调用shutdown()和线程池真正shutdown之间提交任务，会拒绝新任务
	+	线程池会调用rejectedExecutionHandler来处理这个任务。如果没有设置默认是AbortPolicy，会抛出异常；实现RejectedExecutionHandler接口，可自定义处理器。

2. 线程池的执行过程

+	当线程数量未达到corePoolSize的时候，就会创建新的线程来执行任务。
+	当核心线程数已满，就会把任务放到阻塞队列。
+	当队列已满
	+	未达到最大线程数，就会新建非核心线程来执行任务。
	+	达到了最大线程数，则选择一种拒绝策略来执行。


3. 线程池常用方法

+	execute 方法来提交任务给线程池
+	submit 方法用来提交任务，并且可以获取返回值
+	shutdown方法用来关闭线程池
+	shutdownNow也会关闭线程池。但是，它不再接受新任务，并且会尝试终止正在运行的任务


4. Executors 定义线程池

```
	// corePoolSize和maximumPoolSize设置为相同，可以创建固定大小的线程池。当线程数量达到核心线程数时，新任务就会放到阻塞队列里边等待执行。
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
	
	// corePoolSize和maximumPoolSize设置为1，可以创建单任务线程池。若线程空闲则执行，否则把任务放到阻塞队列。
	public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
	
	// 如果使用SynchronousQueue队列，可以创建可缓存（扩容）线程池。创建一个可根据实际情况调整线程个数的线程池。这句话，可以理解为，有多少任务同时进来，就会创建同等数量的线程去执行任务。当然，这是在线程数不能超过Integer最大值的前提下。
	public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
	
	// 如果使用DelayedWorkQueue延时队列，可以创建延迟连接池
	public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```

5. 设置参数


> ```
     * 需要根据几个值来决定
            - tasks ：每秒的任务数，假设为500~1000
            - taskcost：每个任务花费时间，假设为0.1s
            - responsetime：系统允许容忍的最大响应时间，假设为1s
        * 做几个计算
            - corePoolSize = 每秒需要多少个线程处理？ 
                * threadcount = tasks/(1/taskcost) =tasks*taskcout =  (500~1000)*0.1 = 50~100 个线程。corePoolSize设置应该大于50
                * 根据8020原则，如果80%的每秒任务数小于800，那么corePoolSize设置为80即可
            - queueCapacity = (coreSizePool/taskcost)*responsetime
                * 计算可得 queueCapacity = 80/0.1*1 = 80。意思是队列里的线程可以等待1s，超过了的需要新开线程来执行
                * 切记不能设置为Integer.MAX_VALUE，这样队列会很大，线程数只会保持在corePoolSize大小，当任务陡增时，不能新开线程来执行，响应时间会随之陡增。
            - maxPoolSize = (max(tasks)- queueCapacity)/(1/taskcost)
                * 计算可得 maxPoolSize = (1000-80)/10 = 92
                * （最大任务数-队列容量）/每个线程每秒处理能力 = 最大线程数
            - rejectedExecutionHandler：根据具体情况来决定，任务不重要可丢弃，任务重要则要利用一些缓冲机制来处理
            - keepAliveTime和allowCoreThreadTimeout采用默认通常能满足
```

# 三、BlockingQueue和Worker

1. BlockingQueue

BlockingQueue 源码如下，用于存储任务

```
private final BlockingQueue<Runnable> workQueue;

// 阻塞队列
public interface BlockingQueue<E> extends Queue<E> {
	// 添加元素，无空间抛出异常
    boolean add(E e);
	
	// 添加元素，无空间返回false（不阻塞）
    boolean offer(E e);
    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;

	// 如果队列满了，一直阻塞，直到队列不满了或者线程被中断
    void put(E e) throws InterruptedException;


	// 如果队列空了，一直阻塞，直到队列不为空或者线程被中断-->阻塞
    E take() throws InterruptedException;
	// 如果队列不空，出队；如果队列已空且已经超时，返回null
    E poll(long timeout, TimeUnit unit) throws InterruptedException;

	// 返回此队列可以理想的（在没有内存或资源限制的情况下）无阻塞接受的其他元素的数量；如果没有内部限制，则返回Integer.MAX_VALUE
    int remainingCapacity();

    boolean remove(Object o);

    public boolean contains(Object o);

	// 从此队列中删除所有可用的元素，并将它们*添加到给定的集合中。此操作可能比重复轮询此队列更有效
    int drainTo(Collection<? super E> c);
    int drainTo(Collection<? super E> c, int maxElements);
}
```

相关实现类：

+	ArrayBlockingQueue 基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象
+	LinkedBlockingQueue 基于链表的阻塞队列，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回
+	DelayQueue　DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素
+	PriorityBlockingQueue  基于优先级的阻塞队列
+	SynchronousQueue 一种无缓冲的等待队列。有两种不同的方式，即公平模式和非公平模式。
	+	如果采用公平模式：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，从而体系整体的公平策略；
	+	但如果是非公平模式（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时配合一个LIFO队列来管理多余的生产者和消费者

2. Worker

Worker类是ThreadPoolExecutor中定义的内部类，其代码如下；一个线程就是一个Worker对象,它与一个线程绑定,当Worker执行完毕就是线程执行完毕

```
private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        
        private static final long serialVersionUID = 6138294804551838833L;

        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

        
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
        public void run() {
            runWorker(this);
        }

        // Lock methods
        //
        // The value 0 represents the unlocked state.
        // The value 1 represents the locked state.

        protected boolean isHeldExclusively() {
            return getState() != 0;
        }

        protected boolean tryAcquire(int unused) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        protected boolean tryRelease(int unused) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        public void lock()        { acquire(1); }
        public boolean tryLock()  { return tryAcquire(1); }
        public void unlock()      { release(1); }
        public boolean isLocked() { return isHeldExclusively(); }

        void interruptIfStarted() {
            Thread t;
            if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                }
            }
        }
    }
```

Worker继承了AbstractQueuedSynchronizer，锁的粒度细化到每个Worker；直接使用CAS获取，避免阻塞。

Worker实现了Runnable接口，有thead、firstTask属性:
+	thead 执行任务的线程，通过ThreadFactory获取线程
+	firstTask 待执行任务

# 四、相关文章
> [JAVA ThreadPoolExecutor线程池参数设置技巧](https://www.imooc.com/article/5887)