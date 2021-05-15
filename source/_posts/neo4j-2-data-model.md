---
title: nexo4j学习指南（二）Nexo4j的使用
date: 2019-08-31 14:46:37
tags: [neo4j]
---

上一篇文章中，可以了解到图数据库的一些基础知识。这一篇文章，是为了说明如何使用图数据库。

# 一、安装与使用
 

### neo4j 的配置
1. 下载 https://neo4j.com/
2. 需要配置JDK, 在命令行输入 java -version查看是否安装成功
3. 配置neo4j的环境变量

```
	NEO4J_HOME  E:\Util\neo4j-community-3.5.8-windows\neo4j-community-3.5.8\bin（包根目录）
	path %NEO4J_HOME%/bin
```

### neo4j 的常用命令
1. cd到bin目录
2. 命令： neo4j { console | start | stop | restart | status } 
3. 开启服务：neo4j console， url：http://127.0.0.1:7474/browser/
4. 默认账号：neo4j neo4j


# 二、neo4j概述


### neo4j 基础
 
+ 是一个高性能的nosql图形数据库，是一个嵌入式的、基于磁盘的、具备完全的事务特性的Java持久化引擎。由于是使用java编程的数据库，所以需要电脑有JDK的支持，即需要安装配置JDK。
+ 我在windows系统中使用的是  community版本，即社区版本，没有图形界面。通过命令行启动后，可以通过网页进行访问。


### neo4j 的四个概念

+ nodes（节点）：图模型的基本单位是节点和关系，都可以包含属性。而节点与关系都是一行数据，有零个或多个标签，用于分组。
+ Relationships（关系）：关系的功能是组织和连接节点，关系可以带有或者没有指向。
+ Properties（属性）：表示节点或关系的一些特性
+ Labels（标签） ：用于分组


# 三、Cypher 的使用


### 什么是 Cypher
“Cypher”是一个描述性的图形查询语言，允许不必编写图形结构的遍历代码对图形存储有表现力和效率的查询。Cypher还在继续发展和成熟，这也就意味着有可能会出现语法的变化。同时也意味着作为组件没有经历严格的性能测试。

### Cypher特性
+ 通过模式匹配数据库中的关系和节点
+ 允许使用变量来命名、绑定元素以及参数
+ 支持索引与约束


这个查询语言分为四个部分：
	
	START：在图中的开始点，通过元素的ID或索引查找获得。
	MATCH：图形的匹配模式，束缚于开始点。
	WHERE：过滤。
	RETURN：返回所需要的；可以多返回，即可以返回多个变量（逗号分隔）
	


	

### Cypher的语法
+ 节点语法：（）、（fo）代表节点
+ 关系语法：（n）-[]-（m） （n）->（m） （n）->（m）, '-'用于连接节点，代表的是关系，有方向加箭头
+ 模式语法：match p=()-[r:friend]->() return p limit 25 #这个语句匹配”()-[r:friend]->()“限定25为最大结果集，把结果存放在变量p，并返回它
+ 标签语法：create（qt:PC{name:"qt"}）-[:simple]-> (java:PC{name:"java"})

 

##### Cypher字句
以下是 cypher 的常用字句，可以实现基本的数据库需求
```
WHERE 提供过滤模式匹配结果的条件
CREATE |CREATE UNIQUE 创建节点和关系
MERGE(merge) 保证查询一定存在，不存在就创建
DELETE 删除节点和关系
set 设置属性值
foreach 遍历列表并更新
UNION 	合并查询结果
with 链式查询。前一个查询的结果作为下一个查询的条件（类似sql的内连接）
start 指定起点的查询，不推荐使用
```
##### 操作符
操作符有三种，数学、等于以及关系：

```
	数学操作符有+，-，*，/和%。当然只有+对字符有作用。
	等于操作符有=，<>，<，>，<=，>=。
```

##### 注解
\/\/单行注解

```
	START n=node(1) RETURN b //这是行结束注释
	START n=node(1) WHERE n.property = "//这部是一个注释" RETURN b
```
##### 增加
创建节点
```
	create({name:"pt",age:11})
```
创建关系
```
	create({name:"aa"})-[:know]->({name:"bb"})-[:know]->({name:"cc"})
```




##### 查询
匹配节点
```
	match p=({name:"pt"}) return p
	MATCH (n{name:"李四"}) RETURN n
```
匹配关系
```
	表示关系：-[r:值]->
	match p=()-[r:friend]->() return p limit 25
```
匹配所有关系
```
	match p=()-[]->() return p limit 25
```
匹配单边节点
```
	match p=({name:"pt"})-[]->() return p limit 25
```
带条件的查询
```
	match(n)-[]->(f) where n.age =11 return f
```
查询返回熟悉
```
	MATCH (people:Person) RETURN people.name LIMIT 10
```


##### 更新与删除
更新属性age为11的节点数据
```
	match (n)-[]->(m) where n.age =11 set n.name = m.age+"_Peter" return n.name
	MATCH (n {name:'张三'}) SET n={age:20} //修改节点信息,覆盖节点属性
```
删除节点熟悉
```
	match (n)-[]->() where n.name = '34_Peter' remove n.age return n
```
删除节点和关系
```
	match (n)-[r]->(m) where n.name = '34_Peter' delete r,m 
```
	
##### 其它命令

查看用户
```
	:server user list
```
添加用户
```
	:server user add
```

##### 索引	

```
	CREATE INDEX ON :Person(name) //为"Person"标签的name属性创建索引
	DROP CONSTRAINT ON (n:Person) ASSERT n.name IS UNIQUE //创建节点属性唯一约束
```
