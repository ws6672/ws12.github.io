---
title: redis（一）入门
date: 2020-09-24 11:01:19
tags: [Redis]
---

# 一、基础

Redis （Remote Dictionary Server）可以翻译为远程词典服务器，，是一个开源的使用ANSI- C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。顾名思义，它的存储形式是以字典形式存储结构，并且允许其他应用通过 TCP 协议进行读写操作。

Redis的架构包括两个部分：Redis Client和Redis Server。Redis客户端负责向服务器端发送请求并接受来自服务器端的响应。服务器端负责处理客户端请求，例如，存储数据，修改数据等。 


### 1.1 衡量Redis优劣

衡量一个数据库系统的优劣，可以从以下几个方面着手：
+	安全性
+	性能
+	可持续性
+	业务支撑

1. 安全性

Redis被设计成仅有可信环境下的可信用户才可以访问。这意味着将Redis实例直接暴露在网络上或者让不可信用户可以直接访问Redis的tcp端口或Unix套接字，是不安全的。正常情况下，使用Redis的web应用程序是将Redis作为数据库，缓存，消息系统，网站的前端用户将会查询Redis来生成页面，或者执行所请求的操作，或者被web应用程序用户所触发。总而言之，Redis并没有最大地去优化安全方面，而是尽最大可能去优化高性能和易用性。

2. 性能

+	Redis基于内存的数据库。数据被存储在内存中，不需要到硬盘读取。

```
	CPU上下文的切换大概在 1500ns 左右。
	从内存中读取 1MB 的连续数据，耗时大约为 250us，假设1MB的数据由多个线程读取了1000次，那么就有1000次时间上下文的切换，
	那么就有1500ns * 1000 = 1500us ,切换上下文的时间是读取数据消耗时间的6倍。
```
+	单线程。多线程的本质就是 CPU 模拟出来多个线程的情况，这种模拟出来的情况就有一个代价，就是上下文的切换数据库来说，它没有上下文的切换就是效率最高的。所以，Redis被设计成单线程。CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。
+	使用多路I/O复用模型（连接重复使用），非阻塞IO（不能立刻得到调用结果之前，函数不会阻塞当前线程）

3. 可持续性

Redis 将数据存储在内存中，重启后数据消失。为了保证数据的可持续性，Redis提供RDB（默认持久化方式，定时存储数据）、AOF（定时存储命令）两种方式来持久化数据。

4. 业务支撑

+	热点数据的缓存(redis速度块、支持的数据类型比较丰富、expire)
+	计数器相关问题（incrby）
+	排行榜（SortedSet）
+	分布式锁（setnx）
+	延时操作（Pub/Sub、expire）
+	分页、模糊搜索（ZRANGEBYLEX）
+	队列（list push、list pop）


> [Redis为什么是单线程](https://cloud.tencent.com/developer/article/1120615)
[Redis的性能瓶颈](https://www.cnblogs.com/qwangxiao/p/8535202.html)

---

# 二、使用

相关资源：
[redis 官网](https://redis.io/)
[redis 文档](http://redisdoc.com/)
[window版的Redis v3.0.503](https://github.com/ServiceStack/redis-windows)


### 2.1 通过Vagrant载Redis

Vagrant是一个基于Ruby的工具，用于创建和部署虚拟化开发环境。它 使用Oracle的开源VirtualBox虚拟化系统，使用 Chef创建自动化虚拟环境。

1. 在Windows上安装Vagrant
2. 下载vagrant-redis.zip vagrant配置
3. vagrant-redis.zip在任何文件夹中提取，例如在c:\vagrant-redis
4. 启动Virtual Box VM vagrant up

```
cd c:\vagrant-redis
vagrant up
```

### 2.2 Redis配置与使用

1. 修改配置文件 `redis.windows.conf`

```
	# maxmemory <bytes>
	# 设置大约1GB
	maxmemory 1024000000

	# requirepass foobared
	# 设置密码
	requirepass 111
	# 设置端口
	port 6379
```

2. 通过命令启动

```
	cd e:/util/redis
	e:
	# 服务端
	
	redis-server.exe redis.windows.conf

	# 客户端
	redis-cli.exe

	# 添加【redis】服务
	redis-server --service-install redis.windows.conf --port 6379
	# 卸载【redist】服务
	redis-server --service-uninstall --service-name Redis
	# 启动服务
	redis-server --service-start
	# 停止服务
	redis-server --service-stop
```

3. Redis React——视图化工具

Redis React 可以非常便捷的查询键值对，也可以输入命令。[下载](https://github.com/ServiceStackApps/RedisReact#download),打开后即可使用。

4. 为快速登陆创建批处理文件

```
cd E:\Util\redis-latest
e:
redis-server.exe redis.windows.con
```

5. Redis无需密码
```
可以在redis.conf里设置:

	# requirepass foobared
	requirepass yourpassword

重新启动redis-server redis.conf
 
集群设置密码(修改所有redis集群中的redis.conf文件)
	masterauth 1234 
	requirepass 1234
```

### 2.3 Linux中下载与安装

0. Linux环境下安装redis报错structredisServer没有名为XXXX的成员

安装gcc套装
+	yum install cpp
+	yum install binutils
+	yum install glibc
+	yum install glibc-kernheaders
+	yum install glibc-common
+	yum install glibc-devel
+	yum install gcc
+	yum install make
升级gcc
+	yum -y install centos-release-scl
+	yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
+	scl enable devtoolset-9 bash

1. [下载](https://redis.io/)

2. 安装
```
>pscp C:\Users\wsz\Downloads\Compressed\redis-6.0.8.tar.gz cen@192.168.223.129:/home/cen

首先查看gcc-c++是否安装
	[root@localhost src]# gcc -v
	没有安装，则进入root：
		$ su
		$ yum install gcc-c++
进入下载包所在的文件；

解压：
	$ tar zxvf redis-4.0.1.tar.gz

进入解压文件中的src文件夹

编译：
	 
	[zws@localhost src]$ 
	tar zxvf redis-4.0.10.tar.gz -C /usr/local/
	[zws@localhost src]$ Cd /usr/local
	[zws@localhost src]$ cd  redis-4.0.10
	[zws@localhost src]$ make
	[zws@localhost src]$ cd  src
	[zws@localhost src]$ make install
```

3. 使用, 进入

```
	Cd redis-4.0.10
	Redis-server
	Reids-cli
```


### 2.4 数据结构

它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。

+	Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。




+	String（字符串）
	+	存放key/value存储
	+	set
	+	get
	+	decr （decrement）递减
		+	将数字格式的键减一
		+	如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 DECR 操作。
		+	本操作的值限制在 64 位(bit)有符号数字表示之内。
	+	incr （increment）递增
		+	将数字格式的键增一
		+	如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 incr 操作。
		+	本操作的值限制在 64 位(bit)有符号数字表示之内。

	+	mget
		+	返回给定的一个或多个字符串键的值。
		+	如果给定的字符串键里面， 有某个键不存在， 那么这个键的值将以特殊值 nil 表示。
+	Hash（散列）
	+	存放二维数据
	+	hset
		+	格式：HSET hash field value （eg: `hset image id 11` `插入一个hash表 key 为image,域为id，值为11的hash数据`）
		+	返回值 当 HSET 命令在哈希表中新创建 field 域并成功为它设置值时， 命令返回 1 ； 如果域 field 已经存在于哈希表， 并且 HSET 命令成功使用新值覆盖了它的旧值， 那么命令返回 0 。
	+	hget
		+	格式：HGET hash field （eg: `HGET image id` ）
		+	如果给定域不存在于哈希表中， 又或者给定的哈希表并不存在， 那么命令返回 nil 。
	+	hgetall
		+	格式：HGET hash （eg: `HGET image` ）
		+	返回哈希表key中，所有的域和值。
	+	HSETNX 不存在就添加，存在就放弃操作
		+	HSETNX hash field value
	+	HEXISTS 检测是否存在域
		+	HEXISTS hash field
	+	HDEL 删除域
		+	HDEL key field [field …]
+	LIST（列表）
	+	LPUSH	`LPUSH key value [value …]`
	+	LPUSHX	`LPUSHX key value`
		+	将值 value 插入到列表 key 的表头，当且仅当 key 存在并且是一个列表。
	+	RPUSH
		+	将一个或多个值 value 插入到列表 key 的表尾(最右边)。
	+	RPUSHX
		+	将一个或多个值 value 插入到列表 key 的表尾(最右边)，当且仅当 key 存在并且是一个列表。
	+	LPOP	`LPOP key`
		+	移除并返回列表 key 的头元素。
	+	RPOP
		+	移除并返回列表 key 的尾元素。
	+	RPOPLPUSH	`RPOPLPUSH source destination`
		+	这个命令执行两个操作，将`source`的尾元素移动到`destination`列表
	+	LREM `LREM key count value`
		+	根据参数 count 的值，移除列表中与参数 value 相等的元素。
		+	count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
		+	count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
		+	count = 0 : 移除表中所有与 value 相等的值。
	+	LLEN
	+	LINDEX	`LINDEX key index`
		+	返回列表 key 中，下标为 index 的元素。
	+	LINSERT
	+	LSET
	+	LRANGE
	+	LTRIM
	+	BLPOP
	+	BRPOP
	+	BRPOPLPUSH	
+	集合（sets）
+	有序集合（sorted sets） zsets
+	bitmaps
+	hyperloglogs
+	地理空间（geospatial）


***用途***

1. 会话存储(Session)
```
SET randomHash "{userId}" EX 60
```

2. 全页面缓存
```
SET key "<html>...</html>" EX 60
```

3. 顺序排序
```
ZADD sortedSet 1 "one"
```

4. 队列（电子邮件队列等）
```
HSET messages <id> <message>
```

5. pub/sub()
```
PUBLISH channel message;
SUBSCRIBE channel;
```

6. 分布式锁


---

### 2.5 在Windows上安装 Redis6

通过Cygwin3工具可以在WIN模拟Linux环境。[下载地址](https://cygwin.com/setup-x86_64.exe)

1. 一路默认选项即可，到一个页面有 User Url，添加`http://mirrors.aliyun.com/cygwin/`即可。
2. Select Pacakage, 额外添加 make，gcc-core，gcc-g++ libgcc1 libgccpp1
3. 一路安装直至完成即可

[下载redis源码](https://redis.io/),编译源码包

1. 把下好的源码包安装到`cygwin64\home\<用户名>`下，就可以通过`Cygwin.bat`访问模拟环境看到了
2. 解压，编译
```
tar -xvf redis-5.0.7.tar.gz
make & make install
```
3. 编译完成后，会生成很多`.exe`文件，把`.exe`文件、`redis.conf`、`cygwin64\bin\cygwin1.dll`复制到新文件中
4. 配置好redis.conf，即可获得一个可启动的客户端。


> [Redis6 Windows 版本编译](http://blog.pluskid.org/?p=366)

### 可视化工具（支持Redis6）
QuickRedis是永久免费的Redis Desktop管理器。它支持直接连接，标记和群集模式，支持多种语言，支持数亿个键，并具有出色的UI。同时支持Windows，Mac OS X和Linux平台。

相关资源：
[QuickOfficial](https://github.com/quick123official/quick_redis_blog/)
[QuickOfficial github](https://github.com/quick123official/quick_redis_blog/)

# 三、Jedis

1. [下载](https://mvnrepository.com/artifact/redis.clients/jedis/2.9.0)
	
	
2. 下载 [commons-poolx.x.x.jar  ——使用附加包](http://mvnrepository.com/artifact/org.apache.commons/commons-pool2) 连接的时候需开放端口或关闭防火墙(命令如下 sudo service iptables stop ,需要注释掉bind127.0.0.1 ，设置protected-mode为NO)


3. Maven使用
```
	<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
	<dependency>
	    <groupId>redis.clients</groupId>
	    <artifactId>jedis</artifactId>
	    <version>2.9.0</version>
	</dependency>
```

4. [相关api](http://xetorthio.github.io/jedis/)

### 3.1 Jedis 使用

1. Jedis 连接池

```

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class JPool {
    //最大连接总数
    private int maxTotal;
    //最大空闲数
    private int maxIdle;
    //连接池配置
    private JedisPoolConfig jpc;
    //连接池
    private JedisPool jp;
    //连接
    private Jedis j;
	
	//配置密码
//		jedis.auth("password_value");
	
    

    public JPool() {
        // TODO 自动生成的构造函数存根
    }
    
    public JPool(int maxTotal,int maxIdle) {
        // TODO 自动生成的构造函数存根
        this.maxTotal = maxTotal;
        this.maxIdle = maxIdle;
        init();
        
    }
    
    public void init() {
        jpc = new JedisPoolConfig();
        jpc.setMaxTotal(this.maxTotal);
        jpc.setMaxIdle(this.maxIdle);
        initProFile();
         
    }
    
    
    public void initProFile() {
        InputStream is = JPool.class.getClassLoader().getResourceAsStream("Redis.properties");
        
         Properties pro = new Properties();
        
        try {
            pro.load(is);
  
            String ip = pro.getProperty("IP");
             
            int port = Integer.parseInt(pro.getProperty("PORT"));
             
            this.jp = new JedisPool(jpc,ip,port);
            
        } catch (IOException e) {
            // TODO 自动生成的 catch 块
            e.printStackTrace();
        }
    }
    
    public Jedis getLink() {
        return jp.getResource();
    }
}

```

2. 配置文件（Redis.properties）
```
IP = 127.0.0.1
PORT = 6379
```

3. 测试

```
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPoolConfig;

public class TJ {
    public static void main(String[] args) {
        JPool jp = new JPool(30, 10);
        Jedis jedis = jp.getLink();
        //set 命令 保存
        jedis.set("Test", "This is a test");
                
        // get 命令 获取
        System.out.println(jedis.get("Test")); 
        
        jedis.close();
    }
}
```

4. 单节点

```
public class JedisTest {
 
	@Test
	public void testJedisSingle() {
		//创建一个jedis的对象。
		Jedis jedis = new Jedis("192.168.25.153", 6379);
		//调用jedis对象的方法，方法名称和redis的命令一致。
		jedis.set("key1", "jedis test");
		String string = jedis.get("key1");
		System.out.println(string);
		//关闭jedis。
		jedis.close();
	}
	
	/**
	 * 使用连接池
	 */
	@Test
	public void testJedisPool() {
		//创建jedis连接池
		JedisPool pool = new JedisPool("192.168.25.153", 6379);
		//从连接池中获得Jedis对象
		Jedis jedis = pool.getResource();
		String string = jedis.get("key1");
		System.out.println(string);
		//关闭jedis对象
		jedis.close();
		pool.close();
	}
}
```

5. 集群

```
@Test
	public void testJedisCluster() {
		HashSet<HostAndPort> nodes = new HashSet<>();
		nodes.add(new HostAndPort("192.168.25.153", 7001));
		nodes.add(new HostAndPort("192.168.25.153", 7002));
		nodes.add(new HostAndPort("192.168.25.153", 7003));
		nodes.add(new HostAndPort("192.168.25.153", 7004));
		nodes.add(new HostAndPort("192.168.25.153", 7005));
		nodes.add(new HostAndPort("192.168.25.153", 7006));
		
		JedisCluster cluster = new JedisCluster(nodes);
		
		cluster.set("key1", "1000");
		String string = cluster.get("key1");
		System.out.println(string);
		
		cluster.close();
	}
```

### 3.2 Jedis 相关方法

Jedis 相关使用方法如下：

```
Set<String> keys = jedis.keys("*"); //列出所有的key

Set<String> keys = jedis.keys("key"); //查找特定的key



//移除给定的一个或多个key,如果key不存在,则忽略该命令. 

jedis.del("key1");

jedis.del("key1","key2","key3","key4","key5");



//移除给定key的生存时间(设置这个key永不过期)
jedis.persist("key1"); 

//检查给定key是否存在
jedis.exists("key1"); 

//将key改名为newkey,当key和newkey相同或者key不存在时,返回一个错误
jedis.rename("key1", "key2");

//返回key所储存的值的类型。 
//none(key不存在),string(字符串),list(列表),set(集合),zset(有序集),hash(哈希表) 
jedis.type("key1");

//设置key生存时间，当key过期时，它会被自动删除。 
jedis.expire("key1", 5);//5秒过期 
 


//字符串值value关联到key。 
jedis.set("key1", "value1"); 

//将值value关联到key，并将key的生存时间设为seconds(秒)。 
jedis.setex("foo", 5, "haha"); 

//清空所有的key
jedis.flushAll();

//返回key的个数 
jedis.dbSize();

//哈希表key中的域field的值设为value。 
jedis.hset("key1", "field1", "field1-value"); 
jedis.hset("key1", "field2", "field2-value"); 

Map map = new HashMap(); 
map.put("field1", "field1-value"); 
map.put("field2", "field2-value"); 
jedis.hmset("key1", map); 


//返回哈希表key中给定域field的值 
jedis.hget("key1", "field1");

//返回哈希表key中给定域field的值(多个)
List list = jedis.hmget("key1","field1","field2"); 
for(int i=0;i<list.size();i++){ 
   System.out.println(list.get(i)); 
} 

//返回哈希表key中所有域和值
Map<String,String> map = jedis.hgetAll("key1"); 
for(Map.Entry entry: map.entrySet()) { 
   System.out.print(entry.getKey() + ":" + entry.getValue() + "\t"); 
} 

//删除哈希表key中的一个或多个指定域
jedis.hdel("key1", "field1");
jedis.hdel("key1", "field1","field2");

//查看哈希表key中，给定域field是否存在。 
jedis.hexists("key1", "field1");

//返回哈希表key中的所有域
jedis.hkeys("key1");

//返回哈希表key中的所有值
jedis.hvals("key1");



//将值value插入到列表key的表头。 
jedis.lpush("key1", "value1-0"); 
jedis.lpush("key1", "value1-1"); 
jedis.lpush("key1", "value1-2"); 

//返回列表key中指定区间内的元素,区间以偏移量start和stop指定.
//下标(index)参数start和stop从0开始;
//负数下标代表从后开始(-1表示列表的最后一个元素,-2表示列表的倒数第二个元素,以此类推)
List list = jedis.lrange("key1", 0, -1);//stop下标也在取值范围内(闭区间)
for(int i=0;i<list.size();i++){ 
   System.out.println(list.get(i)); 
} 

//返回列表key的长度。 
jedis.llen("key1")



//将member元素加入到集合key当中。 
jedis.sadd("key1", "value0"); 
jedis.sadd("key1", "value1"); 

//移除集合中的member元素。 
jedis.srem("key1", "value1"); 

//返回集合key中的所有成员。 
Set set = jedis.smembers("key1"); 

//判断元素是否是集合key的成员
jedis.sismember("key1", "value2")); 

//返回集合key的元素的数量
jedis.scard("key1");
 
//返回一个集合的全部成员，该集合是所有给定集合的交集
jedis.sinter("key1","key2")
 
//返回一个集合的全部成员，该集合是所有给定集合的并集
jedis.sunion("key1","key2")

//返回一个集合的全部成员，该集合是所有给定集合的差集
jedis.sdiff("key1","key2");

```

---


