---
title: mysql 实战（一）数据导出导出
date: 2019-12-06 09:51:42
tags: [mysql]
---

mysqldump导出，source导入，谁用谁知道。

# 一、初识mysqldump
在初入职场的时候，刚进行正式的开发，数据库半生不熟，表格的创建大都用navicat。一天，老大要我迁移一个数据库，不用说，上navicat。当时是在局域网，用navicat连另外一台电脑的mysql，但navicat迁移大数据库有问题，有时候跑着跑着就卡死了。于是，用navicat导出数据，导出倒是蛮快的，但导入就慢得跟蜗牛一般。

于是，因此使用了mysqldump。

### 1. 什么是 mysqldump
mysqldump可以产生一组能够被执行以再现原始数据库对象定义和表数据的SQL语句。它转储一个或多个MySQL数据库以进行备份或转移到另一台SQL服务器。所述的mysqldump 命令也可以生成CSV输出，其他分隔符的文本或XML格式。

需要的权限：
+	SELECT 查表权限
+	SHOW VIEW 查看视图权限
+	TRIGGER 触发器权限
+	LOCK TABLES 锁表权限
	+	 --single-transaction[单事务]，启用该选项可以避免表在导出的过程中被其他用户删除、修改
+	ALTER DATABASE
	+	mysqldump导出的时候可以保存数据的编码格式，但导入数据库的编码可能与导入数据的编码不同，所以使用mysqldump的用户还需要拥有修改数据库的权限


>	对于大规模备份和还原， 物理备份更适合于以可以快速还原的原始格式复制数据文件。如果您的表主要是InnoDB 表，或者混合使用InnoDB 和MyISAM表，可以考虑使用MySQL Enterprise Backup产品的 mysqlbackup命令。如果您的表主要是MyISAM 表，请考虑改用mysqlhotcopy ，以获得比mysqldump备份和还原操作更好的性能 。

### 差别
用mysqldump导出数据和用navicat导出有很大的不同。navicat导出是一条数据一个插入语句，即一条数据要执行一次；而mysqldump导出的时候，是把多条插入语句放置在一起，即多条数据执行一次即可。数据量越大，差距越明显。

##### navicat 导出的sql文件
```
INSERT INTO `tttt` VALUES ('0300F130000170GNN00FAB003', '20130321');
INSERT INTO `tttt` VALUES ('0300F130000170GNN00FAB003', '20130321');
```

##### mysqldump导出的文件
```
INSERT INTO `tttt` VALUES ('0300F130000170GNN00FAB003', '20130321'),INSERT INTO `tttt` VALUES ('0300F130000170GNN00FAB003', '20130321');
```

### mysqldump 使用


```
mysqldump -u <username> -p <dbname> > /path/to/***.sql

shell> mysqldump [options] db_name [tbl_name ...]
shell> mysqldump [options] --databases db_name ...
shell> mysqldump [options] --all-databases
```


##### 选项

```
-h, --host=name：服务器IP
-u, --user=name：登录名
-p, --password[=name]:登录密码
-A, --all-databases：导出所有数据库
-B, --databases：导出指定的数据库，多个数据库名使用空格分割
--tables：导出指定表
-d, --no-data：仅导出表结构，不导出数据
-t, --no-create-info：不导出表创建语句
-n, --no-create-db：不导出CREATE DATABASE IF EXISTS语句
-e, --extended-insert：将多条记录合并成一条INSERT语句来提高插入效率
--add-drop-table：在创建表之前加入DROP TABLE语句
--hex-blob ：将二进制的数据以16进制导出
-R, --routines：导出存储过程和存储函数
--triggers：导出触发器
--master-data[=#]：导出CHANGE MASTER命令，当设置为1时，CHANGE命令正常导出，当设置为2时，CHANGE命令以注释模式导出
                    master-data开启时，会默认启用--lock-all-tables选项，并自动禁用--lock-tables选项
--dump-slave：在从库上执行时，dump-slave用来导出当前主库上的位置信息
--single-transaction：单实例模式运行
--lock-all-tables，-x：在导出前对所有表加全局只读锁，并自动关闭--single-transaction 和 --lock-tables 选项
--lock-tables ：在导出当前表数据前才对表进行加锁，该选项指使用与MyISAM表。--lock-tables无法保证所有表数据在数据库级别一致。
--default-character-set=charset：设置导出时使用的字符集
--quick，-q ：在导出大表时很有用，它强制 mysqldump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中。
--skip-opt参数，会关闭默认--opt选项，导致：
		导出脚本中缺少DROP TABLE IF EXISTS语句
		导出脚本中CREATE TABLE语句中缺少存储引擎设置
		导出脚本中CREATE TABLE语句中缺少默认字符集设置
		导出脚本中CREATE TABLE语句中缺少自增列和自增初始值设置。
```

### 实例

*导出*
```
# 复制单数据库【sys】
mysqldump -u root -p sys > e:test.sql
# 复制单数据库【sys】数据
mysqldump -u root -p sys-t > e:test.sql
# 复制单数据库【sys】结构
mysqldump -u root -p sys-d > e:test.sql
# 复制所有数据库 
mysqldump -u root -p -A > e:test.sql 
# 复制多个数据库
mysqldump -u root --databases sys sys2 > e:test.sql 

```

*导入*
```
# 进入账号
mysql>source f:\test.sql
# 命令行
mysql -uroot -p123456 mydb <f:\mydb.sql
```

# 二、导入优化探索

近几日，在使用source导入11g的文件的时候，运行了十个小时还未结束；判定导入应当存在优化空间。

### SELECT INTO…OUTFILE语句、LOAD DATA …INFILE

SELECT INTO…OUTFILE语句把表数据导出到一个文本文件中，并用LOAD DATA …INFILE语句恢复数据。但是这种方法只能导出或导入数据，并不包括表的结构，意味着不适用于迁移数据库，仅适合迁移单表数据。

+	有读写权限
+	文件名必须唯一
+	fields terminated by ','必须存在，否则打开的文件的列在同一的单元格中出现

```
SELECT * INTO OUTFILE 'data.txt' FIELDS TERMINATED BY ',' FROM table2;
LOAD DATA INFILE 'data.txt' INTO TABLE table2 FIELDS TERMINATED BY ',';
```

导入导出单表数据可以使用SELECT INTO…OUTFILE语句，但我这次要导入的是整个数据库而非单表数据，所以这个方案并不适用。




### innodb_flush_log_at_trx_commit、sync_binlog 参数

*innodb_flush_log_at_trx_commit*
innodb_flush_log_at_trx_commit  是引擎关于事务提交时刷新日志调度策略。


+	0【延写延刷】：log buffer将每秒一次地写入log file中，并且log file的flush(刷到磁盘)操作同时进行。*在这一秒里执行写、刷操作（多个事务）*。
+	1【实写实刷】：每次事务提交时MySQL都会把log buffer的数据写入log file，并且flush(刷到磁盘)中去，该模式为系统默认。*每个事务提交都执行写、刷操作*。
+	2【实写延刷】：每次事务提交时MySQL都会把log buffer的数据写入log file，但是flush(刷到磁盘)操作并不会同时进行。该模式下，MySQL会每秒执行一次 flush(刷到磁盘)操作（受限于进程策略，不会保证每秒一次）。*每个事务提交都执行写操作，但刷操作每秒一次（多事务）*

 
 
*sync_binlog*
sync_binlog 参数表示每写缓冲多次就同步到磁盘
+	sync_binlog 的默认值是0，像操作系统刷其他文件的机制一样，MySQL不会同步到磁盘中去而是依赖操作系统来刷新binary log。
+	sync_binlog为1时，每写缓冲1次就同步到磁盘
+	sync_binlog为500时，每写缓冲500次就同步到磁盘


*设置策略*
+	当innodb_flush_log_at_trx_commit、sync_binlog 参数均为1，安全性最高，也是默认的策略
+	当innodb_flush_log_at_trx_commit、sync_binlog 参数均为0，安全性最低，速度最快；
+	当innodb_flush_log_at_trx_commit 为2、sync_binlog 参数为500，安全性适中，速度较快，只有在操作系统崩溃或者系统掉电的情况下，上一秒钟所有事务数据才可能丢失。

在这次使用，我需要导入的是为11g的测试数据库，需要的只是部分表，所以对数据的完整性不在意。所以，两个参数都被设置为0。


```
show VARIABLES like '%innodb_flush_log_at_trx_commit%';
show VARIABLES like '%sync_binlog%';
-- 推荐配置
set global innodb_flush_log_at_trx_commit = 0;
set global sync_binlog = 0;
-- 重启数据库后失效？
-- 永久生效需要写入 【my.ini】配置文件
```




### 导入多个sql文件
+	新建test.sql文件
+	```
	source 1.sql
	source 2.sql
```
+	执行，`source test.sql`

###	max_allowed_packet、innodb_log_file_size

+	【max_allowed_packet】根据配置文件会限制server接受的数据包大小。有时候大的插入和更新会被max_allowed_packet 参数限制掉，导致失败（当表存在blog字段较为突出）；但设置过大，可能会有超出内存的异常，需要谨慎设置。
>	当传输的packet大于max_allowed_packet时，触发错误EN_NET_PACKET_TOO_LARGE,并且关闭Connection。在有的客户端中也会显示信息Lost connection to MySQL server during query
 
+	【innodb_log_file_size】该参数决定着mysql事务日志文件（ib_logfile0）的大小；当一个日志文件写满后，innodb会自动切换到另外一个日志文件，而且会触发数据库的检查点（Checkpoint），这会导致innodb缓存脏页的小批量刷新，会明显降低innodb的性能。

```
set global max_allowed_packet = 1024*1024*2000;	
-- set global innodb_log_file_size = 1024*1024*2000;	
-- innodb_log_file_size只读变量，需要在【my.ini】配置
```

### 删除多余的日志
```
show binary logs;
purge binary logs to 'binlog.000024'; // 删除000017之前的日志
-- 文件的名称是查出来的最后一条数据
```
### 结论
基于以上的各种优化，我的配置文件【my.ini】如今大致如下：
```
	[mysql]
	#设置mysql客户端默认字符集
	default-character-set=utf8
	[mysqld]
	#设置3306端口
	port = 3306
	# 设置mysql的安装目录
	basedir=D:\\software\\mysql
	# 允许最大连接数
	max_connections=200
	# 服务端使用的字符集默认为8比特编码的latin1字符集
	character-set-server=utf8
	# 创建新表时将使用的默认存储引擎
	default-storage-engine=INNODB
	# mysql事务日志文件（ib_logfile0）的大小
	innodb_log_file_size=2000M
	# mysql server 允许的最大数据包
	max_allowed_packet=200M
	sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
	# 事务提交时刷新日志的策略
	innodb_flush_log_at_trx_commit=2
	# 设置缓冲多少次就同步到磁盘（本地）
	sync_binlog=500
```

# 参考文章
>	[sync_binlog innodb_flush_log_at_trx_commit 浅析](http://blog.itpub.net/22664653/viewspace-1063134/)
[mysql innodb_log_file_size 和innodb_log_buffer_size参数](http://blog.itpub.net/29654823/viewspace-2147683/)
[InnoDB启动选项和系统变量](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit)
