---
title: mysql 基础语法（三）查询二三事
date: 2020-04-15 14:48:13
tags: [mysql]
---

### 一、运算符的使用

MySQL 数据库中的表结构确立后，表中的数据代表的意义就已经确定。而通过 MySQL 运算符进行运算，就可以获取到表结构以外的另一种数据。

MySQL 支持 4 种运算符，分别是：
1) 算术运算符
执行算术运算，例如：加、减、乘、除等。

|算术运算符|说明|
|:--|:--|
|+|加法运算|
|-|减法运算|
|*|乘法运算|
|/|除法运算，返回商|
|%|求余运算，返回余数|


2) 比较运算符
包括大于、小于、等于或者不等于，等等。主要用于数值的比较、字符串的匹配等方面。例如：LIKE、IN、BETWEEN AND 和 IS NULL 等都是比较运算符，还包括正则表达式的 REGEXP 也是比较运算符。

|比较运算符|说明|
|:--|:--|
|=|等于|
|<|小于|
|<=|小于等于|
|>|大于|
|>=|大于等于|
|<=>|安全的等于，不会返回 UNKNOWN|
|<> 或!=|不等于|
|IS NULL 或 ISNULL|判断一个值是否为 NULL|
|IS NOT NULL|判断一个值是否不为 NULL|
|LEAST|当有两个或多个参数时，返回最小值|
|GREATEST|当有两个或多个参数时，返回最大值|
|BETWEEN AND|判断一个值是否落在两个值之间|
|IN|判断一个值是IN列表中的任意一个值|
|NOT IN|判断一个值不是IN列表中的任意一个值|
|LIKE|通配符匹配|
|REGEXP|正则表达式匹配|


3) 逻辑运算符
包括与、或、非和异或等逻辑运算符。其返回值为布尔型，真值（1 或 true）和假值（0 或 false）。

|逻辑运算符|说明|
|:--|:--|
|NOT 或者 !|逻辑非|
|AND 或者 &&|逻辑与|
|OR 或者 \|\||逻辑或|
|XOR|逻辑异或|


4) 位运算符
包括按位与、按位或、按位取反、按位异或、按位左移和按位右移等位运算符。位运算必须先将数据转换为二进制，然后在二进制格式下进行操作,运算完成后，将二进制的值转换为原来的类型，返回给用户。

|位运算符|说明|
|:--|:--|
|\||按位或|
|&|按位与|
|^|按位异或|
|<<|按位左移|
|>>|按位右移|
|~|按位取反，反转所有比特|


各种符合的优先级如下：

|优先级由低到高排列|运算符|
|:--|:--|
|1|=(赋值运算）、:=|
|2|II、OR|
|3|XOR|
|4|&&、AND|
|5|NOT|
|6|BETWEEN、CASE、WHEN、THEN、ELSE|
|7|=(比较运算）、<=>、>=、>、<=、<、<>、!=、 IS、LIKE、REGEXP、IN|
|8|\||
|9|&|
|10|<<、>>|
|11|-(减号）、+|
|12|*、/、%|
|13|^|
|14|-(负号）、〜（位反转）|
|15|!|


*** 

### 二、查询


***条件查询***

在使用 MySQL SELECT语句时，可以使用 WHERE 子句来指定查询条件，从 FROM 子句的中间结果中选取适当的数据行，达到数据过滤的效果。

语法格式如下：
	`WHERE <查询条件> {<判定运算1>，<判定运算2>，…}`
	
判定运算的语法分类如下：
	+	`<表达式1>{=|<|<=|>|>=|<=>|<>|！=}<表达式2>`
	+	`<表达式1>[NOT]LIKE<表达式2>`
	+	`<表达式1>[NOT][REGEXP|RLIKE]<表达式2>`
	+	`<表达式1>[NOT]BETWEEN<表达式2>AND<表达式3>`
	+	`<表达式1>IS[NOT]NULL`
	
实例：

	```

	-- 单一条件查询
	select t.PMONTH `month`, t.PYEAR `year` from ttt t where t.PYEAR = 2013   limit 5;

	-- 多条件查询
	select t.PMONTH `month`, t.PYEAR `year` from ttt t where t.PYEAR = 2013 and t.PMONTH = 1   limit 5;

	-- 模糊查询
	select t.PMONTH `month`, t.PYEAR `year` from ttt t where t.PYEAR like '201%'   limit 5;

	-- 模糊查询 结尾
	select t.PMONTH `month`, t.PYEAR `year` from ttt t where t.PYEAR like '%3'   limit 5;

	```

***内连接查询***

内连接是通过在查询中设置连接条件的方式，来移除查询结果集中某些数据行后的交叉连接。简单来说，就是利用条件表达式来消除交叉连接的某些数据行。
在 MySQL FROM 子句中使用关键字 INNER JOIN 连接两张表，并使用 ON 子句来设置连接条件。如果没有任何条件，INNER JOIN 和 CROSS JOIN 在语法上是等同的，两者可以互换。

语法格式如下：
	`SELECT <列名1，列名2 …> FROM <表名1> INNER JOIN <表名2> [ ON子句]`


实例：
```
-- 并表查询
 SELECT id,name,age,dept_name
    FROM tb_students_info,tb_departments
		WHERE tb_students_info.dept_id=tb_departments.dept_id;
		
-- 内连接（INNER JOIN）

SELECT id,name,age,dept_name
	FROM tb_students_info INNER JOIN tb_departments
		WHERE tb_students_info.dept_id=tb_departments.dept_id;
```



***LEFT/RIGHT JOIN：外连接查询***

MySQL 中内连接是在交叉连接的结果集上返回满足条件的记录；而外连接先将连接的表分为基表和参考表，再以基表为依据返回满足和不满足条件的记录。

外连接更加注重两张表之间的关系。按照连接表的顺序，可以分为左外连接和右外连接。

在左外连接的结果集中，除了匹配的行之外，还包括左表中有但在右表中不匹配的行，对于这样的行，从右表中选择的列的值被设置为 NULL，即左外连接的结果集中的 NULL 值表示右表中没有找到与左表相符的记录。

右外连接又称为右连接，在 FROM 子句中使用 RIGHT OUTER JOIN 或者 RIGHT JOIN。与左外连接相反，右外连接以右表为基表，连接方法和左外连接相同。在右外连接的结果集中，除了匹配的行外，还包括右表中有但在左表中不匹配的行，对于这样的行，从左表中选择的值被设置为 NULL。


实例：
	```
	-- 左连接
	SELECT*FROM ttt t LEFT JOIN t_bas_unit_rule tr 
		ON t.GRIDCODE=tr.GRIDCODE;

	-- 右连接
	SELECT*FROM ttt t RIGHT JOIN t_bas_unit_rule tr 
		ON t.GRIDCODE=tr.GRIDCODE;
	```

![左右连接](/image/mysql/zylj.png)

***子查询***

子查询指一个查询语句嵌套在另一个查询语句内部的查询，这个特性从 MySQL 4.1 开始引入，在 SELECT 子句中先计算子查询，子查询结果作为外层另一个查询的过滤条件，查询可以基于一个表或者多个表。

子查询中常用的操作符有 ANY（SOME）、ALL、IN 和 EXISTS。子查询可以添加到 SELECT、UPDATE 和 DELETE 语句中，而且可以进行多层嵌套。子查询也可以使用比较运算符，如“<”、“<=”、“>”、“>=”、“！=”等。


实例：

```
-- in 查询
SELECT*FROM ttt t where t.GRIDCODE in(select GRIDCODE from t_bas_unit_rule);

-- not in
SELECT*FROM ttt t where t.GRIDCODE not in(select GRIDCODE from t_bas_unit_rule);


-- EXISTS 查询
SELECT*FROM ttt t where exists(select GRIDCODE from t_bas_unit_rule tr where tr.GRIDCODE = 'CSG' );


-- 运算符
SELECT*FROM ttt t where t.GRIDCODE = (select distinct GRIDCODE from t_bas_unit_rule tr where tr.GRIDCODE = 'CSG');

```


***分组查询***
在 MySQL SELECT 语句中，允许使用 GROUP BY 子句，将结果集中的数据行根据选择列的值进行逻辑分组，以便能汇总表内容的子集，实现对每个组而不是对整个结果集进行整合。

语法格式如下：
	`GROUP BY { <列名> | <表达式> | <位置> } [ASC | DESC]`

对于 GROUP BY 子句的使用，需要注意以下几点。
+	GROUP BY 子句可以包含任意数目的列，使其可以对分组进行嵌套，为数据分组提供更加细致的控制。
+	GROUP BY 子句列出的每个列都必须是检索列或有效的表达式，但不能是聚合函数。若在 SELECT 语句中使用表达式，则必须在 GROUP BY 子句中指定相同的表达式。
+	除聚合函数之外，SELECT 语句中的每个列都必须在 GROUP BY 子句中给出。
+	若用于分组的列中包含有 NULL 值，则 NULL 将作为一个单独的分组返回；若该列中存在多个 NULL 值，则将这些 NULL 值所在的行分为一组。


***指定过滤条件***
在 MySQL SELECT 语句中，除了能使用 GROUP BY 子句分组数据外，还可以使用 HAVING 子句过滤分组，在结果集中规定了包含哪些分组和排除哪些分组。

语法格式如下：
	`HAVING <条件>`
	
HAVING 子句和 WHERE 子句非常相似，HAVING 子句支持 WHERE 子句中所有的操作符和语法，但是两者存在几点差异：
+	WHERE 子句主要用于过滤数据行，而 HAVING 子句主要用于过滤分组，即 HAVING 子句基于分组的聚合值而不是特定行的值来过滤数据，主要用来过滤分组。
+	WHERE 子句不可以包含聚合函数，HAVING 子句中的条件可以包含聚合函数。
+	HAVING 子句是在数据分组后进行过滤，WHERE 子句会在数据分组前进行过滤。WHERE 子句排除的行不包含在分组中，可能会影响 HAVING 子句基于这些值过滤掉的分组。


***

### 三、其余操作


***去重查询***


通过`distinct`去重，它往往用来查询不重复的具体条数，而不是具体数据。因为，它只能返回目标字段，而无法返回其它字段。
```
-- 去重
select distinct PMONTH from ttt 

```

如果使用多个字段，那么久需要多个字段不同：
```
select *, count(distinct PMONTH) from ttt group by PMONTH
```

***别名***

为表指定别名的基本语法格式为：
`<表名> [AS] <别名>`

实例：
```
-- 表别名
select t.PMONTH, t.PYEAR from ttt t;

-- 字段别名
select t.PMONTH `month`, t.PYEAR `year` from ttt t;

```

***限制查询结果的记录条数***

基本的语法格式如下：
	`<LIMIT> [<位置偏移量>,] <行数>`

实例：
```
-- 查询至多五行
select t.PMONTH `month`, t.PYEAR `year` from ttt t limit 5;

-- 偏移三行，查询至多五行
select t.PMONTH `month`, t.PYEAR `year` from ttt t limit 3, 5;

```


***对查询结果进行排序***

其语法格式为：
	`ORDER BY {<列名> | <表达式> | <位置>} [ASC|DESC]`
	
关键字 ASC 表示按升序分组，关键字 DESC 表示按降序分组，其中 ASC 为默认值。这两个关键字必须位于对应的列名、表达式、列的位置之后。


实例：

```
select t.PMONTH `month`, t.PYEAR `year` from ttt t order by `year`,`month` limit 5;
```	
