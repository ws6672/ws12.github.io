---
title: MySQL SQL 练习题合集
date: 2020-09-18 10:30:23
tags: [mysql]
---



### 一、几道SQL查询题

1. 表结构定义
```
	学生表
	1.1 student(Sid,Sname,Sage,Ssex)
	1.2 Sid 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别

	课程表
	2.1 course(Cid,Cname,Tid)
	2.2 Cid 课程编号,Cname 课程名称,Tid 教师编号

	教师表
	3.1 teacher(Tid,Tname)
	3.2 Tid 教师编号,Tname 教师姓名

	成绩表
	4.1 sc(Sid,Cid,score)
	4.2 Sid 学生编号,Cid 课程编号,score 分数
```


2. SQL语句定义

```
	create table student(Sid varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
	insert into student values('01' , '赵雷' , '1990-01-01' , '男');
	insert into student values('02' , '钱电' , '1990-12-21' , '男');
	insert into student values('03' , '孙风' , '1990-05-20' , '男');
	insert into student values('04' , '李云' , '1990-08-06' , '男');
	insert into student values('05' , '周梅' , '1991-12-01' , '女');
	insert into student values('06' , '吴兰' , '1992-03-01' , '女');
	insert into student values('07' , '郑竹' , '1989-07-01' , '女');
	insert into student values('08' , '王菊' , '1990-01-20' , '女');

	create table course(Cid varchar(10),Cname varchar(10),Tid varchar(10));
	insert into course values('01' , '语文' , '02');
	insert into course values('02' , '数学' , '01');
	insert into course values('03' , '英语' , '03');

	create table teacher(Tid varchar(10),Tname varchar(10));
	insert into teacher values('01' , '张三');
	insert into teacher values('02' , '李四');
	insert into teacher values('03' , '王五');

	create table sc(Sid varchar(10),Cid varchar(10),score decimal(18,1));
	insert into sc values('01' , '01' , 80);
	insert into sc values('01' , '02' , 90);
	insert into sc values('01' , '03' , 99);
	insert into sc values('02' , '01' , 70);
	insert into sc values('02' , '02' , 60);
	insert into sc values('02' , '03' , 80);
	insert into sc values('03' , '01' , 80);
	insert into sc values('03' , '02' , 80);
	insert into sc values('03' , '03' , 80);
	insert into sc values('04' , '01' , 50);
	insert into sc values('04' , '02' , 30);
	insert into sc values('04' , '03' , 20);
	insert into sc values('05' , '01' , 76);
	insert into sc values('05' , '02' , 87);
	insert into sc values('06' , '01' , 31);
	insert into sc values('06' , '03' , 34);
	insert into sc values('07' , '02' , 89);
	insert into sc values('07' , '03' , 98);
```

3. 题目

```
	-- 查询" 01 “课程比” 02 "课程成绩高的学生的信息及课程分数

	SELECT  -- 获取所有信息
		st.*,
		temp.score1,
		temp.score2 
	FROM
		(
	SELECT -- 过滤分数
		sc1.score score1,
		sc2.score score2,
		sc1.Sid 
	FROM
		( SELECT * FROM sc WHERE sc.Cid = '01' ) sc1
		JOIN ( SELECT * FROM sc WHERE sc.Cid = '02' ) sc2 ON sc1.Sid = sc2.Sid 
		HAVING
			( sc1.score > sc2.score )
		) temp
		LEFT JOIN student st ON temp.Sid = st.Sid;
		
		

	-- 修改学号为"01"的语文成绩为100
	UPDATE sc
	INNER JOIN ( SELECT * FROM course WHERE course.cname = '语文' ) cs 
	SET sc.score = 100 
	WHERE
		sc.Sid = '01' 
		AND sc.Cid = cs.Cid;

	-- 查询同时存在" 01 “课程和” 02 "课程的情况
	SELECT
		Sid 
	FROM
		sc 
	WHERE
		Cid IN ( '01', '02' ) 
	GROUP BY
		( Sid ) 
	HAVING
		count( Cid ) = 2;
		
		
	-- 查询平均成绩大于70分的学生学号和姓名
	SELECT
		st.Sid,
		st.Sname 
	FROM
		student st
		INNER JOIN ( SELECT Sid, avg( score ) FROM sc GROUP BY Sid HAVING avg( score ) > 70 ) temp ON st.Sid = temp.Sid;
		
	-- 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
	SELECT
		st.Sid,
		st.Sname,
		temp.score 
	FROM
		student st
		INNER JOIN ( SELECT Sid, avg( score ) score FROM sc GROUP BY Sid HAVING avg( score ) >= 85 ) temp ON st.Sid = temp.Sid;

	-- 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
	SELECT
	st.Sname, temp.Cname, temp.score
	FROM
		student st
		INNER JOIN ( SELECT sc.Sid, cs.Cname, sc.score FROM sc inner join course cs on sc.Cid=cs.Cid where score>70 ) temp ON st.Sid = temp.Sid;
		
	-- 查询 1990 年出生的学生名单

	SELECT
	st.Sname, st.Sage
	FROM
		student st where DATE_FORMAT(st.Sage, '%Y') = '1990'
		
	-- 求每门课程的学生人数
	SELECT
		cs.cname,
		temp.sum 
	FROM
		( SELECT cid, count( sid ) sum FROM sc GROUP BY cid ) temp
		LEFT JOIN course cs ON temp.cid = cs.cid;

	-- 查询男生、女生人数
	SELECT
		st.Ssex,
		count( st.Sname ) sum 
	FROM
		student st 
	GROUP BY
		st.Ssex
		
	-- 查询名字中含有「风」字的学生信息
	SELECT
		st.* 
	FROM
		student st where st.Sname like '%风%';
```

### 二、[从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)


***题目***
> 某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。


```

Customers 表：

+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders 表：

+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
例如给定上述表格，你的查询应返回：

+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

***题解***

```
-- 击败了66.68%（内查询）
SELECT
	cs.NAME AS 'Customers' 
FROM
	Customers cs 
WHERE
	Id NOT IN ( SELECT CustomerId FROM Orders );
	
-- 击败了80.48%（左连接）
SELECT
	cs.NAME AS 'Customers' 
FROM
	Customers cs
	LEFT JOIN Orders od ON cs.Id = od.CustomerId 
WHERE
	od.id IS NULL
```


### 三、多表查询的N种情况

这种查询方式主要用于过滤数据，检测数据是否在表中。以 leetcode [从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)为例，来阐述几种情况


表数据如下：

```
Customers 表：

+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders 表：

+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```

***图解如下***

![多表联结](/image/leetcode/join.png)

1. 找出所有客户订购东西的情况（左连接）

```
SELECT
	cs.NAME AS 'Customers', od.*
FROM
	Customers cs
	LEFT JOIN Orders od ON cs.Id = od.CustomerId 
```


2. 找出所有从不订购任何东西的客户（左连接，过滤重叠部分）

```
-- 从所有客户购物情况中过滤掉有购物记录的
SELECT
	cs.NAME AS 'Customers' 
FROM
	Customers cs
	LEFT JOIN Orders od ON cs.Id = od.CustomerId 
WHERE
	od.id IS NULL 
```


3. 找出有订购东西的所有客户（内连接）

```
SELECT
	cs.NAME AS 'Customers'
FROM
	Customers cs
	INNER JOIN Orders od ON cs.Id = od.CustomerId 
```

4. 找出所有用户订单的客户情况（右连接）

```
SELECT
	od.ID AS 'Customers' 
FROM
	Customers cs
	RIGHT JOIN Orders od ON cs.Id = od.CustomerId 
```

5. 找出购物了但没在客户表中的用户（右连接，过滤重叠部分）

```
SELECT
	od.ID AS 'Customers' 
FROM
	Customers cs
	RIGHT JOIN Orders od ON cs.Id = od.CustomerId 
WHERE
	cs.id IS NULL 
```

6. 全连接

Oracle数据库支持full join，mysql是不支持full join的，但仍然可以同过左外连接+ union+右外连接实现

```
SELECT * FROM Customers
LEFT JOIN Orders ON Customers.id = Orders.id
UNION
SELECT * FROM Customers
RIGHT JOIN Orders ON Customers.id = Orders.id
```

7. 全连接，去除重叠部分

```
SELECT * FROM Customers
LEFT JOIN Orders ON Customers.id = Orders.id
UNION
SELECT * FROM Customers
RIGHT JOIN Orders ON Customers.id = Orders.id

WHERE
	cs.id IS NULL OR od.id IS NULL
```



7. `union all`和`union`的区别

显示结果不同 union会自动压缩多个结果集合中的重复结果,而union all则将所有的结果全部显示出来。
对重复结果的处理不同 union all是直接连接,取到得是所有值,记录可能有重复;union 是取唯一值,记录没有重复

8. `left join` 和 `inner join` 的区别

left join 会返回左表所有的数据，即使右表不存在对应数据
inner join 只会返回两个表的交集部分，所以需要有where