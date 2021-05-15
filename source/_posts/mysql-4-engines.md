---
title: mysql 存储引擎
date: 2020-04-15 15:56:03
tags: [mysql]
---

创建数据库时，存储引擎（尤其是随着数据库的增长）经常会忽略但关键的性能因素。在许多情况下，诱惑是仅接受默认值并继续开发您的项目。这可能会在应用程序生命周期的后期（例如您的团队实施分析和MySQL仪表板时）对性能，备份和数据完整性造成意外的负面影响。

### 一、支持的存储引擎

默认情况下，MySQL 5.7支持十个存储引擎 (InnoDB, MyISAM, Memory, CSV, Archive, Blackhole, NDB, Merge, Federated, and Example)。在服务器上，可以通过以下命令查看对存储引擎的支持：
	`SHOW ENGINES`

### 二、存储引擎的特点

各个存储引擎的使用业务如下：

1. InnoDB

InnoDB是MySQL 5.7中的默认选项，是一个强大的存储引擎，可提供：

+	完全符合ACID。
+	提交，回滚和崩溃恢复。
+	行级锁定。
+	外键（FOREIGN KEY）-参照完整性约束。
+	多用户并发性（通过非锁定读取）


2. MyISAM

基于`InnoDB`,与众不同的功能有：

+	全文搜索索引。
+	表级锁定。
+	缺乏对交易的支持。

尽管它是一个快速的存储引擎，但它最适合用于需要大量事务且不需要事务支持或ACID的大量读取应用程序，例如数据仓库和Web应用程序。

3. NDB（或NDBCLUSTER）

如果数据库将在群集环境中运行，则NDB是首选的存储引擎。最好在您需要：

+	分布式计算。
+	高冗余。
+	高可用性。
+	最高的正常运行时间。

> 注：标准MySQL Server 5.7二进制文件的发行版中不包含对NDB的支持。您将必须更新到最新的MySQL Cluster二进制版本。

4. CSV
当数据需要与使用CSV格式数据的其他应用程序共享时，有用的存储引擎。这些表存储为逗号分隔的值文本文件。缺点是CSV文件未建立索引。因此，数据应存储在InnoDB表中，直到需要导入/导出再临时把引擎切换过来。

5. Blackhole
此引擎接受但不存储数据。与UNIX / dev / null相似，查询始终返回空集。在您不想在本地存储数据或在性能或其他测试情况下的分布式数据库环境中，这很有用。

6. Archive
该表未建立索引，并且在插入时进行压缩。不支持交易。使用此存储引擎可以存档和检索过去的数据。

7. Federated

此存储引擎用于通过链接多个不同的物理MySQL服务器来创建单个本地逻辑数据库。没有数据存储在本地服务器上，查询将在相应的远程服务器上自动执行。它非常适合分布式数据集市环境，并且在使用MySQL进行分析报告时可以大大提高性能。


### 三、指定存储引擎建表

```
CREATE TABLE Shared_Data (
	Data_ID INT(50) NOT NULL, 
	`Name` VARCHAR(50) NOT NULL, 
	Description VARCHAR(150)  NOT NULL
) ENGINE='CSV';
```

### 参考文章

> [the-beginners-guide-to-mysql-storage-engines](https://dzone.com/articles/the-beginners-guide-to-mysql-storage-engines)