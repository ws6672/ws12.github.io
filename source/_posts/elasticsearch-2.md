---
title: ES（二）索引（index）管理
date: 2020-07-30 13:15:41
tags: [elasticsearch]
---

在本章中，要深入了解什么是索引以及如何配置一个索引。

在 elasticsearch中，一切的设计就是为了提高搜索效率。ES是建立在全文搜索引擎 Apache Lucene(TM) 基础上的搜索引擎，利用倒排索引对索引的每个字段进行分析，提供了强大的搜索功能。


### 1. 索引结构

索引（index）定义组成：

+	`_settings`
	+	索引配置，包括分片与副片的配置
+	`_aliases`，索引别名。
	+	类似软连接，指向一个或多个索引，可以用于索引分组、索引切换
+	`_mapping`
	+	映射与分析，用于配置字段类型（数据模式）；如果不进行配置，那么会依据数据的格式设置字段的类型。


测试：

```
// 获取配置（megacorp 是索引名称）
GET /megacorp/_settings
// 获取映射
GET /megacorp/_mapping
// 获取别名
GET /megacorp/_alias
```


##### 1.1 倒排索引

Elasticsearch使用一种叫做倒排索引(inverted index)的结构来做快速的全文搜索。倒排索引 由在文档中出现的唯一的单词列表，以及对于每个单词在文档中的位置组成。

例如：存在以下两个索引的content字段：
```
1. The quick brown fox jumped over the lazy dog 
2. Quick brown foxes leap over lazy dogs in summer
```

为了创建倒排索引，我们首先切分每个文档的 `content` 字段为单独的单词（我们把它们叫做 词(terms)或者表征(tokens)）。然后，建立每个词在文档中是否出现为判定标准建立二维表。

***查询流程***

1. 假如查询`quick brown`,将该查询分为最小词组（quick、brown）；
2. 然后按表，查询该词组在所有content出现的数目；
3. 加入相似度算法 (similarity algorithm)，计算匹配单词的数目，，这样我们就可以说第一个文档比第二个匹配度 更高。

***优化***

将词为统一为标准格式，这样就可以找到不是确切匹配查询，但是足以相似从而可以关联的文档。

```
1. "Quick" 可以转为小写成为 "quick" 。 
2. "foxes" 可以被转为根形式 "fox" 。同理 "dogs" 可以被转为 "dog" 。
3. "jumped" 和 "leap" 同义就可以只索引为单个词 "jump"
```

这个标记化和标准化的过程叫做`分词(analysis)`.

> 注：索引文本和查询字符串都要标准化为相同的形式



##### 1.2 创建索引

我们可以通过添加一个文档创建了默认索引 ，新的字段通过动态映射的方式被添加到类型映射。如果需要对这个索引做更多的控制，需要手动创建索引。

手动创建索引的好处：

+	设置数量适中的主分片
+	配置适合的分析器与映射

索引模块是为每个索引创建的模块，用于控制与索引相关的所有方面，我们可以基于以下几个方面来配置索引：
+	`_settings` 	
+	`_alias` 
+	`_mapping`  

##### 1.3 索引的基本语法


```
# 创建索引配置
PUT /new_megacorp 
{
	"settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    },
	"mappings": {
		"properties" : {
			"about" : {
			  "type" : "text"
			},
			"age" : {
			  "type" : "long"
			},
			"first_name" : {
			  "type" : "text"
			},
			"interests" : {
			  "type" : "text"
			},
			"last_name" : {
			  "type" : "text"
			},
			"pm" : {
			  "type" : "integer"
			},
			"tag" : {
			  "type" : "text"
			}
		}
	}
}

 
# 查询索引配置
GET /new_megacorp

# 删除索引
DELETE /new_megacorp

# 删除多个索引
DELETE /index_one,index_two
DELETE /index_*

# 删除全部索引
DELETE /_all
DELETE /*

# 修改索引-语法与修改文档相同


```


### 2. 索引设置（_settings）

可以通过修改配置来自定义索引行为，详细配置参照 [索引模块](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/index-modules.html)

##### 2.1 索引级别

每个索引设置都有一个相对的`索引级别` 。索引级别如下：
+	`静态索引（static）`：只能在索引创建时配置或者在索引关闭时配置
+	`动态索引（dynamic）`：在实时索引上对其进行更改

***静态索引设置***

`index.number_of_shards `：索引应具有的主要分片数。默认为1。此设置只能在创建索引时设置。不能在封闭索引上更改它。
> 分片的数量限于1024每个索引。此限制是一项安全限制，可防止意外创建因资源分配而使集群不稳定的索引

index.shard.check_on_startup：打开前是否应检查碎片是否损坏。
+	false:（默认）打开分片时不检查是否损坏。
+	checksum :检查物理损坏。
+	true:检查物理和逻辑损坏。就CPU和内存使用而言，这要昂贵得多。

index.codec：该default值使用LZ4压缩来压缩存储的数据，但是可以将其设置为best_compression 使用DEFLATE以获得更高的压缩率，但是会降低存储字段的性能。

index.routing_partition_size：自定义路由值可以到达的分片数量。默认为1

index.load_fixed_bitset_filters_eagerly：指示是否为嵌套查询预加载缓存的过滤器。可能的值为true（默认）和false。

index.hidden：指示默认情况下是否应隐藏索引。使用通配符表达式时，默认情况下不返回隐藏索引。

***动态索引设置***

以下是与任何特定索引模块都不相关的所有动态索引设置的列表：

`index.number_of_replicas`：每个主分片具有的副本数。默认为1。

`index.refresh_interval`：执行刷新操作的频率，这使对索引的最近更改可见以进行搜索。默认为1s。可以设置-1为禁用刷新

index.auto_expand_replicas：根据集群中数据节点的数量自动扩展副本的数量。设置为以短划线分隔的上下限（例如0-5）或all 用于上限（例如0-all）。默认为false（即禁用）。

index.search.idle.after：分片在被视为搜索空闲之前不能接收搜索或获取请求的时间。（默认为30s）

[更多动态设置](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/index-modules.html)
 

> 注：
更改封闭索引上的静态或动态索引设置可能会导致不正确的设置，如果不删除并重新创建索引，则无法纠正这些设置。


##### 2.2 实例

下面是两个最重要的设置：

> number_of_shards：每个索引的主分片数，交替值是5。这个配置在索引创建后不能修改。
number_of_replicas：每个主分片的副本数，默认值是1，对于活动的索引库，这个配置可以随时修改。

```
PUT /my_test_index
{
	"settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    }
}

# 结果：
	{
	  "acknowledged" : true,
	  "shards_acknowledged" : true,
	  "index" : "my_test_index"
	}

GET /my_test_index

# 结果：
	{
	  "my_test_index" : {
		"aliases" : { },
		"mappings" : { },
		"settings" : {
		  "index" : {
			"creation_date" : "1596032939706",
			"number_of_shards" : "1",
			"number_of_replicas" : "0",
			"uuid" : "5331PSOGS--Jvf-KW10ZeA",
			"version" : {
			  "created" : "7080099"
			},
			"provided_name" : "my_test_index"
		  }
		}
	  }
	}

```

### 3. 索引别名（_alias）

索引别名API允许使用一个名字来作为一个索引的别名，所有API会自动将别名转换为实际的索引名称。 别名也可以映射到多个索引，别名不能与索引具有相同的名称。别名可以用于：
+	索引迁移
+	多个索引的查询统一
+	实现视图功能  


##### 3.1 基本语法

索引中关于别名的设置，一般配置在`_alias`字段中，相关语法如下：

```
# 查看所有别名
GET /_alias

# 别名与多个索引相关联
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test", "alias" : "alias1" } },
        { "add" : { "index" : "my_test_index", "alias" : "alias1" } }
    ]
}
{
  "acknowledged" : true
}

# 查看某个别名下的索引：GET /_alias/alias_name
GET /_alias/alias1



# 查看某个索引的别名：GET /index_name/_alias
GET /test/_alias

# 添加并删除别名
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test", "alias" : "alias1" } },
        { "add" : { "index" : "test", "alias" : "alias2" } }
    ]
}

# 使用数组的形式
POST /_aliases
{
    "actions" : [
        { "add" : { "indices" : ["test", "my_test_index"], "alias" : "alias3" } }
    ]
}

# 使用通配符
GET /_alias/ali*

# 创建视图
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "megacorp",
                 "alias" : "alias2",
                 "filter" : { "term" : { "last_name" : "李" } }
            }
        }
    ]
}
# 通过视图获取数据
GET /alias2/_doc/_search

# 添加单个别名
PUT /{index}/_alias/{name}

# 删除别名
DELETE /{index}/_alias/{name}

```

### 4. 映射机制（_mapping）

ES处理数据最重要功能之一是映射与分析。
+	映射是`ES`对数据的认知，通过它可以知道字段是什么类型的数据以及如何去解析字段。

##### 4.1 映射机制

映射(mapping)机制用于进行字段类型确认，将每个字段匹配为一种确定的数据类型 ( string , number , booleans , date 等)。
如果不在存储数据之前设置好索引的类型，那么该数据库会依据数据类型进行mapping（模式定义）。Elasticsearch为对字段类型进行猜测，动态生成了字段和类型的映射关系。

```
# 测试用例
curl -XPOST http://localhost:9200/test/_create/ -H 'Content-Type:application/json' -d '{"date":2014-09-15}'

GET /test/_search?q=2014 # 12 个结果
GET /test/_search?q=2014-09-15 # 还是 12 个结果 ! 
GET /test/_search?q=date:2014-09-15 # 1 一个结果 
GET /test/_search?q=date:2014 # 0 个结果 !

# 在上面的例子中，date的类型会被自动识别为 `date`类型.`date` 类型的字段和 `string` 类型的字段的索引方式是不同的，因此导致查询结果的不同
```

##### 4.2 映射(mapping)定义


***定义***

索引定义一般是设置`_mapping`字段，`Mapping` 类似数据库中的 `schema` 的定义，作用如下：
+	定义字段名称
+	定义字段类型
+	倒排索引配置（分片、副本等）
+	Mapping 会把 Json 文档映射成 Lucene 所需要的扁平格式


当你索引一个包含新字段的文档——一个之前没有的字段——`Dynamic Mapping` 会自动根据文档信息，推算出字段的类型，使得你无需手动创建 Mapping。在写入文档时候，而且如果索引不存在，会自动创建索引。类型映射如下：

|Json 类型|Elasticsearch 类型|
|:--|:--|
|字符串|1.匹配日期格式，转为 Date<br/>2.数值转为 float 或者 long,默认关闭<br/>3.转为 Text，并且增加 keyword|
|布尔值|boolean|
|浮点数|float|
|整 数|long|
|对 象|Object|
|数 组|由第一个非空数值的类型所决定|
|空 值|忽略|

> 如果你索引一个带引号的数字—— "123" ，它将被映射为 "text" 类型，而 不是 "long" 类型。然而，如果字段已经被映射为 "long" 类型，Elasticsearch将尝试转 换字符串为long，并在转换失败时会抛出异常。

***使用***


```
# 关闭索引
POST /megacorp/_close    

# 开启索引
POST /megacorp/_open

# 查看索引映射
	GET /megacorp/_mapping/
	
	{
	  "megacorp" : { # 索引
		"mappings" : {
		  "properties" : { # 属性
			"about" : {
			  "type" : "text", # 
			  "fields" : {
				"keyword" : {
				  "type" : "keyword",
				  "ignore_above" : 256
				}
			  }
			},
			......
		  }
		}
	  }
	}

# 添加字段
PUT /megacorp/_mapping
{
 "properties":{
    "pm":{
      "type":"integer"
    }
 }
}

```


***自定义映射***

我们经常需要自定义一些特殊类型 （fields），特别是字符串字段类型。

自定义类型的优势如下：

+	区分`全文（full text）字符串字段`和`准确字段`（区分 分词与不分词，全文的一般要分词，准确的就不需要分词）
+	可以使用特定语言的分析器（译者注：例如中文、英文、阿拉伯语，不同文字的断字、断词 方式的差异）
+	优化部分匹配字段
+	指定自定义日期格式

> ElasticSearch7.x 默认不再支持指定索引类型，默认索引类型是_doc

##### 4.3 索引模板

索引可使用预定义的模板进行创建,这个模板称作Index templates。模板设置包括settings和mappings，通过模式匹配的方式使得多个索引重用一个模板。在创建新索引时, 指定要使用的模板名, 就可以直接重用已经定义好的模板中的设置和映射.

`使用PUT方法创建索引模板`。索引模板用于定义在创建新的索引时自动应用的模板，可以创建普通索引模板，也可以创建别名索引模板等，索引模板中的信息主要包括以下部份：

+	可套用该索引模板的索引名称格式，名称支持通配符，也可以配置多个名称格式匹配格式
+	索引的基本设置（settings）
+	索引的字段映射（mapping）信息
+	别名（alias）

需要配置的数据：
+	settings: 指定index的配置信息, 比如分片数、副本数, tranlog同步条件、refresh策略等信息;
+	mappings: 指定index的内部构建信息

```
# 普通索引模板
PUT /_template/mytp
{
  //索引的名称支持通配,会匹配'log_'前缀的索引
  "index_patterns": ["log_*"],
  "order": 0, // 模板的权重,值越大, 权重越高
  "settings": {
	//设置的分片数
	"number_of_shards": 2,
	// 索引库的副本数
    "number_of_replicas": 1,
	// 缓存的刷新时间
    "refresh_interval": "1s"
  },
  "mappings": {
    "_source": {
	  // 该字段决定是否参与相关度排名计算
		"enabled": true
	},
    "properties": {
      "created":{
        "type": "date"
      }
    }
  }
}

# ​​​​​​​别名索引模板
# 别名索引模板通常用于给索引定义一些索引别名、过滤器别名或路由别名等，自动创建一些索引“视图”，方便后续使用，操作如下：

PUT _template/logs_and_others_alias_template
{

	"index_patterns": ["logs_*", "others_*"],
	"settings": {
		"number_of_shards": 1,
		"number_of_replicas": 0
	},

	"aliases": {
	/*定义别名*/
	"{index}-alias": {},
	"alias_tomcat_filter": {
		"filter": { "term": { "user_name": "cen" } },
		"routing": "cen"
	}
	}
}

# 查看模板
GET /_template/log*

# ​​​​​​​修改模板
# 与别名修改流程一致


# 删除模板
# DELETE /_template/log_name
DELETE /_template/logs_and_others_alias_template
```
> 注：该别名模板中使用了{index}占位符变量，创建索引时，{index}会被替换为真实的索引名称，以保证别名索引唯一性

***索引模板一些优化配置***

mapping 可以分为：

+	动态映射（dynamic：true）：动态添加新的字段（或缺省）。
+	静态映射（dynamic：false）：忽略新的字段。在原有的映射基础上，当有新的字段时，不会主动的添加新的映射关系，只作为查询结果出现在查询中。
+	`严格模式（dynamic： strict）`：如果遇到新的字段，就抛出异常。不允许动态输入，避免脏读.


其它优化如下：

+	设置`_source = false`。不输出文档内容只输出评分情况，再通过ID到原始数据库获取数据；节省磁盘空间并减少磁盘IO。
+	用`keyword类型`。test_field: {"type": "keyword"}

> _all: All Field字段, 如果开启, _all字段就会把所有字段的内容都包含进来,检索的时候可以不用指定字段查询 —— 会检索多个字段, 设置方式: "_all": {"enabled": true};
在ES 6.0开始, _all字段被禁用了, 作为替换, 可以通过copy_to自定义实现all字段的功能.
ES 2.x,日期检测可以通过在根对象上设置 date_detection 为 false 来关闭

优化实例：

```
PUT /_template/search_template
{

	"index_patterns": ["search_*"],
	"settings": {
		"number_of_shards": 1,
		"number_of_replicas": 1
	},
	"aliases": {
		 // 定义别名
		"{index}-alias-search": {}
	},
	"mappings": {
		"_source": {
			"enabled": false
		},
		"dynamic":"strict"
	}
}
```

***多模板匹配冲突处理***

如果创建的索引，同时匹配了多个索引模板，则会同时使用多个索引模板中的定义。由于同时匹配了多个索引模板，可能存在冲突。例如，多个索引模板中定义的副本数不同。如果存在类似这样的定义冲突，那么在建立索引的时候，就不容易确认确认索引建立出来的最终定义。

解决方式如下：
+	在索引模板中指定`order`属性，其值为数字，该值越大表示该模板中的定义被使用的可能性越高
+	通过创建索引的时候，在创建语句中指定需要的配置或字段映射，此优先级是最高的



### 5. 分析机制

分析(analysis)机制用于进行全文文本(Full Text)的分词，以建立供搜索用的反向索引。

分析(analysis)是这样一个过程： 
+	首先，标记化一个文本块为适用于倒排索引单独的词(term) 
+	然后标准化这些词为标准形式，提高它们的“可搜索性”或“查全率”

*分析工作是分析器(analyzer)完成的。*

##### 5.1 分析器

分析器的组成如下：
+	字符过滤器 
	+	首先字符串经过字符过滤器(character filter)，它们的工作是在标记化前处理字符串。字符过滤器能够`去除HTML标记`，或者转换 "&" 为 "and" 。
+	分词器
	+	下一步，分词器(tokenizer)被标记化成独立的词。一个简单的分词器(tokenizer)可以根据空 格或逗号将单词分开
+	标记过滤
	+	最后，每个词都通过所有标记过滤(token filters)，它可以修改词（例如将 "Quick" 转为小 写），去掉词（例如停用词像 "a" 、 "and" 、 "the" 等等），或者增加词（例如同义词 像 "jump" 和 "leap" ）

Elasticsearch提供很多开箱即用的字符过滤器，分词器和标记过滤器。这些可以组合来创建 自定义的分析器以应对不同的需求。


##### 5.2 内建的分析器

们常用的还是它的内建分析器。

测试用例：
`"Set the shape to semi-transparent by calling set_trans(5)"`

***标准分析器***

标准分析器是Elasticsearch默认使用的分析器。对于文本分析，它对于任何语言都是最佳选 择（译者注：就是没啥特殊需求，对于任何一个国家的语言，这个分析器就够用了）。它根 据Unicode Consortium的定义的单词边界(word boundaries)来切分文本，然后去掉大部分标 点符号。最后，把所有词转为小写。解析结果如下：

	set, the, shape, to, semi, transparent, by, calling, set_trans, 5

***简单分析器***

简单分析器将非单个字母的文本切分，然后把每个词转为小写。解析结果如下：

	set, the, shape, to, semi, transparent, by, calling, set, trans

***空格分析器***

空格分析器依据空格切分文本。它不转换小写。解析结果如下：

	Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

***语言分析器***

特定语言分析器适用于很多语言。它们能够考虑到特定语言的特性。例如， english 分析器 自带一套英语停用词库——像 and 或 the 这些与语义无关的通用词。这些词被移除后，因为 语法规则的存在，英语单词的主体含义依旧能被理解。解析结果如下：

	set, shape, semi, transpar, call, set_tran, 5

当我们索引(index)一个文档，全文字段会被分析为单独的词来创建倒排索引。不过，当我们 在全文字段搜索(search)时，我们要让查询字符串经过同样的分析流程处理，以确保这些词在 索引中存在。


***测试分析器***

可以使用 analyze API来查看文本是如何被分析的。在查询字符串参 数中指定要使用的分析器，被分析的文本做为请求体：
```
GET /_analyze?analyzer=standard&text=Text to analyze
```

***指定分析器***

当Elasticsearch在你的文档中探测到一个新的字符串字段，它将自动设置它为全文 string 字 段并用 standard 分析器分析。但是如果想使用一个更适合这个数据的语言分析器，必须通过映射(mapping)人工设置这些字段。

### 6. 索引迁移

索引迁移相当于关系型数据库中的数据库迁移。迁移的目的通常是为了修改索引结构；而重建索引可能因为数据类型问题导致
用户数据部分迁移失败。

###### 6.4 reindex接口

ES提供了reindex接口用于数据的迁移，它只是将数据从一个索引复制到另外一个索引。使用`_reindex`的要求如下：
+	源索引中的文档都启用_source
+	新索引不能通过reindex接口定义，所以在使用该接口前需要定义好新索引的映射、碎片等配置。

***实例如下***

```
PUT /new_megacorp 
{
	"settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    },
	"mappings": {
		"properties" : {
			"about" : {
			  "type" : "text"
			},
			"age" : {
			  "type" : "long"
			},
			"first_name" : {
			  "type" : "text"
			},
			"interests" : {
			  "type" : "text"
			},
			"last_name" : {
			  "type" : "text"
			},
			"pm" : {
			  "type" : "integer"
			},
			"tag" : {
			  "type" : "text"
			}
		}
	}
}

# 重建索引
POST /_reindex
{
  "source": {
    "index": "megacorp"
  },
  "dest": {
    "index": "new_megacorp"
  }
}

# 可以看到数据被迁移到新索引中
GET /new_megacorp/_search

```
 