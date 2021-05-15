---
title: mysql 进阶（三）优化
date: 2020-04-15 21:12:41
tags: [mysql]
---


### 为什么需要优化

使一个系统更快的最重要因素当然是基本设计。此外，还需要知道系统正做什么样的事情，以及瓶颈是什么。

常见的系统瓶颈如下：
+	磁盘搜索。需要花时间从磁盘上找到一个数据，用在现代磁盘的平均时间通常小于10ms，因此理论上我们能够每秒大约搜索1000次。这个时间在新磁盘上提高不大并且很难为一个表进行优化。优化它的方法是将数据分布在多个磁盘上。
+	磁盘读/写。当磁盘放入正确位置后，我们需要从中读取数据。对于现代的磁盘，一个磁盘至少传输10-20Mb/s的吞吐。这比搜索要容易优化，因为你能从多个磁盘并行地读。
+	CPU周期。我们将数据读入内存后，需要对它进行处理以获得我们需要的结果。表相对于内存较小是最常见的限制因素。但是对于小表，速度通常不成问题。
+	内存带宽。当CPU需要的数据超出CPU缓存时，主缓存带宽就成为内存的一个瓶颈。这在大多数系统正是一个不常见的瓶颈但是你应该知道它。


### 优化函数
SQL查询语句越复杂，需要的开销越多。
例如，执行GRANT语句。如果该语句不涉及所有表，或者能精确的设置到具体表，那么开销会大大降低。

如果问题是与具体MySQL表达式或函数有关，可以使用mysql客户程序所带的BENCHMARK()函数执行定时测试。语法如下：
`BENCHMARK(loop_count,expression)`

例如：
```
SELECT BENCHMARK(10000000,1+1);
```

### EXPLAIN

EXPLAIN语句可以用作DESCRIBE的一个同义词，获得关于MySQL如何执行SELECT语句的信息,语法如下：

+	`EXPLAIN tbl_name`
+	`EXPLAIN [EXTENDED] SELECT select_options`

 

***查询结果中各列的含义***

+	`select_type`		select的类型
	+	`SIMPLE`	简单表，不使用表连接或子查询
	+	`PRIMARY`	主查询，即外层的查询
	+	`UNION`	UNION中的第二个或者后面的查询语句
	+	`SUBQUERY`	子查询中的第一个
+	`table`	输出结果集表名
+	`type`	访问表的方式
	+	`ALL`	全表扫描
		+	没有查询参数或参数没有加索引
		+	使用 in、not in等关键字
	+	`index`	
		+	索引全扫描
	+	`range`	索引范围扫描
		+	<、<=、>、>=、between
	+	`ref`	非唯一索引扫描
		+	普通索引（非唯一索引）
		+	join操作
	+	`eq_ref`	唯一索引扫描
		+	多表连接时使用primary key或者unique index作为关联条件
	+	`const`,`system`	单表最多有一个匹配行
		+	表关联查询时必定会有一张表进行全表扫描
	+	`NULL`	不用扫描表或索引
+	`possible_keys` 可能使用的索引
+	`key`	实际使用的索引
+	`key_len`	使用索引字段的长度
+	`ref`	使用哪个列或常数与key一起从表中选择行。
+	`rows` 涉及行数
+	`filtered`	返回结果的行占需要读到的行（行列的值）的百分比
+	`Extra`	执行情况的说明和描述
	+	`Using Index`	表示索引覆盖，不会回表查询
	+	`Using Where`	表示进行了回表查询
	+	`Using Index Condition`	表示进行了ICP优化
	+	`Using Flesort`	表示MySQL需额外排序操作, 不能通过索引顺序达到排序效果
	
	
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