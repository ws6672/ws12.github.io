---
title: mysql 基础语法（一）
date: 2020-04-14 15:51:34
tags: [mysql]
---

### 一、数据库

***1.创建***

```
CREATE DATABASE [IF NOT EXISTS] <数据库名>
[[DEFAULT] CHARACTER SET <字符集名>] 
[[DEFAULT] COLLATE <校对规则名>];
```
> 注：[]表示可选，不设置就系统默认。`IF NOT EXISTS`表示不存在就创建。`COLLATE` 指定字符集的默认校对规则。


实例：
`create database if not exists mysqlt character set 'utf8';`

***2.查询***

在 MySQL 中，可使用 SHOW DATABASES 语句来查看或显示当前用户权限范围以内的数据库。查看数据库的语法格式为：

```
SHOW DATABASES [LIKE '数据库名'];
```

实例：
`show DATABASEs like '%mysql%';`

默认的几个表如下：
+	information_schema  主要存储了系统中的一些数据库对象信息
+	mysql	MySQL 的核心数据库，类似于 SQL Server 中的 master 表，主要负责存储数据库用户、用户访问权限等 MySQL 自己需要使用的控制和管理信息              
+	performance_schema	主要用于收集数据库服务器性能参数。
+	sakila	样例数据库，类似于表模板             
+	sys	MySQL 5.7 安装完成后会多一个 sys 数据库。sys 数据库主要提供了一些视图，数据都来自于 performation_schema，主要是让开发者和使用者更方便地查看性能问题。
+	world	world 数据库是 MySQL 自动创建的数据库，该数据库中只包括 3 张数据表，分别保存城市，国家和国家使用的语言等内容


***3.修改***

我们可以通过语法对数据库的字符集和校对规则进行修改，相关信息在`db.opt`文件中，语法如下

```
ALTER DATABASE [数据库名] { 
[ DEFAULT ] CHARACTER SET <字符集名> |
[ DEFAULT ] COLLATE <校对规则名>}
```

说明：
+	ALTER DATABASE可用于修改数据库的字符集以及校对规则；
+	数据库名称可忽略，表示所有默认数据库
+	用户需要拥有 `ALTER权限`

***4.删除***

当数据库不再使用时应该将其删除，以确保数据库存储空间中存放的是有效数据。删除数据库是将已经存在的数据库从磁盘空间上清除，清除之后，数据库中的所有数据也将一同被删除。语法如下：

```
DROP DATABASE [ IF EXISTS ] <数据库名>
```

实例：
`drop DATABASE mysqlt;`

> 注：MySQL 安装后，系统会自动创建名为 information_schema 和 mysql 的两个系统数据库，系统数据库存放一些和数据库相关的信息，如果删除了这两个数据库，MySQL 将不能正常工作。

***5.选择***

在 MYSQL中，通过`USE`切换数据库，语法如下：

```
USE <数据库名>;
```

***

### 二、存储引擎

***1. 什么是存储引擎***

数据库存储引擎是数据库底层软件组件，数据库管理系统使用数据引擎进行创建、查询、更新和删除操作。`不同的存储引擎提供不同的存储机制、索引类型、锁定水平等功能，还拥有不同的特定功能。`

***2. 查看系统支持的存储引擎***

`show ENGINEs;`

结果：
![存储引擎](/image/mysql/engines.png)

其中，
+	Support 列的值表示某种引擎是否能使用，YES表示可以使用，NO表示不能使用；DEFAULT表示该引擎为当前默认的存储引擎。
+	Transactions 表示是否支持事务；
+	XA 表示是否支持分布式事务；
+	Savapoints 表示是否支持还原点。


***3. 存储引擎的抉择***

存储引擎的相关功能：

|功能|MylSAM|MEMORY|InnoDB|Archive|
|:--|:--|:--|:--|:--|
|存储限制|256TB|RAM|64TB|None|
|支持事务|No|No|Yes|No|
|支持全文索引|Yes|No|No|No|
|支持树索引|Yes|Yes|Yes|No|
|支持哈希索引|No|Yes|No|No|
|支持数据缓存|No|N/A|Yes|No|
|支持外键|No|No|Yes|No|

存储引擎选择原则：

+	需要支持事务，支持ACID,并可以并发控制，选择 `InnoDB`
+	数据表主要用于插入与查询，则 MyISAM 引擎提供较高的处理效率
+	如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存的 MEMORY 引擎中，MySQL 中使用该引擎作为临时表，存放查询的中间结果。
+	数据表如果只有插入与查询，当只有固定配置的数据，适合使用`Archive`，避免数据被修改

> 注：InnoDB 事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键。MySQL 5.5.5 之后，InnoDB 作为默认存储引擎。
`MyISAM 是基于 ISAM 的存储引擎`，并对其进行扩展，是在 Web、数据仓储和其他应用环境下最常使用的存储引擎之一。`MyISAM 拥有较高的插入、查询速度，但不支持事务`。
MEMORY 存储引擎将表中的数据存储到内存中，为查询和引用其他数据提供快速访问。

***4. 修改默认存储引擎***

使用下面的语句可以修改数据库临时的默认存储引擎:
```
SET default_storage_engine=< 存储引擎名 >
```

***


### 三、mysql中的数据类型

***1.MySQL数据类型简介***

数据类型（data_type）是指系统中所允许的数据的类型。MySQL 数据类型定义了列中可以存储什么数据以及该数据怎样存储的规则。

MySQL 的数据类型有大概可以分为 5 种，分别是整数类型、浮点数类型和定点数类型、日期和时间类型、字符串类型、二进制类型等。


***2.数值类型***

+	*整数类型*包括 TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT

|类型名称|说明|存储需求|说明|存储需求|
|:--|:--|:--|:--|:--|
|TINYINT|很小的整数|1个字节|-128〜127|0 〜255|
|SMALLINT|小的整数|2个宇节|-32768〜32767|0〜65535|
|MEDIUMINT|中等大小的整数|3个字节|-8388608〜8388607|0〜16777215|
|INT (INTEGHR)|普通大小的整数|4个字节|-2147483648〜2147483647|0〜4294967295|
|BIGINT|大整数|8个字节|-9223372036854775808〜9223372036854775807|0〜18446744073709551615|
		
+	*浮点数*类型包括 FLOAT 和 DOUBLE，定点数类型为 DECIMAL。
	+	浮点类型和定点类型都可以用(M, D)来表示，其中M称为精度，表示总共的位数；D称为标度，表示小数的位数。
	+	`浮点数类型的取值范围为 M（1～255）和 D（1～30，且不能大于 M-2）。` DECIMAL 的默认 D 值为 0、M 值为 10。
		
|类型名称|说明|存储需求|
|:--|:--|:--|
|FLOAT|单精度浮点数|4 个字节|
|DOUBLE|双精度浮点数|8 个字节|
|DECIMAL (M, D)，DEC|压缩的“严格”定点数|M+2 个字节|
			
> 整型数据类型可以在定义表结构时指定所需的显示宽度，如果不指定，则系统为每一种类型指定默认的宽度值。

***3.日期/时间类型***
时间类型包括 YEAR、TIME、DATE、DATETIME 和 TIMESTAMP。

+	YEAR:非法 YEAR值将被转换为 0000。
+	TIME  数据格式为` HH:MM:SS`,也可以不用冒号隔开，例如:`101112`,会被解释成`10:11:12`,但是`106112`是不合法的，会被解释成`00:00:00`。
+	DATE:日期格式为 'YYYY-MM-DD'
+	DATETIME ：日期格式为 'YYYY-MM-DD HH：MM：SS'，
+	TIMESTAMP ：日期格式为` YYYY-MM-DD HH：MM：SS`。但是 TIMESTAMP 列的取值范围小于 DATETIME 的取值范围，为 '1970-01-01 00：00：01'UTC～'2038-01-19 03：14：07'UTC。在插入数据时，要保证在合法的取值范围内。

|类型名称|日期格式|日期范围|存储需求|
|:--|:--|:--|:--|
|YEAR|YYYY|1901 ~ 2155|1 个字节|
|TIME|HH:MM:SS|-838:59:59 ~ 838:59:59|3 个字节|
|DATE|YYYY-MM-DD|1000-01-01 ~ 9999-12-3|3 个字节|
|DATETIME|YYYY-MM-DD HH:MM:SS|1000-01-01 00:00:00 ~ 9999-12-31 23:59:59|8 个字节|
|TIMESTAMP|YYYY-MM-DD HH:MM:SS|1980-01-01 00:00:01 UTC ~ 2040-01-19 03:14:07 UTC|4 个字节|


> 提示：MySQL 允许“不严格”语法：任何标点符号都可以用作日期部分之间的间隔符。例如，'98-11-31'、'98.11.31'、'98/11/31'和'98@11@31' 是等价的，这些值也可以正确地插入数据库。


***4. 字符串类型***

字符串类型包括 CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM 和 SET 等。

+	CHAR(M) 为固定长度字符串，在定义时指定字符串列长。当检索到 CHAR 值时，尾部的空格将被删除。
+	VARCHAR(M) 是长度可变的字符串，M 表示最大列的长度，M 的范围是 0～65535。VARCHAR 的最大实际长度由最长的行的大小和使用的字符集确定，而实际占用的空间为字符串的实际长度加 1。
+	TEXT 列保存非二进制字符串，如文章内容、评论等。当保存或查询 TEXT 列的值时，不删除尾部空格。
	+	TEXT 类型分为 4 种：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。不同的 TEXT 类型的存储空间和数据长度不同。
+	ENUM 是一个字符串对象，值为表创建时列规定中枚举的一列值。其语法格式如下：`<字段名> ENUM( '值1', '值1', …, '值n' )`
+	SET 是一个字符串的对象，可以有零或多个值，SET 列最多可以有 64 个成员，值为表创建时规定的一列值。指定包括多个 SET 成员的 SET 列值时，各成员之间用逗号,隔开，语法格式如下：`SET( '值1', '值2', …, '值n' )`


|类型名称|说明|存储需求|
|:--|:--|:--|
|CHAR(M)|固定长度非二进制字符串|M 字节，1<=M<=255|
|VARCHAR(M)|变长非二进制字符串|L+1字节，在此，L< = M和 1<=M<=255|
|TINYTEXT|非常小的非二进制字符串|L+1字节，在此，L<2^8|
|TEXT|小的非二进制字符串|L+2字节，在此，L<2^16|
|MEDIUMTEXT|中等大小的非二进制字符串|L+3字节，在此，L<2^24|
|LONGTEXT|大的非二进制字符串|L+4字节，在此，L<2^32|
|ENUM|枚举类型，只能有一个枚举字符串值|1或2个字节，取决于枚举值的数目 (最大值为65535)|
|SET|一个设置，字符串对象可以有零个或 多个SET成员|1、2、3、4或8个字节，取决于集合 成员的数量（最多64个成员）|

***5. 二进制类型***

二进制类型包括 BIT、BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。



|类型名称|说明|存储需求|
|:--|:--|:--|
|BIT(M)|位字段类型|大约 (M+7)/8 字节|
|BINARY(M)|固定长度二进制字符串|M 字节|
|VARBINARY (M)|可变长度二进制字符串|M+1 字节|
|TINYBLOB (M)|非常小的BLOB|L+1 字节，在此，L<2^8|
|BLOB (M)|小 BLOB|L+2 字节，在此，L<2^16|
|MEDIUMBLOB (M)|中等大小的BLOB|L+3 字节，在此，L<2^24|
|LONGBLOB (M)|非常大的BLOB|L+4 字节，在此，L<2^32|





### 四、数据表


***1.创建***

创建数据表的过程是规定数据列的属性的过程，同时也是实施数据完整性（包括实体完整性、引用完整性和域完整性）约束的过程。接下来我们介绍一下创建数据表的语法形式。

```
CREATE TABLE <表名> ([
<列名1> <类型1> [,…] <列名n> <类型n>
])[表选项][分区选项];
```
CREATE TABLE 命令语法比较多，其主要是由以下几部分组成：

+	表名
	+	可以创建指定数据库的表（` 'db_name'.'tbl_nam'`），也可以忽略数据库，表示在当前数据下创建
+	表创建定义（create-definition）
	+	表创建定义，由列名（col_name）、列的定义（column_definition）以及可能的空值说明、完整性约束或表索引组成。
+	表选项（table-options）
+	分区选项（partition-options）


注意事项如下：
+	要创建的表的名称不区分大小写，不能使用SQL语言中的关键字，如DROP、ALTER、INSERT等。
+	数据表中每个列（字段）的名称和数据类型，如果创建多个列，要用逗号隔开。

实例:

```
create table test (
	id int(11),
	name VARCHAR(40),
	salary FLOAT
);
```

***2. 查看表结构***
DESCRIBE/DESC 语句可以查看表的字段信息，包括字段名、字段数据类型、是否为主键、是否有默认值等，语法规则如下：

```
describe tb_name;
```

例如：`describe test;`

***3. 修改***

在 MySQL 中可以使用 ALTER TABLE 语句来改变原有表的结构，例如增加或删减列、创建或取消索引、更改原有列类型、重新命名列或表等。

```
ALTER TABLE <表名> 

{ ADD COLUMN <列名> <类型> -- 添加字段
| CHANGE COLUMN <旧列名> <新列名> <新列类型> -- 修改字段名
| ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT } -- 修改字段数据类型
| MODIFY COLUMN <列名> <类型> -- 修改字段数据类型
| DROP COLUMN <列名>  --删除字段
| RENAME TO <新表名> } --修改表名
```

***4. 删除***

在删除表的同时，表的结构和表中所有的数据都会被删除，因此在删除数据表之前最好先备份，以免造成无法挽回的损失。语法如下

```
DROP TABLE [IF EXISTS] 表名1 [ ,表名2, 表名3 ...]
```


### 五、表约束

查询表约束：
`SHOW create table test2`

***1. 主键约束***

`PRIMARY KEY` 是主键约束，使用如下：

```
create table test (
	id int(11) PRIMARY key,
	name VARCHAR(40),
	salary FLOAT
)
```

设置复合主键：
```
create table test (
	id int(11) ,
	name VARCHAR(40),
	salary FLOAT,
	PRIMARY key(id,name)
)
```
修改表时添加主键：
```
alter table test add primary key (salary);
```

***2. 外键约束***

MySQL 外键约束（FOREIGN KEY）用来在两个表的数据之间建立链接，它可以是一列或者多列。一个表可以有一个或多个外键。
外键的主要作用是保持数据的一致性、完整性。

外键规则：
+	父表已存在，且主键存在。
+	外键的非空值需要在主表中可查
+	外键中列的数目必须和父表的主键中列的数目相同。
+	外键中列的数据类型必须和父表主键中对应列的数据类型相同。

实例：
```
create table test2 (
	id int(10) PRIMARY key,
	t_name VARCHAR(100),
	test_id int(11),
	constraint `test2_test` foreign key(`test_id`) references   `test`(`id`)
);
```

删除外键约束：
```
alter table test2 drop foreign key `test2_test`
```

修改表时添加外键约束：
```
alter table test2 ADD constraint `test2_test` foreign key (`test_id`) references `test`(`id`); 
```


***3. 唯一约束***

在定义完列之后直接使用 UNIQUE 关键字指定唯一约束，语法规则如下：

```
<字段名> <数据类型> UNIQUE
```

修改表时添加:
```
alter table test2 ADD constraint `test2_unique` unique(t_name)
```

删除唯一约束：
```
-- alter table `表名` drop index `约束名`

alter table `test2` drop index 	`test2_unique`
```

***4. 检查约束（CHECK）***

检查约束使用 CHECK 关键字，具体的语法格式如下：
`CHECK <表达式>`


修改表时添加的实例：
```
--	alter table `表名`  ADD CONSTRAINT 约束名  CHECK(表达式);
	alter table `test2`  ADD CONSTRAINT TEST2_CHECK  CHECK( t_name != 'mk' and t_name != 'jk');
```

> 注：若将 CHECK 约束子句置于所有列的定义以及主键约束和外键定义之后，则这种约束也称为基于表的 CHECK 约束。该约束可以同时对表中多个列设置限定条件。

***5. 默认值（DEFAULT）***

创建表时可以使用 DEFAULT 关键字设置默认值约束，具体的语法规则如下：
`<字段名> <数据类型> DEFAULT <默认值>;`

在修改表时添加默认值约束:

```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <数据类型> DEFAULT <默认值>;
```

删除默认值约束:
```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> DEFAULT NULL;
```

***6. 非空约束（NOT NULL）***

创建表时可以使用 NOT NULL 关键字设置非空约束，具体的语法规则如下：
`<字段名> <数据类型> NOT NULL;`

在修改表时添加非空约束
```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名>
<字段名> <数据类型> NOT NULL;
```

删除非空约束

```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> NULL;
```

***实例***

```
create table tt2(
	id int(10) primary key,  -- 主键
	name VARCHAR(80) not null, -- 非空约束
	salary int(10) default 1000, -- 默认值
	test2_id int(11) unique, -- 唯一约束
	constraint check_constraint check(salary >=0), -- 检查约束
	constraint foreign_constraint foreign key(test2_id) references test2(id)  -- 外键
);
```

### 六、数据

***插入数据（添加数据）***

INSERT 语句有两种语法形式，分别是 INSERT…VALUES 语句和 INSERT…SET 语句:


1) INSERT…VALUES语句
INSERT VALUES 的语法格式为：
	```INSERT INTO <表名> [ <列名1> [ , … <列名n>] ]
	VALUES (值1) [… , (值n) ];```
	
	
2) INSERT…SET语句
语法格式为：
	```INSERT INTO <表名>
	SET <列名1> = <值1>,
			<列名2> = <值2>,
			…```
			
			

***修改数据（更新数据）***

使用 UPDATE 语句修改单个表，语法格式为：
```UPDATE <表名> SET 字段 1=值 1 [,字段 2=值 2… ] [WHERE 子句 ]
[ORDER BY 子句] [LIMIT 子句]```

语法说明如下：
<表名>：用于指定要更新的表名称。
+	SET 子句：用于指定表中要修改的列名及其列值。其中，每个指定的列值可以是表达式，也可以是该列对应的默认值。如果指定的是默认值，可用关键字 DEFAULT 表示列值。
+	WHERE 子句：可选项。用于限定表中要修改的行。若不指定，则修改表中所有的行。
+	ORDER BY 子句：可选项。用于限定表中的行被修改的次序。
+	LIMIT 子句：可选项。用于限定被修改的行数。

> 注：修改一行数据的多个列值时，SET 子句的每个值用逗号分开即可。



***删除数据***
 
在 MySQL 中，可以使用 DELETE 语句来删除表的一行或者多行数据。
使用 DELETE 语句从单个表中删除数据，语法格式为：
	```DELETE FROM <表名> [WHERE 子句] [ORDER BY 子句] [LIMIT 子句]```