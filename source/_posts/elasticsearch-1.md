---
title: ES（一）数据处理
date: 2020-07-30 13:09:04
tags: [elasticsearch]
---

组织数据是为我们的目标所服务，而数据库是为数据服务的。Elasticsearch 为数据提供了`基于分布式的全文搜索服务`。在深入学习前，需要了解一些基本的语法与知识。


### 1. 概述

对象(object)是一种语言相关，记录在内存中的的数据结构。为了在网络间发送、存储它，我们需要一些标准的格式来表示它。JSON (JavaScript Object Notation)是一种`可读的以文本来表示对象`的方式。它已经成为NoSQL世界中数据交换的一种事实标准。当对象被序列 化为JSON，它就成为JSON文档(JSON document)了。

Elasticsearch是一个分布式的文档(document)存储引擎。它可以实时存储并检索复杂数据结构——序列化的JSON文档。换言说，一旦文档被存储在Elasticsearch中，它就可以在集群的任一节点上被检索。

`在Elasticsearch中，每一个字段的数据都是默认被索引的`。也就是说，每个字段专门有一个`反向索引`用于快速检索。而且，与其它数据库不同，它可以在同一个查询中利用所有的这些反向索引，以惊人的速度返回结果。


> 注：反向索引：反向索引方向则是正向索引的逆向，建立从单词 (word) 到文档 (document lsit) 的映射关系。


##### 1.1 什么是文档？ 

程序中大多的实体或对象能够被序列化为包含键值对的JSON对象，键(key)是字段(field)或属 性(property)的名字，值(value)可以是字符串、数字、布尔类型、另一个对象、值数组或者 其他特殊类型，比如表示日期的字符串或者表示地理位置的对象。
在Elasticsearch中，文档(document)这个术语有着 特殊含义。它特指最顶层结构或者根对象(root object)序列化成的JSON数据（以唯一ID标识 并存储于Elasticsearch中）。


##### 1.2 文档结构

+	元数据（metadata）
	+	_index(逻辑上的命名空间|索引):文档在哪存放
		+	在Elasticsearch中，每一个字段的数据都是默认被索引的。也就是说，每个字段专门有一个`反向索引`用于快速检索。
		+	索引名：这个名字必须是全部小写，不能以下划线开头，不能包含逗号
	+	~_type:文档表示的对象类别~
	+	_id:文档唯一标识;定义或ES生成
		+	自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally unique identifiers, 或者叫 UUIDs


注：数据被存储和索引在分片(shards)中，索引只是一个把一个或多个分片 分组在一起的逻辑空间。


### 2. 基本语法

***访问的两种方式***

```
# 通过Linux终端
[cen@192 ~]$ curl -i -XGET http://localhost:9200/website/blog/124?pretty

# 通过 kibana网页端的控制台：http://127.0.0.1:5601/app/kibana#/dev_tools/console
# 该控制台简化了curl工具的使用
GET /website/blog/124

```

##### 2.1 增删改查

***添加文档***

```
# 添加索引文档
# _index、_id 唯一确定一个文档
PUT /{index}/{type}/{id} { "field": "value", ... }

# 自定义ID
PUT /website/123 { "title": "My first blog entry", "text": "Just trying this out...", "date": "2014/01/01" }

# 自动生成的ID
POST /website/ { "title": "My second blog entry", "text": "Still trying this out...", "date": "2014/01/01" }
```

***检索文档***

```
# 检索文档
# `?pretty`参数：用于美化输出
GET /website/123?pretty
curl -i -XGET http://localhost:9200/website/blog/124?pretty


# 检索文档-部分参数
GET /website/123?_source=title,text
GET /website/123/_source

# 检查文档是否存在
curl -i -XHEAD http://localhost:9200/website/123
	-- 不存在返回 404
	-- 文档存在返回200
```

***更新文档***

文档在Elasticsearch中是不可变的——我们不能修改他们。如果需要更新已存在的文档，我们可以使用index API 重建索引(reindex) 或者替换掉它。

文档更新的流程如下：
1. 从旧文档中检索JSON 
2. 修改它 
3. 隐藏旧文档 
4. 索引新文档

在内部，Elasticsearch已经标记旧文档为删除并添加了一个完整的新文档。旧版本文档不会 立即消失，但你也不能去访问它。Elasticsearch会在你继续索引更多数据时清理被删除的文 档

```
POST /megacorp/_doc/9z_oYXMBgrJR-nh9qWAR
{"age":1111}
	{
	  "_index" : "megacorp",
	  "_type" : "_doc",
	  "_id" : "9z_oYXMBgrJR-nh9qWAR",
	  "_version" : 2,
	  "result" : "updated",
	  "_shards" : {
		"total" : 2,
		"successful" : 1,
		"failed" : 0
	  },
	  "_seq_no" : 8,
	  "_primary_term" : 3
	}
```

***删除文档***

删除一个文档也不会立即从磁盘上移除，它只是被 标记成已删除。Elasticsearch将会在你之后添加更多索引的时候才会在后台进行删除内 容的清理。

```
# DELETE /website/blog/123  返回值：200/404


DELETE /megacorp/_doc/9z_oYXMBgrJR-nh9qWAR?pretty
	{
	  "_index" : "megacorp",
	  "_type" : "_doc",
	  "_id" : "9z_oYXMBgrJR-nh9qWAR",
	  "_version" : 3,
	  "result" : "deleted",
	  "_shards" : {
		"total" : 2,
		"successful" : 1,
		"failed" : 0
	  },
	  "_seq_no" : 9,
	  "_primary_term" : 3
	}

# 如果文档不存在—— "result" 的值是 "not_found" —— _version 依旧增加了。这是内部记录的一部 分，它确保在多节点间不同操作可以有正确的顺序
	{
	  "_index" : "megacorp",
	  "_type" : "_doc",
	  "_id" : "9z_oYXMBgrJR-nh9qWAR",
	  "_version" : 4,
	  "result" : "not_found",
	  "_shards" : {
		"total" : 2,
		"successful" : 1,
		"failed" : 0
	  },
	  "_seq_no" : 10,
	  "_primary_term" : 3
	}


```


> UUIDs:自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally unique identifiers, 或者叫 UUIDs。
pretty:在任意的查询字符串中增加 pretty 参数，类似于上面的例子。会让Elasticsearch美化输 出(pretty-print)JSON响应以便更加容易阅读。 _source 字段不会被美化，它的样子与我 们输入的一致。

##### 2.2 处理冲突

当使用 index API更新文档的时候，我们读取原始文档做修改，然后将整个文档(whole document)一次性重新索引，只有最近的索引请求会生效——Elasticsearch中只存储最后被索引 的任何文档。如果其他人同时也修改了这个文档，他们的修改将会丢失。

***悲观并发控制（Pessimistic concurrency control）***

悲观并发控制:认为冲突一定会发生，所以在读数据前会提前锁定一行。

这在关系型数据库中被广泛的使用，假设冲突的更改经常发生，为了解决冲突我们把访问区 块化。典型的例子是在读一行数据前锁定这行，然后确保只有加锁的那个线程可以修改这行 数据。


***乐观并发控制（Optimistic concurrency control）***

乐观并发控制:认为冲突一定不会发生，所以在写数据前才会提前锁定一行。

+	被Elasticsearch使用，假设冲突不经常发生，也不区块化访问，然而，如果在读写过程中数 据发生了变化，更新操作将失败。这时候由程序决定在失败后如何解决冲突。实际情况中， 可以重新尝试更新，刷新数据（重新读取）或者直接反馈给用户。

+	Elasticsearch是分布式的。当文档被创建、更新或删除，文档的新版本会被复制到集群的其 它节点。Elasticsearch即是同步的又是异步的，意思是这些复制请求都是平行发送的，并无 序(out of sequence)的到达目的地。这就需要一种方法确保老版本的文档永远不会覆盖新的 版本。

+	`每个文档都有一个 _version 号码， 这个号码在文档被改变时加一。Elasticsearch使用这个 _version 保证所有修改都被正确排 序`。当一个旧版本出现在新版本之后，它会被简单的忽略。

`实例`

```
# 发起请求
GET /website/blog/1

# 请求成功

{ "_index": "website", "_type": "blog", "_id": "1", "_version": 2 "created": false }

# 请求冲突
{ "error" : "VersionConflictEngineException[[website][2] [blog][1]: version conflict, current [2], provided [1]]", "status" : 409 }
```

***使用外部版本控制系统***

一种常见的结构是使用一些其他的数据库做为主数据库，然后使用Elasticsearch搜索数据， 这意味着所有主数据库发生变化，就要将其拷贝到Elasticsearch中。如果有多个进程负责这 些数据的同步，就会遇到上面提到的并发问题。

如果主数据库有版本字段——或一些类似于 timestamp 等可以用于版本控制的字段——是你 就可以在Elasticsearch的查询字符串后面添加 version_type=external 来使用这些版本号。版 本号必须是整数，大于零小于 9.2e+18 ——Java中的正的 long。


如果冲突发生，我们唯一要做的仅仅是重新尝试更新既可。


##### 2.4 局部更新

通过使用 update API，我们可以使用一个请求来实现局部更新，例如增加 数量的操作。 
文档是不可变的——它们不能被更改，只能被替换。 update API必须遵循相同的 规则。表面看来，我们似乎是局部更新了文档的位置，内部却是像我们之前说的一样简单的 使用 update API处理相同的检索-修改-重建索引流程，我们也减少了其他进程可能导致冲突 的修改。


```
POST /website/blog/1/_update { "doc" : { "tags" : [ "testing" ], "views": 0 } }

```

***使用Groovy脚本局部更新***
当API不能满足要求时，Elasticsearch允许你使用脚本实现自己的逻辑。脚本支持 非常多的API，例如搜索、排序、聚合和文档更新。脚本可以通过请求的一部分、检索特 殊的 .scripts 索引或者从磁盘加载方式执行。

默认的脚本语言是`Groovy`，一个快速且功能丰富的脚本语言，语法类似于Javascript。它 在一个沙盒(sandbox)中运行，以防止恶意用户毁坏Elasticsearch或攻击服务器。

```
POST /website/blog/1/_update { "script" : "ctx._source.views+=1" }
```

***更新可能不存在的文档***

当我们试图更新一个 不存在的文档，更新将失败。在这种情况下，我们可以使用 upsert 参数定义文档来使其不存在时被创建。

相关命令如下：`POST /website/pageviews/1/_update { "script" : "ctx._source.views+=1", "upsert": { "views": 1 } }`

***更新和冲突***

按照 Elasticsearch的数据逻辑，当更新发生冲突的时候只需要重新尝试更新即可。这些可以通过 retry_on_conflict 参数设置重试次数来自动完成，这样 update 操作将会在发 生错误前重试——这个值默认为0。

```
POST /website/pageviews/1/_update?retry_on_conflict=5 
{ 
	"script" : "ctx._source.views+=1", "upsert": { "views": 0 } 
}
```


##### 2.5 检索多个文档

像Elasticsearch一样，检索多个文档依旧非常快。合并多个请求可以避免每个请求单独的网 络开销。如果你需要从Elasticsearch中检索多个文档，相对于一个一个的检索，更快的方式 是在一个请求中使用multi-get或者 mget API。

mget API参数是一个 docs 数组，数组的每个节点定义一个文档 的 _index 、 _type 、 _id 元数据。如果你只想检索一个或几个确定的字段，也可以定义一 个 _source 参数：

```
POST /_mget 
{ "docs" : 
	[ 
		{ "_index" : "megacorp","_id":"9j_oYXMBgrJR-nh9XWCm"},
		{ "_index" : "megacorp","_id":"-j_tYXMBgrJR-nh9M2Al"} 
	] 
}

# 文档不存在并不影响第一个文档的检索。每个文档的检索和报告都是独立的。
POST /website/blog/_mget { "ids" : [ "2", "1" ] }
```

### 3. 索引

通过创建数据创建索引库虽然省事，但是没有创建索引库而通过ES自身生成的这种并不友好，因为它会使用默认的配置，字段结构都是text(text的数据会分词，在存储的时候也会额外的占用空间)，分片和索引副本采用默认值，默认是5和1，ES的分片数在创建之后就不能修改，除非reindex(下面会讲到)，所以这里我们还是指定数据模板进行创建。


##### 3.1 ES的数据结构

***基本数据类型***

1. 核心数据类型
+	Text（分词）
+	Keyword（索引ID）
	+	若不用于range查询则可以使用keyword代替,因为keyword类型字段针对term或term-level的查询更加友好
	
2. 数值数据类型（numeric类型）
+	long	-2^63 ~ 2^63-1
+	integer	-2^31 ~ 2^31-1
+	short	-32768 ~ 32767
+	byte	-128 ~ 127
+	double	64位精度
+	float	32位精度
+	half_float	16位精度
+	scaled_float	可配置scaling_factor参数 

	
3. 日期数据类型
+	json格式没有date类型。日期查询会在内部转换为long类型形式的范围查询,并且聚合和存储字段的结果将转换为字符串(具体取决于该字段的日期转换格式);日期将始终以字符串形式呈现,即使一开始在JSON文档中提供的类型为long
+	可以使用`||`分隔符指定多种日期格式,es会依次尝试每种格式,直到找到匹配的格式
	
4. 布尔数据类型（boolean）
5. 二进制数据类型（binary）
+	存储字符串base64处理之后的值
	
6. 范围数据类型（range类型）
+	integer_range
+	float_rangellong_range
+	double_range
+	date_range
+	ip_range



***复杂数据类型***

对象数据类型
	
	object 用于单个JSON对象

嵌套数据类型（nested 用于JSON对象数组）
+	nested类型是object类型的一种特殊形式,其允许对象数组被索引且可以独立进行查询
+	es没有内部对象的概念,因此其将对象继承结构转成简单的key-value的list结构

***地理数据类型***

地理位置数据类型

	geo_point 纬度/经度积分

地理形状数据类型

	geo_shape 用于多边形等复杂形状

***专业数据类型***

IP数据类型

	ip 用于IPv4和IPv6地址

完成数据类型

	completion 提供自动完成建议

令牌计数数据类型

	token_count 计算字符串中令牌的数量
	mapper-murmur3
	murmur3 在索引时计算值的哈希并将其存储在索引中
	mapper-annotated-text
	annotated-text 索引包含特殊标记的文本（通常用于标识命名实体）

渗滤器类型

	接受来自query-dsl的查询

join 数据类型

	为同一索引内的文档定义父/子关系

别名数据类型

	为现有字段定义别名。

***多字段***
	为不同的目的以不同的方式对同一字段建立索引通常很有用。例如，一个string字段可以映射为text用于全文搜索的字段，也可以映射为keyword用于排序或聚合的字段。或者，您可以使用standard分析仪， english分析仪和 french分析仪索引文本字段。这是多领域的目的。大多数数据类型通过fields参数支持多字段。


##### 3.2 实例

```
PUT test1
{
    "settings" : {
        "number_of_shards" : 10, # 设置的分片数
        "number_of_replicas" : 1, # 索引库的副本数
        "refresh_interval" : "1s" # 缓存的刷新时间
		
		# 其它字段
		# store: true/false 表示该字段是否存储，默认存储。
		# doc_values: true/false 表示该字段是否参与聚合和排序。
		# index: true/false 表示该字段是否建立索引，默认建立
    },
    "mappings" : {
        "_doc" : {
            "properties" : { # 索引属性值
                "uid" : { "type" : "long" },
                "phone" : { "type" : "long" },
                "message" : { "type" : "keyword" },
                "msgcode" : { "type" : "long" },
                 "sendtime" : {  
                  "type" : "date",
                  "format" : "yyyy-MM-dd HH:mm:ss" 
  }
                
            }
        }
    }
}
```

### 4. 参考文章
> [ElasticSearch实战系列二: ElasticSearch的DSL语句使用教程](https://www.cnblogs.com/xuwujing/p/11567053.html)
