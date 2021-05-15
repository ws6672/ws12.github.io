---
title: mysql 进阶（一）事务
date: 2020-04-15 20:11:21
tags: [mysql]
---

事务（Transaction），一般是指要做的或所做的事情。在计算机术语中是指访问并可能更新数据库中各种数据项的一个程序执行单元(unit)。
事务一般表示一个不可再分的最小单元，需要批量的DML(insert、update、delete)语句共同联合完成。


### 事务的四大特征(ACID)

+	原子性(A)：不可再分的最小单位
+	一致性(C)：多个语句必须保证同时成功或者同时失败
+	隔离性(I)：两个事务之间需要具有隔离性
+	持续性(D)：事务结束后，提交的数据需要能永久保存到内存中

### 事务的隔离级别——隔离性

事务具有四个隔离级别

+	读未提交：read uncommitted
	+	事务B可以读取事务A未提交的数据——会导致`脏读`
+	读已提交：read committed
	+	事务A提交了数据，事务B才能读取，会导致`不可重复读`
	+	`不可重复读`是指：事务B内两次读取事务A的数据，结果不一样；主要针对 update、delete操作
	+	`Oracle默认隔离级别`
+	可重复读：repeatable read
	+	事务A提交了数据，事务B也不能读取；但是，会导致`幻读`；INNODB不会导致幻读。
	+	`幻读`是指：前后两次读取数据记录数不一致；主要针对insert操作。
	+	`mysql的默认级别`
+	串行化：serializable
	+	当事务A运行时，事务B排队等待，速度慢，并发差


> 注：与 SQL 标准不同的地方在于InnoDB 存储引擎在 REPEATABLE-READ（可重读）事务隔离级别下使用的是Next-Key Lock 锁算法，
因此可以避免幻读的产生，这与其他数据库系统(如 SQL Server)是不同的。因此，虽然它的隔离级别是可重复读，但是通过锁算法达到了串行化。

### 设置事务的隔离级别

语法如下：

```
SET [GLOBAL | SESSION] TRANSACTION ISOLATION LEVEL <isolation-level>
<isolation-level>为：
   READ UNCOMMITTED
   READ COMMITTED
   REPEATABLE READ
   SERIALIZABLE
例如：
-- 当前会话有效
	set transaction isolation level read committed
-- 全局有效
	set  GLOBAL  transaction isolation level read committed

```

### 查询隔离级别

```
1.查看当前会话隔离级别 select @@tx_isolation;
2.查看系统当前隔离级别 select @@global.tx_isolation;

-- ERROR 1193 (HY000): Unknown system variable 'tx_isolation' 
-- 新版MYSQL中，隔离级别相关变量被修改

select @@transaction_isolation;
select @@global.transaction_isolation;

```

