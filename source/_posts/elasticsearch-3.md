---
title: ES（三）搜索与排序
date: 2020-08-05 23:00:01
tags: [elasticsearch]
---

搜索就是Elasticsearch的核心服务之一，而排序是为了更好的搜索。

在上一章中，我们知道了elasticsearch的作用以及一些基本的语法。Elasticsearch通过保存JSON文档，然后可以根据ID检索它们。`Elasticsearch不只会存储(store)文档，也会索引(indexes)文档内容来使之可以被搜索。` 


### 1. 基础

***基本概念***
1. 映射(Mapping)：每个字段中的数据类型解析方式
2. 分析(Analysis)：全文（text）是如何进行处理的
3. 领域特定语言查询(Query DSL)：Elasticsearch使用的灵活的、强大的查询语言
 


### 2. 搜索

Elasticsearch真正强大之处在于可以从混乱的数据中找出有意义的信息，具备强大的全文检索功能。


##### 2.1 ES为搜索提供的服务

+	对于确定字段（例如 integer类型数据），可以进行结构化查询
+	对于可比较字段（例如 date类型），可以进行排序
+	对于全文本字段，可以全文检索，匹配关键字后按相关性排序

##### 2.2 基本搜索


***2.2.0 搜索结果相关的参数***

+	hits字段 
	+	响应中最重要的部分是 hits ，它包含了 total 字段来表示匹配到的文档总数， hits 数组还包含了匹配到的前10条数据。`hits`中还包含一个`hits 数组`
+	`_score字段`
	+	每个节点都有一个 `_score字段`，这是相关性得分(relevance score)，它衡量了文档与查询的 匹配程度。默认的，返回的结果中关联性最大的文档排在首位；这意味着，它是按 照 _score 降序排列的。这种情况下，我们没有指定任何查询，所以所有文档的相关性是一样 的，因此所有结果的 _score 都是取得一个`中间值 1 `。max_score 指的是所有文档匹配查询中 _score 的最大值。
+	took
	+	took 告诉我们整个搜索请求花费的毫秒数。
+	shards
	+	`_shards` 节点告诉我们参与查询的分片数（ total 字段），有多少是成功的（ successful 字 段），有多少的是失败的（ failed 字段）。通常我们不希望分片失败，不过这个有可能发 生。如果我们遭受一些重大的故障导致主分片和复制分片都故障，那这个分片的数据将无法 响应给搜索请求。这种情况下，Elasticsearch将报告分片 failed ，但仍将继续返回剩余分片 上的结果
+	timeout
	+	time_out 值告诉我们查询超时与否。一般的，搜索请求不会超时。如果响应速度比完整的结 果更重要，你可以定义 timeout 参数为 10 或者 10ms （10毫秒），或者 1s （1秒）：`GET /_search?timeout=10ms`

> 注：需要注意的是 timeout 不会停止执行查询，它仅仅告诉你目前顺利返回结果的节点然后 关闭连接。在后台，其他分片可能依旧执行查询，尽管结果已经被发送。

***2.2.1 空搜索***

最基本的搜索API表单是空搜索(empty search)，它没有指定任何的查询条件，只返回集群索引中的所有文档：
`GET /_search`

 
空搜索实例如下：

```
GET /megacorp/_search

{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
	
    "hits" : [
      {
        "_index" : "megacorp",
        "_type" : "_doc",
        "_id" : "-T_tYXMBgrJR-nh9FGCf",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "三",
          "last_name" : "张",
          "age" : 32,
          "about" : "我喜欢收集唱片",
          "interests" : [
            "音乐"
          ],
          "text" : "张三喜欢收集唱片"
        }
      },
      ......
    ]
  }
}

```

***2.2.2 多索引和多类别***

通过限制搜索的不同索引或类型，我们可以在集群中跨所有文档搜索。Elasticsearch转发搜 索请求到集群中平行的主分片或每个分片的复制分片上，收集结果后选择顶部十个返回给我 们。

通常，当然，你可能想搜索一个或几个自定的索引或类型，我们能通过定义URL中的索引或 类型达到这个目的：

```
/_search 在所有索引的所有类型中搜索 
/gb/_search 在索引 gb 的所有类型中搜索
/gb,us/_search 在索引 gb 和 us 的所有类型中搜索 
/g*,u*/_search 在以 g 或 u 开头的索引的所有类型中搜索 
/_all/_search 在所有索引的 user 和 tweet 中搜索 search types user and tweet in all indices
```

> 注：搜索一个索引有5个主分片和5个索引各有一个分片事实上是一样的。

***2.2.3 分页***

和SQL使用 LIMIT 关键字返回只有一页的结果一样，Elasticsearch接受 from 和 size 参数：
+	size : 结果数，默认 10 
+	from : 跳过开始的结果数，默认 0

```
GET /megacorp/_search
{
  "size": 2,
  "from": 0
}
```

***2.2.4 在集群系统中深度分页***

> 假设在一个有5个主分片的索引中搜索。当 我们请求结果的第一页（结果1到10）时，每个分片产生自己最顶端10个结果然后返回它 们给请求节点(requesting node)，它再排序这所有的50个结果以选出顶端的10个结果。 现在假设我们请求第1000页——结果10001到10010。工作方式都相同，不同的是每个分 片都必须产生顶端的10010个结果。然后请求节点排序这50050个结果并丢弃50040个！ 你可以看到在分布式系统中，排序结果的花费随着分页的深入而成倍增长。这也是为什 么网络搜索引擎中任何语句不能返回多于1000个结果的原因。


 

### 3. 请求体查询

某些特定语言（特别是 JavaScript）的 HTTP 库是不允许 GET 请求带有请求体的。对于一个查询请求，Elasticsearch 的工程师偏向于使用 `GET` 方式，因为他们觉得它比 `POST` 能更好的描述信息检索（retrieving information）的行为。然而，因为带请求体的 GET 请求并不被广泛支持，所以 `search API`同时支持 `POST` 请求:

```
	POST /_search
	{
	  "from": 30,
	  "size": 10
	}
```

***实例***

```
	# 空查询
	GET /_search

	# 带匹配的查询
	GET /index_2014*/_search

	# 分页查询
	GET /_search
	{
	  "from": 30,
	  "size": 10
	}
```


##### 3.1 查询表达式（结构化搜索）

查询表达式(Query DSL)是一种非常灵活又富有表现力的 查询语言。 Elasticsearch 使用它可以以简单的 JSON 接口来展现 Lucene 功能的绝大部分。

语法如下：

```
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}


# 指定字段
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}

# 例如
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}

# QUERY_NAME	query
# FIELD_NAME	match
# ARGUMENT	tweet
# VALUE	elasticsearch
```


Elasticsearch提供了基于JSON的完整查询DSL（特定于域的语言）来定义查询。将查询DSL视为查询的AST（抽象语法树），它由两种子句组成：
+	叶子查询子句：`叶查询子句` 中寻找一个特定的值在某一特定领域，如 match，term或 range查询。这些查询可以自己使用。
+	复合查询子句：`复合查询子句` 包装其他叶查询或复合查询，并用于以逻辑方式组合多个查询（例如 bool或dis_max查询），或更改其行为（例如 constant_score查询）。



> 一般情况下，一条经过缓存的过滤查询要远胜一条查询语句的执行效率。
过滤语句的目的就是缩小匹配的文档结果集，所以需要仔细检查过滤条件。原则上来说，使用查询语句做全文本搜索或其他需要进行相关性评分的时候，剩下的全部用过滤语句。
查询子句的行为会有所不同，具体取决于它们是在 查询上下文中还是在过滤器上下文中使用。



##### 3.2 过滤语句

+	term 过滤
+	terms 过滤
+	range过滤
+	exists 和 missing 过滤
+	bool 过滤

***term过滤***

term主要用于精确匹配哪些值，比如数字，日期，布尔值或 not_analyzed的字符串(未经分析的文本数据类型)：

```
# kibana控制台
GET /megacorp/_search
{
  "query": {
    "term": {
      "age": {
     "value": "32"
      }
    }
  }
}
# linux控制台
curl -XGET "http://localhost:9200/megacorp/_search" -H 'Content-Type: application/json' -d'{  "query": {    "term": {      "age": {     "value": "32"      }    }  }}'

# 其它数据格式
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```

***terms 过滤***

terms 跟 term 有点类似，但 terms 允许指定多个匹配条件。 如果某个字段指定了多个值，那么文档需要一起去做匹配：

```
GET /megacorp/_search
{
  "query": {
    "terms": {
      "age":["32","31"]
    }
  }
}
```

***range过滤***

允许按照指定范围查找一批数据：

```
GET /megacorp/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 40
      }
    }
  }
}
```

***exists 和 missing 过滤***

xists 和 missing 过滤可以用于查找文档中是否包含指定字段或没有某个字段，类似于SQL语句中的IS_NULL条件

```
GET /megacorp/_search
{
  "query": {
    "exists": {
      "field":  "age"
    }
  }
}
```

***bool 过滤***

bool 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，bool 过滤器由三部分组成：
```
	{
	   "bool" : {
		  "must" :     [],
		  "should" :   [],
		  "must_not" : [],
	   }
	}
must
	所有的语句都 必须（must） 匹配，与 AND 等价。
must_not
	所有的语句都 不能（must not） 匹配，与 NOT 等价。
should
	至少有一个语句要匹配，与 OR 等价。
```

在实际应用中，我们很有可能会过滤多个值或字段，可以通过嵌套bool过滤器实现：

```
GET /megacorp/_search
{
   "query" : {
		"constant_score" : {
			 "filter" : {
				"bool" : {
				  "should" : [
					{ "term" : {"age" : 32}}, 
					{ "bool" : { 
					  "must" : [
						{ "term" : {"first_name" : "三"}}, 
						{ "term" : {"last_name" : "张"}} 
					  ]
					}}
				  ]
			   }
			}
		}
  }
}
```

 
##### 3.3 查询语句



***bool 查询***

bool 查询与 bool 过滤相似，用于合并多个查询子句。不同的是，bool 过滤可以直接给出是否匹配成功， 而bool 查询要计算每一个查询子句的 _score

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

***bool嵌套查询***

```
{
  "bool" : {
    "should" : [
      { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, 
      { "bool" : { 
        "must" : [
          { "term" : {"productID" : "JODL-X-1937-#pV7"}}, 
          { "term" : {"price" : 30}} 
        ]
      }}
    ]
  }
}
```

***match_all 查询***

使用match_all 可以查询到所有文档，是没有查询条件下的默认语句：

```
{
    "match_all": {}
}
```

***match 查询***

match查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。如果你使用 match 查询一个全文本字段，它会在真正查询之前用分析器先分析match一下查询字符

```
GET /megacorp/_search
{
  "query": {
    "match": {
      "about": "喜欢"
    }
  }
}
```

***multi_match 查询***

multi_match查询允许你做match查询的基础上同时搜索多个字段

```
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```

***match_phrase***

短语查询,词组中的排序意味着几个词的位置是连续且有顺序

```
{
    "match_phrase": {
        "title":    "full text search",
    }
}

#设置slop词组间隔:
{
    "match_phrase": {
        "title": {
            "query": "full text search",
            "slop":1
        }
    }
}
```

***phrase_prefix 查询***

与词组中最后一个词条进行前缀匹配。

```
GET /megacorp/_search
{
  "query": {
    "match_phrase_prefix": {
      "about": "我"
    }
  }
}
```

***regexp查询***

通配符查询

```
{
    "query": {
        "regexp": {
            "title": "W[0-9].+" 
        }
    }
}
```

***返回指定字段***

```
{
    "_source":["id","name"],
    "query":{...}
}
```

### 4. 排序与相关性

对于`text`字段的数据，搜索结果会按照相关性排列，默认降序。对于某些确定字段，可以按照值进行排序。


##### 4.1  相关性

每个文档都有相关性评分，用一个正浮点数字段 _score 来表示 。 _score 的评分越高，相关性越高。

查询语句会为每个文档生成一个 _score 字段。评分的计算方式取决于查询类型 不同的查询语句用于不同的目的： fuzzy 查询会计算与关键词的拼写相似程度，terms 查询会计算 找到的内容与关键词组成部分匹配的百分比，但是通常我们说的 relevance 是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。


***相似度算法***

Elasticsearch 的相似度算法被定义为检索词频率/反向文档频率， TF/IDF ，包括以下内容：

+	检索词频率（TF）——检索词在当前文档的出现频率。频率越高，相关性也越高
+	反向文档频率（IDF）——每个检索词在所有文档中的出现频率。频率越高，相关性越低
+	字段长度准则（fieldNorm）——内容越长，相关性越低

> 如果多条查询子句被合并为一条复合查询语句，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。


了解相关性的判断：
```
# 通过explain解析参数
GET /_search?explain 
{
   "query"   : { "match" : { "tweet" : "honeymoon" }}
}

GET /us/tweet/12/_explain
{
   "query" : {
      "bool" : {
         "filter" : { "term" :  { "user_id" : 2           }},
         "must" :  { "match" : { "tweet" :   "honeymoon" }}
      }
   }
}
```

***_score***
默认情况下，返回的结果是按照`相关性`进行排序的——最相关的文档排在最前。在 Elasticsearch 中， 相关性得分 由一个浮点数进行表示，并在搜索结果中通过 `_score` 参数返回， 默认排序是 `_score` 降序。 

例如
```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```

对于确定字段来说，因为该字段的值是确定的，这个参数是没有意义的。


##### 4.2  按值排序

对于某些确定字段，可以使用`sort`进行值排序。可以排序的类型：数字类型、日期等。当使用该方法进行排序，那么就不会对结果进行相关性排序， `_score`和 `max_score`为 null。


相关实例如下：
```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "user_id" : 1 }}
        }
    },
    "sort": { "date": { "order": "desc" }}
}

# 多级排序
# 当一级排序中有相同结果，那么就按照第二个字段进行排序。

GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }}
    ]
}

# 多值字段排序
# mode的值可以为：min、max 、 avg 或是 sum 
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
	"sort": {
		"dates": {
			"order": "asc",
			"mode":  "min"  
		}
	}
}

```

##### 4.3 字符串排序疑难

> 如果被解析的字符串字段也是多值字段， 很少会按照你想要的方式进行排序。如果你想分析一个字符串，如 fine old art ， 这包含 3 项。我们很可能想要按第一项的字母排序，然后按第二项的字母排序，诸如此类，但是 Elasticsearch 在排序过程中没有这样的信息。

为了以字符串字段进行排序，这个字段应仅包含一项： 整个 not_analyzed 字符串。 但是我们仍需要 analyzed 字段，这样才能以全文进行查询.

```
# tweet 字段分词，用于查询
"tweet": {
    "type":     "string",
    "analyzer": "english"
}

# tweet 包含字段raw，用于排序
"tweet": { 
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": { 
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}

GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    },
    "sort": "tweet.raw"
}
```

***Doc Values***
当你对一个字段进行排序时，Elasticsearch 需要访问每个匹配到的文档得到相关的值。倒排索引的检索性能是非常快的，但是在字段值排序时却不是理想的结构。

+	在搜索的时候，我们能通过搜索关键词快速得到结果集。
+	当排序的时候，我们需要倒排索引里面某个字段值的集合。换句话说，我们需要 转置 倒排索引。

转置 结构在其他系统中经常被称作 列存储 。实质上，它将所有单字段的值存储在单数据列中，这使得对其进行操作是十分高效的，例如排序。

Elasticsearch 中的 Doc Values 常被应用到以下场景：

+	对一个字段进行排序
+	对一个字段进行聚合
+	某些过滤，比如地理位置过滤
+	某些与字段相关的脚本计算


字段包含 doc_values 属性。它有两个值， true、false。默认为 true ，即开启。
+	当 doc_values 为 fasle 时，无法基于该字段排序、聚合、在脚本中访问字段值。
+	当 doc_values 为 true 时，ES 会增加一个相应的正排索引，这增加的磁盘占用，也会导致索引数据速度慢一些

[请求体查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_empty_search.html)