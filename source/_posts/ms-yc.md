---
title: 面试记录（五）某车
date: 2020-11-02 11:43:12
tags: [ms]
---

### 一、SQL优化

1. 导致索引失效的原因

+	在索引列上进行操作（计算、函数、类型转换），会导致索引失效而转向全表扫描
+	不遵从最左前缀匹配原则
+	不遵从全值匹配原则
+	mysql在使用不等于（!=或者<>）的时候，无法使用索引会导致全表扫描
	+	在oracle中，可以通过decode函数判空则给默认值1，从而避免索引失效
+	like以通配符开头（`%abc...`）mysql索引失效会变成全表扫描的操作
+	字符串不加单引号索引失效
+	用`or`来连接时索引会失效


2. 索引类型


+	聚集索引(主键索引)
	+	聚集索引就是按照每张表的主键构造一颗B+树，同时叶子节点中存放的即为整张表的记录数据。聚集索引的叶子节点称为数据页，聚集索引的这个特性决定了索引组织表中的数据也是索引的一部分。
+	辅助索引(二级索引)
	+	非主键索引，叶子节点=键值+书签。Innodb存储引擎的书签就是相应行数据的主键索引值。
+	覆盖索引理解
	+	select查询字段都被设置成索引，无需从表里获取。不是所有类型的索引都可以成为覆盖索引。覆盖索引必须要存储索引的列，而哈希索引、空间索引和全文索引等都不存储索引列的值，所以MySQL只能使用B-Tree索引做覆盖索引。


3. SQL优化方法

+	对查询的优化，尽量避免全表查询，对查询频率高的字段建立索引
+	避免在where字句中对null值进行判断，会导致索引失效进而进行全表扫描。
+	应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描
+	应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描
+	in 和 not in 也要慎用，否则会导致全表扫描
+	应尽量避免在 where 子句中对字段进行表达式操作、函数操作，否则将导致引擎放弃使用索引而进行全表扫描
+	不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。  
+	尽量用 exists 代替 in 
+	尽量使用数字型字段

### 二、缓存

1. 缓存穿透
缓存穿透是指缓存和数据库中都没有某种数据，而用户基于不存在的数据不断进行请求，导致服务器压力过大

解决方案：
+	接口校验，直接过滤不合理的值
+	数据库没有的数据，缓存（key-value）中value设置为null，设置短暂的存活时间


2. 缓存击穿
缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

解决方案：
+	设置热点数据永远不过期。
+	加互斥锁

3. 缓存雪崩
缓存雪崩是指缓存中大批量数据同一时间过期，而查询数据量巨大，引起数据库压力过大甚至宕机。

解决方案：
+	设置随机过期时间，避免同时间过期
+	热点数据均匀分布到集群
+	热点数据永不过期

### 三、多线程

1. 同步和异步的区别

+	同步是把所有的操作都执行完成，得出最终结果服务端再返回给用户，期间客户端无法进行其他工作
+	异步是当客户端发送给服务端请求时，在等待服务端响应的时候，客户端可以做其他的事情，这样节约了时间，提高了效率。

2. Java实现同步实现与异步实现

同步实现
+	ThreadLocal  保证不同线程拥有不同实例，相同线程一定拥有相同的实例，用于隔离变量
+	synchronized( )  同步代码
+	wait() 与 notify() 阻塞和释放
+	volatile
	+	volatile 修饰的变量不会保留副本，而是会相互共享数据。如果变量被声明为volatile，在每次访问时都会和主存一致
	+	Volatile 变量具有 synchronized 的可见性特性，但是不具备原子特性。volatile 保证了变量的安全性，无法保证线程的安全性
	+	volatile 要使线程安全有两个要求：
		+	对变量的写操作不依赖于当前值
		+	该变量没有包含在具有其他变量的不变式中

异步实现
+	FutureTask 实现Callable接口，通过子类创建FutureTask对象，executor提交
	+	Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。
	+	```
		public class Task implements Callable<Integer>{
			
			@Override
			public Integer call() throws Exception {
				System.out.println("子线程在进行计算");
				Thread.sleep(3000);
				int sum = 0;
				for(int i=0;i<100;i++)
					sum += i;
				return sum;
			}

		}
		
		ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        executor.submit(futureTask);
        executor.shutdown();
		
		Task task = new Task();
		FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
		Thread thread = new Thread(futureTask);
		thread.start();
		
		if(futureTask.get()!=null){  
			System.out.println("task运行结果"+futureTask.get());
		}else{
			System.out.println("future.get()未获取到结果"); 
		}
	```

3. java synchronized原理

每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：
+	如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。
+	如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.
+	如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。


4. AQS源码

AbstractQueuedSynchronizer（AQS 抽象队列同步器）：CAS操作+volatile关键字+改造的CLH队列
+	使用整型的volatile变量（命名为state）来维护同步状态
+	存在一个FIFO的 等待队列


### 四、笔试题

1. java集合那些是有序的？
+	List 接口的集合类，比如ArrayList、LinkedList
+	LinkedHashMap
+	ConcurrentSkipListMap
+	TreeMap
+	TreeSet

2. redis的key和value大小限制？
redis的key和string类型value限制均为512MB

3. hashmap原理？
底层由数组和链表组成，数组被分为一个个桶（bucket），通过哈希算法决定哈希桶的位置；为了解决哈希碰撞的问题，使用了红黑树。当节点超过8个，自动转换为红黑树。

4. 如何查询11到30条记录id自增不连续?

```
-- ORACLE
select * from employees e where rownum betwon 11 and 30;

-- MySQL
select top 20 * from (select top 30 ID from A order by ID) as T1 order by T1.ID desc
```

5. 左模糊，右模糊，全模糊，哪个可以用索引？

+	左模糊、全模糊不用索引
+	右模糊会使用索引

6. 消息队列优缺点和使用场景是什么
优点是：解耦、异步、削峰
缺点是：系统可用性降低、复杂性提高、可能有数据一致性的问题
常用的使用场景：异步处理，应用解耦，流量削锋和消息通讯四个场景。

7. 2个文件都是10万条数据，取出相同的数据？
使用hyperloglog

8. 遇到运行时异常，你的思路？
使用二分法定位错误代码

