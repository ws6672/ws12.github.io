---
title: oracle探索笔记（二）基础语法
date: 2019-10-22 10:03:43
tags: [oracle]
---

学习一种语言，往往需要涉及到一堆繁杂的语法，在这篇文章中将简略的对Oracle的语法进行总览。

# 一、表


### 1. 建表
+	用圆括号而不是大括号
+	varchar2需要跟字段的长度，id的类型是 number
+	建表最好不用关键字，在使用plsql等连接数据库的工具时，如果起名高亮，那么就是关键字
+	如果要加关键字，需要用双引号括起来（""），并英文大写

```
create table t_sz_tkyth (
  tid number primary key,
  pdate date,
  area varchar2(20),
  qwdqfgl varchar2(20),
  qwmbfgl varchar2(20),
  zxdqfgl varchar2(20),
  zxmbfgl varchar2(20),
  work_type varchar2(20)
);
```



# 二、分页、排列

### 1. 分页

##### *ROWNUM + WHERE*
这种方式不支持排序的数据进行分页。

```
	--	【ROWNUM <=10】对数据进行第一次过滤，即选择前十条数据，在此处如果使用【ROWNUM>=10】则会报错；由于已经过滤，所以如果排序则是局部的，无法对全局生效。
	--	【rn >4】二次过滤
select * from (
       select ROWNUM as rn, t.* from TMDA_REPORT_ENERGY t where ROWNUM <=10
       ) mt where rn >4
```
##### *ROWNUM + order by*
通过两重内查询，先排序后分页，效率相对较低。

```
	select * from (
		   select ROWNUM as rn, t1.* from (
				select * from TMDA_REPORT_ENERGY t order by pdate asc --排序
			) t1 where ROWNUM <=10 --分页
       ) t2 where rn >=7
```

### 2. 排列

在oracle中，与mysql有许多类似的语法，譬如排序都用【order by】。而除了排序，oracle对于数据的排列有很好的支持。

##### *order by 排序*

使用【order by asc|desc】排序适合查全表的情况,因为查出来的结果中 【rownum】是乱序的。在oracle中所有的排序都是基于【order by asc|desc】的。

```
	-- 查询结果按日期排序，由小到大
	select rownum, t.* from TMDA_REPORT_ENERGY t order by t. pdate asc;
	
	-- 查询结果按日期排序，由大到小
	select *from TMDA_REPORT_ENERGY t order by t. pdate desc;
```

##### *row_number函数*

使用【row_number()over(order by 列名 desc)】查询，会先对数据进行排序，再赋予行号，而不考虑重复数据的问题。它适合查几条数据的情况，这时候查出的rownum是有序的。

```
	-- 先根据pdate排序，在为每一行分配行号，从1开始
	select row_number()over(order by t.pdate asc), t.* from TMDA_REPORT_ENERGY t 
```



##### *rank函数*

【rank函数】会先对数据进行排序，再赋予行号；如果遇到重复数据的时候，重复数据的序号一致，但会空出排名。

> 例如，前六条数据对应字段相同，则前六条数据序号都为1，而第七条数据从7开始。

```
	-- 按pdate排序
	select rank()over(order by t. pdate asc), t.* from TMDA_REPORT_ENERGY t 
```

##### *dense_rank 函数* 

【dense_rank 函数】会先对数据进行排序，再赋予行号；如果遇到重复数据的时候，重复数据的序号一致，下一条非重复数据序号加一。

> 例如，前六条数据对应字段相同，则前六条数据序号都为1，而第七条数据从2开始。

```
	-- 按pdate排序
	select dense_rank()over(order by t. pdate asc), t.* from TMDA_REPORT_ENERGY t 
```

##### *ntile(n) 分析函数*

【ntile(n)】分析函数可以对一个数据分区中的有序结果集进行划分，将其分组为各个桶，并为每个小组分配一个唯一的组编号。如果无法均匀的分配，那么多出的数据会被分到第一个分组中。在统计学术语中，NTILE函数创建等宽直方图信息。

> 它把有序的数据集合 平均分配 到 指定的数量（num）个桶中, 将桶号分配给每一行。如果不能平均分配，则优先分配较小编号的桶，并且各个桶中能放的行数最多相差1。假如分成三部分，则分为0-34%-67%-100%。

```
	-- 按pdate排序，然后分组
	select ntile(4) over(order by t. pdate asc), t.* from TMDA_REPORT_ENERGY t 
```


 


# 三、关键字

### 1. delete、drop 和 truncate 的使用

|命令|删除内容|速度|带where|回滚|
|:--|:--|:--|:--|:--|
| delete |用于删除表数据，|删除速度慢|可|可|
| drop |删除表结构和表数据|删除速度快|不可|不可|
| truncate |用于删除表数据，|删除速度快|不可|不可|




# 参考文献
> [oracle排序的几种方法](https://www.cnblogs.com/yeys/p/7647819.html)—— 【探索_之路】

