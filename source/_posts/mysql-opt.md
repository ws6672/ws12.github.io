---
title: MySQL 优化(一)设计与功能优化
date: 2021-05-16 13:56:21
tags: [mysql]
---

MYSQL优化包括设计、功能、架构与高性能SQL几个方面。Mysql优化，一方面是找出系统的瓶颈，提高mysql数据库整体的性能，另外一个方面需要合理的结构设计和参数调整，以提高用户操作响应的速度。同时还要尽可能节省系统资源，以便系统可以提供更大负荷的服务。mysql数据库优化是多方面的，原则是减少系统的瓶颈，减少资源的占用，增加系统反应的速度。

+	设计：存储引擎，字段类型，范式与逆范式
+	功能：索引，缓存，分区分表
+	高性能SQL：SQL优化、explain
+	架构：主从复制，读写分离，负载均衡。

# 一、设计优化


### 存储引擎


存储引擎是一种用来存储MySQL中对象（记录和索引）的一种特定的结构（文件结构），处于MySQL服务器的最底层，直接存储数据

指定存储引擎：`Create table tableName () engine=myisam|innodb`


1. 查询支持的引擎

```
show engines;
````

|Engine|Support|Comment|Transactions|XA|Savepoints|
|:--|:--|:--|:--|:--|:--|
|MEMORY|YES|基于哈希的，存储在内存中，对临时表有用|NO|NO|NO|
|MRG_MYISAM|YES|与MyISAM表相同的集合|NO|NO|NO|
|CSV|YES|CSV存储引擎|NO|NO|NO|
|FEDERATED|NO|联合MySQL存储引擎||||
|PERFORMANCE_SCHEMA|YES|性能模式|NO|NO|NO|
|MyISAM|YES|MyISAM存储引擎|NO|NO|NO|
|InnoDB|DEFAULT|支持事务、行级锁定和外键|YES|YES|YES|
|BLACKHOLE|YES|/dev/null存储引擎（您向其写入的任何内容都将消失）|NO|NO|NO|
|ARCHIVE|YES|存档存储引擎|NO|NO|NO|

> 注：engine：引擎名称
suppot：MySQL数据库是否支持
comment：说明
transactions：是够支持事务
xa：是否支持XA事务（分布式事务）
savepoints：是否支持保存savepoints之间的内容

2. InnoDB存储引擎

Mysql版本>=5.5 默认的存储引擎，MySQL推荐使用的存储引擎。支持事务，行级锁定，外键约束。事务安全型存储引擎。更加注重数据的完整性和安全性。innodb擅长事务、数据的完整性及高并发处理，不擅长快速插入（插入前要排序，消耗时间）和检索


创建数据库表后生成如下文件：
+	db.opt存放了数据库的配置信息，比如数据库的字符集还有编码格式
+	XX.frm是表结构文件，仅存储了表的结构、元数据(meta)，包括表结构定义信息等；表引擎都会有一个frm文件。
+	XX.ibd是表索引文件，包括了单独一个表的数据及索引内容。



共享表空间
	+	Innodb的所有数据保存在一个单独的表空间里面，而这个表空间可以由很多个文件组成，一个表可以跨多个文件存在，所以其大小限制不再是文件大小的限制，而是其自身的限制。其表空间的最大限制为64TB，也就是说，Innodb的单表限制基本上也在64TB左右。
	+	优点：表空间可以分成多个文件存放到各个磁盘，所以表也就可以分成多个文件存放在磁盘上，表的大小不受磁盘大小的限制
	+	缺点：所有的数据和索引存放到一个文件，虽然可以把一个大文件分成多个小文件，但是多个表及索引在表空间中混合存储，当数据量非常大的时候，表做了大量删除操作后表空间中将会有大量的空隙，特别是对于统计分析，对于经常删除操作的这类应用最不适合用共享表空间；共享表空间分配后不能回缩：当出现临时建索引或是创建一个临时表的操作表空间扩大后，就是删除相关的表也没办法回缩那部分空间了。
独立表空间
	+	独立表空间是把每个表的数据和表文件放在一起，每表对应一个 idb文件
	+	优点：每个表都有自已独立的表空间，每个表的数据和索引都会存在自已的表空间中，可以实现单表在不同的数据库中移动。
	+	缺点：单表增加过大，当单表占用空间过大时，存储空间不足，只能从操作系统层面思考解决方法

InnoDB采用按表空间（tablespace)的方式进行存储数据, 默认配置情况下会有一个初始大小为10MB， 名字为ibdata1的文件， 该文件就是默认的表空间文件（tablespce file），用户可以通过参数innodb_data_file_path对其进行设置，可以有多个数据文件，如果没有设置innodb_file_per_table的话， 那些Innodb存储类型的表的数据都放在这个共享表空间中，而系统变量innodb_file_per_table=1的话，那么InnoDB存储引擎类型的表就会产生一个独立表空间，独立表空间的命名规则为：表名.idb. 这些单独的表空间文件仅存储该表的数据、索引和插入缓冲BITMAP等信息，其它信息还是存放在共享表空间中。


查看是否开启独立表空间：
```
show variables like 'innodb_file_per_table'
innodb_file_per_table	ON  
	OFF 代表mysql是共享表空间，也就是所有库的数据都存放在一个ibdate1文件中
	ON 表示mysql是独立表空间

```

开启：
```
在my.conf文件中[mysqld] 顶点下添加innodb_file_per_table=1

或者通过命令：set global innodb_file_per_table=1;
	innodb_file_per_table=1 为使用独占表空间
	innodb_file_per_table=0 为使用共享表空间
	innodb_data_home_dir = "C:\mysql\data\"

show variables like 'innodb_data_home_dir' --数据库文件所存放的目录
show variables like 'innodb_log_group_home_dir' --日志存放目录
show variables like 'innodb_data_file_path' --指定innodb 共享 表空间文件
	innodb_data_file_path	ibdata1:12M:autoextend（存储到 ibdata1 初始大小为12M 自动扩容）
```

> 注：InnoDB不创建目录，所以在启动服务器之前请确认”所配置的路径目录”的确存在。这对你配置的任何日志文件目录来说也是真实的。使用Unix或DOS的mkdir命令来创建任何必需的目录。通过把innodb_data_home_dir的值原原本本地部署到数据文件名，并在需要的地方添加斜杠或反斜杠，InnoDB为每个数据文件形成目录路径

3. MyISAM 存储引擎
MySQL<= 5.5 MySQL默认的存储引擎；擅长与处理，高速读与写。
特点：
+	数据和索引分别存储于不同的文件中。
+	数据的存储顺序为插入顺序（没有经过排序、插入速度快，空间占用量小）
+	功能
	+	全文索引支持
	+	数据的压缩存储（myisamPack）
	+	并发性
		+	仅仅支持表级锁定，不支持高并发。
		+	支持并发插入。写操作中的插入操作，不会阻塞读操作（其他操作）

Innodb 和 MyISAM 比较：

+	Innodb ：数据完整性，并发性处理，擅长更新，删除。
+	myisam：高速查询及插入。擅长插入和查询。

其他存储引擎：

+	Archive：存档型，仅提供插入和查询操作。非常高效阻塞的插入和查询。
+	Memory：内存型，数据存储于内存中，存储引擎。缓存型存储引擎。
+	插件式存储引擎：用C和C++开发的存储引擎。


4. 数据库锁

当客户端操作表（记录）时，为了保证操作的隔离性（多个客户端操作不能互相影响），通过加锁来处理。

### 字段类型选择

类型取值如下：

|类型|大小|范围（有符号）|范围（无符号）|用途|
|:--|:--|:--|:--|:--|
|数值类型<br/>|||||
|TINYINT|1 byte|(-128，127)|(0，255)|小整数值|
|SMALLINT|2 bytes|(-32 768，32 767)|(0，65 535)|大整数值|
|MEDIUMINT|3 bytes|(-8 388 608，8 388 607)|(0，16 777 215)|大整数值|
|INT或INTEGER|4 bytes|(-2 147 483 648，2 147 483 647)|(0，4 294 967 295)|大整数值|
|BIGINT|8 bytes|(-9,223,372,036,854,775,808，9 223 372 036 854 775 807)|(0，18 446 744 073 709 551 615)|极大整数值|
|FLOAT|4 bytes|(-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38)|0，(1.175 494 351 E-38，3.402 823 466 E+38)|单精度|
|||||浮点数值|
|DOUBLE|8 bytes|(-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)|0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)|双精度|
|||||浮点数值|
|DECIMAL|对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2|依赖于M和D的值|依赖于M和D的值|小数值|
|日期和时间类型<br/>|||||
|DATE|3|1000-01-01/9999-12-31|YYYY-MM-DD|日期值|
|TIME|3|'-838:59:59'/'838:59:59'|HH:MM:SS|时间值或持续时间|
|YEAR|1|1901/2155|YYYY|年份值|
|DATETIME|8|1000-01-01 00:00:00/9999-12-31 23:59:59|YYYY-MM-DD HH:MM:SS|混合日期和时间值|
|TIMESTAMP|4|1970-01-01 00:00:00/2038|YYYYMMDD HHMMSS|混合日期和时间值，时间戳|
|||结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07|||
|字符串类型<br/>|||||
|CHAR|0-255 bytes|||定长字符串|
|VARCHAR|0-65535 bytes|||变长字符串|
|TINYBLOB|0-255 bytes|||不超过 255 个字符的二进制字符串|
|TINYTEXT|0-255 bytes|||短文本字符串|
|BLOB|0-65 535 bytes|||二进制形式的长文本数据|
|TEXT|0-65 535 bytes|||长文本数据|
|MEDIUMBLOB|0-16 777 215 bytes|||二进制形式的中等长度文本数据|
|MEDIUMTEXT|0-16 777 215 bytes|||中等长度文本数据|
|LONGBLOB|0-4 294 967 295 bytes|||二进制形式的极大文本数据|
|LONGTEXT|0-4 294 967 295 bytes|||极大文本数据|


选择优化的数据类型原则：
+	更小的通常更好
+	简单数据类型需要更少的CPU周期（用MySQL内建的类型(date, time, datetime)来存储时间和日期、使用整型存储IP地址）
+	指定列为NOT NULL（可为NULL的列使得索引、索引统计和值比较都更复杂）

分类：
+	整数类型
	+	TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT分别使用8,16,24,32,64位存储空间。它们可以存储的值得范围从-2(N-1)到2(N-1)-1，如果是UNSIGNED，表示不允许负值，那正数的上限提高一倍
+	浮点类型
	+	FLOAT、DOUBLE、DECIMAL（高精度） 分别使用4字节、8字节、可变个字节存储
+	字符串类型
	+	VARCHAR比CHAR更节省空间，VARCHAR会使用1或2个额外的字节记录字符串的长度。如果使用UTF8字符集，应该选择VARCHAR类型。CHAR适合存储很短的字符串，或者所有值都接近同一个长度。比如MD5加密后的值。对于经常变更的数据，CHAR比VARCHAR更好，因为CHAR类型不易产生碎片
+	日期和时间类型
	+	DATETIME使用8个字节的存储空间，TIMESTAMP使用4个字节，一般情况下尽量选择TIMESTAMP类型

TIMESTAMP 特点：
+	当更新一条数据的时候，设置此类型根据当前系统更新可自动更新时间
+	如果插入一条NULL，也会自动插入当前系统时间
+	创建时，自动设置默认值
+	会根据当前时区来存储和查询时间，存储时对当前时区进行转换，查询时再转换为当前的时区

### 范式与逆范式

为了建立冗余较小、结构合理的数据库，设计数据库时必须遵循一定的规则。在关系型数据库中这种规则就称为范式。范式是符合某一种设计要求的总结。要想设计一个结构合理的关系型数据库，必须满足一定的范式。逆范式是指打破范式，通过增加冗余或重复的数据来提高数据库的性能。


三大范式：
+	第一范式1NF，原子性；段值都是不可分解
+	第二范式2NF，消除部分依赖；表中的每列都和主键相关
+	第三范式3NF，消除传递依赖；表中非主键列不依赖于其它非主键列
+	优点
	+	范式可以避免数据冗余，减少数据库的空间，减轻维护数据完整性的麻烦。
+	缺点
	+	所用的范式越高，对数据操作的性能越低。所以我们在利用范式设计表的时候，要根据具体的需求再去权衡是否使用更高范式去设计表。

# 二、功能优化

功能优化包含索引，缓存，分区分表等部分。

### 索引


在关系数据库中，索引是一种单独的、物理的对数据库表中一列或多列的值进行排序的一种存储结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据页的逻辑指针清单。索引的关键字一定是排序的。索引本质上是表字段的有序子集，类似于一个目录，通过它可以快速定位数据。

1. 分类

+	种类
	+	聚集索引(clustered index)：子顶点存储行记录，有且只有一个聚焦索引。存在主键，则主键是聚焦索引；否则第一个not NULL unique列是聚集索引；否则，InnoDB会创建一个隐藏的row-id作为聚集索引。
	+	普通索引(secondary index)
+	类型
	+	主键索引,primary key：要求关键字不能重复，也不能为NULL。同时增加主键约束
	+	唯一索引,unique index：要求关键字不能重复。同时增加唯一约束
	+	普通索引(secondary index)
	+	普通索引,index：对关键字没有要求
	+	全文索引,fulltext key：关键字的来源不是所有字段的数据，而是从字段中提取的特别关键词。该类型的索引特殊在：关键字的创建上。是为了解决 like&lsquo;%keyword%&rsquo;这类查询的匹配问题。（mysql的全文索引几乎不用，因为它不支持中文，应该使用`sphinx全文索引`）。


	+	复合索引：如果一个索引通过在多个字段上提取的关键字，称之为复合索引

```
-- 主键索引
ALTER TABLE student ADD PRIMARY KEY  (id);
-- 普通索引
ALTER TABLE student ADD INDEX index_stu (`name`);
-- 删除普通索引
ALTER TABLE student drop INDEX index_stu;
-- 唯一索引
ALTER TABLE student  ADD UNIQUE idx_uq_name (`name`)
-- 全文索引
ALTER TABLE student ADD FULLTEXT idx_ft_name (`name`)
-- 复合索引
ALTER TABLE student ADD INDEX idx_fh ( `name`,school_id)
```

2. 最左匹配原则

最左匹配原则都是针对联合索引来说的，构建一颗 B+ 树只能根据一个值来构建，因此数据库依据联合索引最左的字段来构建 B+ 树。在 InnoDB 中联合索引只有先确定了前一个（左侧的值）后，才能确定下一个值。如果有范围查询的话，那么联合索引中使用范围查询的字段后的索引在该条 SQL 中都不会起作用。

例如：
```
-- 定义 a b c 的联合索引
-- a. 生效的例子
-- eg1
select * from t where a=1 and b=1 and c=1;

-- eg2
select * from t where a=1 and c=1 and b=1;
==经过优化器转换==> 
select * from t where a=1 and b=1 and c=1;

-- eg3
select * from t where a=1 and b=1 and c>1;


-- b. 未生效的例子
-- eg1
select * from t where a=1 and b>1 and c=1;（只有ab用上索引）

-- eg2
select * from t where a=1 and c=1;（只有a用上索引）

```

> 注：mysql的执行计划和查询的实际执行过程并不完全吻合，所以参数的查询也会经过优化。


3. 使用OR
必须要保证 OR 两端的条件都存在可以用的索引，该查询才可以使用索引。


4. 索引覆盖
索引拥有的关键字内容，覆盖了查询所需要的全部数据。此时，就不需要在数据区获取数据，仅仅在索引区即可。覆盖就是直接在索引区获取内容，而不需要在数据区获取。explain的输出结果Extra字段为Using index时，能够触发索引覆盖。即查询字段都为索引字段，通过索引直接就可以获取数据，而不需要再查询表。

使用场景：
+	全表count查询优化
+	列查询回表优化（建立联合索引（name+sex），`select id,name,sex ... where name='shenjian'`）
+	分页查询

### 缓存

MySQL 缓存机制就是缓存sql 文本及缓存结果，用KV形式保存再服务器内存中，如果运行相同的sql,服务器直接从缓存中去获取结果，不需要在再去解析、优化、执行sql。如果表修改了（如insert,update,delete,truncate,alter table,drop table或者是drop database 等命令），那么该表的所有索引都不会生效，查询缓存值得相关条目将被清空。对于更新频繁的表，查询缓存并不合适；对于不变的数据以及查询sql大都相似的表，查询缓存会提高很大性能。

1. 命中条件

缓存存在于一个hash表中，通过查询SQL，查询数据库，客户端协议等作为key,在判断命中前，mysql不会解析SQL，而是使用SQL去查询缓存，SQL上的任何字符的不同，如空格、注释，都会导致缓存不命中。如果查询有不确定的数据like now(),current_date()，那么查询完成后结果者不会被缓存，包含不确定的数的是不会放置到缓存中。

2. 工作流程

+	服务器接收SQL，如果涉及表的库有启动缓存，根据SQL及其他条件查找缓存表；
+	如果找到了缓存，则直接返回缓存；
+	如果没有找到缓存，则执行SQL查询，包括原来的SQL解析，优化等；
+	执行完SQL查询结果以后，将SQL查询结果缓存入缓存表。

3. 缓存失败

当某个表正在写入数据，则这个表的缓存（命中缓存，缓存写入等）将会处于失效状态，在Innodb中，如果某个事务修改了这张表，则这个表的缓存在事务提交前都会处于失效状态，在这个事务提交前，这个表的相关查询都无法被缓存。


4. 缓存参数配置

+	query_cache_type: 是否打开缓存
	+	OFF: 关闭
	+	ON: 总是打开
	+	DEMAND: 只有明确写了SQL_CACHE的查询才会吸入缓存
+	query_cache_size: 缓存使用的总内存空间大小,单位是字节,这个值必须是1024的整数倍,否则MySQL实际分配可能跟这个数值不同(感觉这个应该跟文件系统的blcok大小有关)
+	query_cache_min_res_unit: 分配内存块时的最小单位大小
+	query_cache_limit: MySQL能够缓存的最大结果,如果超出,则增加 Qcache_not_cached的值,并删除查询结果
+	query_cache_wlock_invalidate: 如果某个数据表被锁住,是否仍然从缓存中返回数据,默认是OFF,表示仍然可以返回

> 注：这里的缓存仅当数据表的记录改变时，缓存才会被删除，而不是依靠过期时间的。如果存在不想使用缓存的SQL执行，则可以使用 SQL_NO_CACHE语法


### 分区分表

日常开发中经常会遇到大表的情况，所谓的大表是指存储了百万级乃至千万级条记录的表。这样的表过于庞大，导致数据库在查询和插入的时候耗时太长，性能低下，如果涉及联合查询的情况，性能会更加糟糕。分表和表分区的目的就是减少数据库的负担，提高数据库的效率，通常点来讲就是提高表的增删改查效率。




1. 分区

分区，partition，分区是将数据分段划分在多个位置存放，可以是同一块磁盘也可以在不同的机器。分区后，表面上还是一张表，但数据散列到多个位置了。app读写的时候操作的还是大表名字，db自动去组织分区的数据。

MySQL提供4种分区算法：
+	取余
	+	Key
	+	hash 
+	条件
	+	List
	+	range

实例如下：
```
-- 查看是否支持分区
show VARIABLES like 'have_partitioning'

-- 取余：Key
CREATE TABLE partition_1 (
	id  int PRIMARY key AUTO_INCREMENT,
	title VARCHAR(255)
)
partition by key(id) PARTITIONS 3;

分区会生成以下三个文件：
	partition_1#p#p0.ibd
	partition_1#p#p1.ibd
	partition_1#p#p2.ibd
分区与存储引擎无关，是MySQL逻辑层完成的。


-- 取余：hash;，按照某个表达式的值进行取余
CREATE TABLE partition_2 (
	id int UNSIGNED  AUTO_INCREMENT,
	birthday date,
	pname VARCHAR(255),
	PRIMARY KEY 	(id, birthday)
)
partition by hash(month(birthday)) PARTITIONS 12;

-- 条件：List
CREATE TABLE partition_4 (
	id int UNSIGNED  AUTO_INCREMENT,
	birthday date,
	pname VARCHAR(255),
	PRIMARY KEY 	(id, birthday)
)
partition by list(month(birthday)) (
	partition spring VALUES IN(3, 4, 5),
	partition summer VALUES IN(6, 7, 8),
	partition autumn VALUES IN(9, 10, 11),
	partition winter VALUES IN(12, 1, 2)
);


-- 条件：range

CREATE TABLE partition_5 (
	id int UNSIGNED  AUTO_INCREMENT,
	birthday date,
	pname VARCHAR(255),
	PRIMARY KEY 	(id, birthday)
)
partition by range(year(birthday)) (
	partition p_90 VALUES LESS THAN (2000),
	partition p_00 VALUES LESS THAN (2010),
	partition P_10 VALUES LESS THAN (2020),
	partition p_20 VALUES LESS THAN MAXVALUE
);

```

1.2. 其它语法

```
-- 取余
-- 增加分区数量
Alter TABLE TABLE_NAME add partition partitions N
-- 减少分区数量
Alter TABLE TABLE_NAME COALESCE partition N

-- 条件
-- 添加分区
Alter TABLE TABLE_NAME add partition (
partition P_10 VALUES LESS THAN (2020)
)
-- 删除
Alter TABLE TABLE_NAME drop partition P_10

删除条件算法的分区，会导致分区数据丢失。添加分区不会。


```

2. 分表
分表是将一个大表按照一定的规则分解成多张具有独立存储空间的实体表，我们可以称为子表，每个表都对应三个文件，MYD数据文件，.MYI索引文件，.frm表结构文件。这些子表可以分布在同一块磁盘上，也可以在不同的机器上。app读写的时候根据事先定义好的规则得到对应的子表名，然后去操作它。分表技术是比较麻烦的，需要手动去创建子表，app服务端读写时候需要计算子表名。采用merge好一些，但也要创建子表和配置子表间的union关系。（需要手动分表）

+	水平分表：通过结构相同的N个表存储数据，MySQL提供了一个可以将多个结构相同的myisam表合并到一起的存储引擎mrg_myisam。
+	垂直分表：一张表中存在多个字段。这些字段可以分为常用字段和非常用字段，为了提高查表速度，我们可以把这两类字段分开来存储。主要目的，减少每条记录的长度



> [MySQL如何判别InnoDB表是独立表空间还是共享表空间](https://www.cnblogs.com/kerrycode/p/9515200.html)
[mysql 缓存机制](https://blog.csdn.net/qzqanzc/article/details/80418125)