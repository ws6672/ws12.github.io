---
title: mysql 实战（三）explain
date: 2020-01-02 23:17:09
tags: [mysql]
---

# 概述

# 什么是 explain

在MySQL中，我们可以通过EXPLAIN命令获取MySQL如何执行SELECT语句的信息，包括在SELECT语句执行过程中表如何连接和连接的顺序。

# 查询结果中各列的含义

+	select_type		select的类型
	+	SIMPLE	简单表，不使用表连接或子查询
	+	PRIMARY	主查询，即外层的查询
	+	UNION	UNION中的第二个或者后面的查询语句
	+	SUBQUERY	子查询中的第一个
+	table	输出结果集表名
+	type	访问表的方式
	+	ALL	全表扫描
		+	没有查询参数或参数没有加索引
		+	使用 in、not in等关键字
	+	index	
		+	索引全扫描
	+	range	索引范围扫描
		+	<、<=、>、>=、between
	+	ref	非唯一索引扫描
		+	普通索引（非唯一索引）
		+	join操作
	+	eq_ref	唯一索引扫描
		+	多表连接时使用primary key或者unique index作为关联条件
	+	const,system	单表最多有一个匹配行
		+	表关联查询时必定会有一张表进行全表扫描
	+	NULL	不用扫描表或索引
+	possible_keys 可能使用的索引
+	key	实际使用的索引
+	key_len	使用索引字段的长度
+	ref	使用哪个列或常数与key一起从表中选择行。
+	rows 涉及行数
+	filtered	返回结果的行占需要读到的行（行列的值）的百分比
+	Extra	执行情况的说明和描述
	+	Using Index	表示索引覆盖，不会回表查询
	+	Using Where	表示进行了回表查询
	+	Using Index Condition	表示进行了ICP优化
	+	Using Flesort	表示MySQL需额外排序操作, 不能通过索引顺序达到排序效果

# 实例

```
EXPLAIN 
	SELECT
		* 
	FROM
		t_bas_unit t
		LEFT JOIN t_bas_unit_rule t1 ON t.plantid = t1.plantid 
		LIMIT 100;
```

![explain-1.png](/image/mysql/explain-1.png)

*解析*
type都是ALL，所以没有使用索引；select_type是SIMPLE不包含子查询、内查询。

# oracle 使用解释计划

使用`EXPLAIN PLAN FOR`解析查询语句

```
EXPLAIN PLAN FOR SELECT * FROM EMP;
```

查询结果 
```
SELECT plan_table_output FROM TABLE(DBMS_XPLAN.DISPLAY('EMP'));
select * from table(dbms_xplan.display);
```

# 参考文章
>	[MySQL——通过EXPLAIN分析SQL的执行计划](https://juejin.im/post/5b63ac5d5188251b1e1fea0f)