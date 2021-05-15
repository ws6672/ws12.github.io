---
title: Oracle——什么时候使用commit
date: 2019-09-10 22:59:10
tags: oracle
---


# Oracle----DML语句需要提交

### 前因
之前，我在项目中使用的都是Mysql，而Oracle是初次在实际工作中使用。由于太久没有接触，在创建项目假数据时没有提交导致数据查询不到，借此，我了解了Oracle与Mysql作为关系型数据库在事务支持上的差异。



### 一、什么是SQL
结构化查询语言（Structured Query Language)）简称SQL，是操作和检索关系型数据库的标准语言,它的四大功能如下：

+ 数据查询语言（DQL：Data Query Language）：语句主要包括 SELECT ，用于从表中检索；

+ 数据操作语言（*DML*：Data Manipulation Language）：主要包括 insert，update，delete，用于添加、修改和删除表中的行数据；

+ 事务处理语言：（TPL：Transaction Process Language）：语句主要包括 commit， rollback，用于提交和回滚；

+ 数据控制语言：（DCL：Data Control Language）：主要包括 grant，revoke，用于进行授权和收回权限；

+ 数据定义语言（*DDL*：Data Definition language）：主要包括 create， drop， alter，用于定义、销毁、修改数据库对象；


### 二、MySQL的几个引擎
+ MyISAM：适合基本都是单表查询的数据。基于 ISAM【1】的存储引擎，拥有较高的插入、查询速度，但不支持事务。【不会自动提交】

+ InnoDB：是事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键【会自动提交增删改，默认引擎，也是我常用的引擎】

+ Memory：适合临时存放数据，且数据量不大，并且不需要较高的数据安全性

+ Archive：适合存储归档数据【2】，即只有INSERT和SELECT操作

在之前使用MySQL的时候，基本是使用默认的存储引擎 InnoDB，支持自动提交事务，导致切换到Oracle时就掉坑里了。InnoDB支持事务，且DML语句自动提交。而MyISAM不支持事务，即便用事务，它也不会起作用。

在 MySQL 中有存储引擎，而Oracle则没有引擎的概念，而是有OLTP和OLAP模式的之分，而两种模式都支持事务，因为它不允许脏读。在Oracle中，每句DDL语句前后都将提交数据；而DML语句则需要手动提交数据。


> 注1: 索引顺序存取方法（ISAM, Indexed Sequential Access Method）最初是IBM公司发展起来的一个文件系统，可以连续地（按照他们进入的顺序）或者任意地（根据索引）记录任何访问
> 注2：归档数据是不再经常使用的数据


### 三、Oracle 的两个模式———— OLTP、OLAP（数据仓库）

线上事务处理OLTP（on-line transaction processing）、线上分析处理OLAP（On-Line Analytical Processing）。OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如订单处理，票据跟踪或人事档案系统。OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。

##### OLTP优化
主要的调优方法包括在数据库索引并在应用程序使用预调优查询。磁盘排序最小化，共享代码最大化。在许多情况下，出于性能原因，可以合并（非规范化）密切相关的表。

##### OLAP调优
OLAP调优涉及预先构建最常用的集合，然后调整大型排序（磁盘和内存排序的组合）以及在尽可能多的物理驱动器上传输数据，以便尽可能多地搜索数据。Oracle并行查询技术是从OLAP数据库获得最佳性能的关键。大多数OLAP查询本质上是临时的，这使得调优成为问题，因为共享代码的使用被最小化并且索引可能难以优化。

### 四、Oracle中的DDL(自动提交)

+ create table 创建表   
+ alter table  修改表  
+ drop table 删除表  
+ truncate table 删除表中所有行   
+ create index 创建索引  
+ drop index  删除索引

### 五、Oracle中的DML（需要使用【COMMIT;】语句）

+ insert 插入记录
+ update 修改记录
+ delete 删除记录