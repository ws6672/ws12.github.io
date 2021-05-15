---
title: 面试——MySQL
date: 2020-06-29 21:48:05
tags: [ms]
---

### 1. 数据库的三范式是什么
+	第一范式：列不可再分
+	第二范式：可以唯一区分主列，主键约束
+	第三范式：表的非主属性不能依赖于其他表的非主属性 外键约束

### 2. 事务
事务(TRANSACTION)是作为单个逻辑工作单元执行的一系列操作， 这些操作作为一个整体一起向系统提交，要么都执行、要么都不执行 。

##### 2.1 ACID
事务是一个不可分割的工作逻辑单元事务必须具备以下四个属性，简称 ACID 属性：

+	原子性（Atomicity）：事务是一个完整的操作。事务的各步操作是不可分的（原子的）；要么都执行，要么都不执行。
+	一致性（Consistency）：当事务完成时，数据必须处于一致状态。
+	隔离性（Isolation）：对数据进行修改的所有并发事务是彼此隔离的， 这表明事务必须是独立的，它不应以任何方式依赖于或影响其他事务。
+	持久性（Durability）：事务完成后，数据的修改永久保存



##### 2.2 事务的并发问题

+	脏读：可读未提交数据
+	不可重复读：只可读已提交，但事务内前后两次读取***数据不一致***；侧重于修改，锁住满足条件即可。
+	幻读：只可读已提交，读取数据一致，但***数据的记录数不一致***；新增或删除，需要锁住整个表

##### 2.2 事务的隔离级别

+	读未提交（read-uncommitted）：会导致 脏读、不可重复读、幻读
+	不可重复读（read-committed）：会导致 不可重复读、幻读；Oracle默认隔离级别
+	可重复读（repeatable-read）	：会导致 幻读；mysql的默认级别（默认引擎`INNODB`的隔离级别是可重复读，但是通过锁算法达到了串行化,所以它不会导致幻读）
+	串行化（serializable）


### 3. SQL优化
+	查询语句中不要使用select *
+	尽量减少子查询，使用关联查询（left join,right join,inner join）替代
+	减少使用IN或者NOT IN 等全表查询,使用exists，not exists或者关联查询语句替代
+	or 的查询尽量用 union或者union all 代替(在确认没有重复数据或者不用剔除重复数据时，union all会更好)
+	应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。
+	尽量避免在 where 子句中对字段进行 null 值判断

### 4. 简单说一说drop、delete与truncate的区别

+	drop：删表数据和结构，数据提交后生效
+	truncate：删除全表数据，主键标志计数重置
+	delete：删除表数据，主键排序不变

### 5. 什么是内联接、左外联接、右外联接
+	内联接：匹配两张表相关联数据
+	左外联接：左表数据加相关联数据
+	右外联接：右表数据加相关联数据


### 6. 索引类型
+	主键索引（PRIMARY）
+	唯一索引（UNIQUE）
+	普通索引（INDEX）
+	全文索引（FULLTEXT）


### 7. MySQL练习


##### 7.1 查询
查询教师所有的单位即不重复的 Depart 列。
select distinct Depart from teacher 

以Class降序查询Student表的所有记录。
select*from teacher order by Class desc

查询Score表中至少有5名学生选修的并以3开头的课程的平均分数

select Cno,AVG(Degree) from Score where Cno LIKE '3%' group by Cno having Count(Cno) >= 5

查询最低分大于70，最高分小于90的Sno列。

select Sno from Score group by Sno having min(Degree) > 70 and max(Degree) < 90;


##### 7.2 union
 
union用于把两个或者多个select查询的结果集合并成一个
+	默认会去掉两个查询结果集中的重复行
+	默认结果集不排序
+	最终结果集的列名来自于第一个查询的SELECT列表

语法：
```
SELECT …
	UNION [ALL | DISTINCT]
SELECT …
		[UNION [ALL | DISTINCT]
	SELECT …]
```


实例：

```
SELECT v.LINEID,v.LINENAME FROM `v_bas_line` v 
union 
SELECT vt.LINEID,vt.LINENAME FROM `v_bas_line_t` vt
```

输出结果：
![union](/image/ms/mysql/union.png)


union、union all的区别：
+	union会进行去重，union all不会进行去重



##### 7.3 DDL

DDL是数据定义语言，就是对数据库、表层面的操作，如CREATE、ALTER、DROP.


查询用户权限
```
show grants for test@'localhost'
```


授权
```
grant select,update,insert,delete,alter on db.* to test@'test' identify by 'test'
```

切换数据库
```
use eg;
```


### 8. 系统相关

##### 8.1 CMD命令

开启MySQL服务
```
service mysqld start

/init.d/mysqld start

safe_mysql &
```

关闭mysql服务
```
service mysqld stop

/etc/init.d/mysqld stop

mysqladmin -uroot -p123456 shutdown
```

登陆MySQL数据库
```
mysql -uroot -p123456
```

多实例登陆
```
mysql -uroot -p123456 -S /data/3306/mysql.sock
```

查看当前登录的用户。
```
mysql> select user();
```

查看当前数据库版本
```
# mysql -V
mysql> select version();
```




##### 8.2 多实例

mysql多实例实质

在一台机器上开启多个不同的mysql服务端口（3306,3307），运行多个mysql服务进程，这些服务进程通过不同的socket监听不同的服务端口来提供各自的服务；