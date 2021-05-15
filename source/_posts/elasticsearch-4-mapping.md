---
title:  ES（四）深入搜索——结构化搜索与全文搜索
date: 2020-08-06 22:39:51
tags: [elasticsearch]
---
 

### 1. 结构化搜索

在上一章节中，有一部分关于查询表达式的查询语法，主要针对的就是结构化的数据，进行`结构化搜索`。

结构化搜索（Structured search） 是指有关探询那些具有内在结构数据的过程。比如日期、时间和数字都是结构化的：它们有精确的格式，我们可以对这些格式进行逻辑操作。比较常见的操作包括比较数字或时间的范围，或判定两个值的大小。

> 在结构化查询中，我们得到的结果 总是 非是即否，要么存于集合之中，要么存在集合之外。结构化查询不关心文件的相关度或评分；它简单的对文档包括或排除处理。对于结构化文本来说，一个值要么相等，要么不等。没有 更似 这种概念。



##### 1.1 精确值（包含）查找

当进行精确值查找时 会使用过滤器（filters）。过滤器很重要，因为它们执行速度非常快，不会计算相关度（直接跳过了整个评分阶段）而且很容易被缓存，所以要尽可能的使用过滤器。

>  term 和 terms 是 包含（contains） 操作，而非 等值（equals）判断。
由于倒排索引表自身的特性，整个字段是否相等会难以计算，如果确定某个特定文档是否 只（only） 包含我们想要查找的词呢？首先我们需要在倒排索引中找到相关的记录并获取文档 ID，然后再扫描 倒排索引中的每行记录 ，查看它们是否包含其他的 terms 。
可以想象，这样不仅低效，而且代价高昂。正因如此， term 和 terms 是 必须包含（must contain） 操作，而不是 必须精确相等（must equal exactly） 。



***term 和 terms***


term查询：可以用它处理数字（数字），布尔值（布尔值），日期（dates）以及文本（text）。
> 可以使用 `constant_score` 查询以非评分模式来执行，term 查询并以`一`作为统一评分

添加测试数据：
```
POST /products/_doc
{ "price" : 10, "productID" : "XHDK-A-1293-#fJ3" }
POST /products/_doc
{ "price" : 20, "productID" : "KDKE-B-9947-#kL5" }
POST /products/_doc
{ "price" : 30, "productID" : "JODL-X-1937-#pV7" }
POST /products/_doc
{ "price" : 30, "productID" : "QQPX-R-3956-#aD8" }
```


term 查询数字（constant_score不进行评分）：

```

GET /products/_search
{
	"query":{
		"constant_score":{
			"filter": {
				"term": {
					"price":20
				}
			}
		}
	}
}
```

term查询文本：

```
GET /products/_search
{
	"query":{
		"constant_score":{
			"filter": {
				"term": {
					"productID":"QQPX-R-3956-#aD8"
				}
			}
		}
	}
}
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```


查询不到对应的数据，因为`productID`字段类型是`text`，已经自动进行分词从而方便搜索，所以无法准确匹配。但是，在新版本中， 自动为`fields`配置了字段`keyword`,方便排序。

```
GET /products/_mapping
{
  "products" : {
    "mappings" : {
      "properties" : {
        "price" : {
          "type" : "long"
        },
        "productID" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}

# 通过`X.keyword`查找精确

GET /products/_search
{
	"query":{
		"constant_score":{
			"filter": {
				"term": {
					"productID.keyword":"QQPX-R-3956-#aD8"
				}
			}
		}
	}
}
```



同时查找多个精确值：

```
	GET /products/_search
	{
		"query" : {
			"constant_score" : {
				"filter" : {
					"terms" : { 
						"price" : [20, 30]
					}
				}
			}
		}
	}
```



***精确值相等查找***

如果要求包含且只有一个标签，即百分百吻合，可以通过添加新字段实现：

```
{ "tags" : ["search"], "tag_count" : 1 }
{ "tags" : ["search", "open_source"], "tag_count" : 2 }

GET /products/_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                 "bool" : {
                    "must" : [
                        { "term" : { "price" : 20} } 
                    ]
                }
            }
        }
    }
}
```


##### 1.2 范围查找

有时候，我们需要对数字范围进行过滤。例如，我们可能想要查找所有价格大于 $20 且小于 $40 美元的产品。

```
GET /products/_search
{
	"query" : {
		"constant_score" : {
			"filter" : {
				"range" : { 
					"price" : {
						"gte":20,
						"lte":40
					}
				}
			}
		}
	}
}
```

range 查询可同时提供包含（inclusive）和不包含（exclusive）这两种范围表达式，可供组合的选项如下：

```
gt: > 大于（greater than）
lt: < 小于（less than）
gte: >= 大于或等于（greater than or equal to）
lte: <= 小于或等于（less than or equal to）
```

range 查询同样可以应用在日期字段上：

```
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-07 00:00:00"
    }
}


# 过去一小时
"range" : {
    "timestamp" : {
        "gt" : "now-1h"
    }
}

"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-01 00:00:00||+1M" 
    }
}
```


range 查询同样可以处理字符串字段，字符串范围可采用 字典顺序（lexicographically） 或字母顺序（alphabetically）。在倒排索引中的词项就是采取字典顺序（lexicographically）排列的，这也是字符串范围可以使用这个顺序来确定的原因。语法如下：

```
"range" : {
    "title" : {
        "gte" : "a",
        "lt" :  "b"
    }
}
```


##### 1.3 处理NULL值

如果一个字段没有值，那么倒排索引什么都不存。但可以通过以下几个语句查询：


exists 存在查询。这个查询会返回那些在指定字段有任何值的文档
missing 查询本质上与 exists 恰好相反：它返回某个特定 无 值字段的文档

语法如下：
```
GET /products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "exists" : { "field" : "price" }
            }
        }
    }
}

GET /products/_search
{
    "query" : {
        "constant_score" : {
            "filter": {
                "missing" : { "field" : "price" }
            }
        }
    }
}

# 内部字段
{
   "name" : {
      "first" : "John",
      "last" :  "Smith"
   }
}
{
   "name.first" : "John",
   "name.last"  : "Smith"
}


# 过滤
{
    "exists" : { "field" : "name" }
}
# 实际执行

{
    "bool": {
        "should": [
            { "exists": { "field": "name.first" }},
            { "exists": { "field": "name.last" }}
        ]
    }
}
```

> 有时候我们需要区分一个字段是没有值，还是它已被显式的设置成了 null 。在之前例子中，我们看到的默认的行为是无法做到这点的；数据被丢失了。不过幸运的是，我们可以选择将显式的 null 值替换成我们指定 占位符（placeholder） 。
在为字符串（string）、数字（numeric）、布尔值（Boolean）或日期（date）字段指定映射时，同样可以为之设置 null_value 空值，用以处理显式 null 值的情况。不过即使如此，还是会将一个没有值的字段从倒排索引中排除。
当选择合适的 null_value 空值的时候，需要保证以下几点：
它会匹配字段的类型，我们不能为一个 date 日期字段设置字符串类型的 null_value 。
它必须与普通值不一样，这可以避免把实际值当成 null 空的情况。


##### 1.4 内部过滤器

结构化查询其实是依靠内部过滤器实现的，所以了解一下内部过滤器的基本知识还是很有必要的。

内部过滤器的相关操作步骤：

1. 查询匹配文档

通过倒排索引查找匹配的文档

2. 创建`bitset`

过滤器会创建一个 bitset （一个包含 0 和 1 的数组），它描述了哪个文档会包含该 term 。匹配文档的标志位是 1 。本例中，bitset 的值为 [1,0,0,0] 。在内部，它表示成一个 "roaring bitmap"，可以同时对稀疏或密集的集合进行高效编码。

3. 迭代 bitset

Elasticsearch 会循环迭代 bitsets 从而找到满足所有过滤条件的匹配文档的集合。执行顺序是启发式的，但一般来说先迭代稀疏的 bitset （因为它可以排除掉大量的文档）。

4. 增量使用计数

Elasticsearch 能够缓存非评分查询从而获取更快的访问，但是它也会不太聪明地缓存一些使用极少的东西。
Elasticsearch 会为每个索引跟踪保留查询使用的历史状态。如果查询在最近的 256 次查询中会被用到，那么它就会被缓存到内存中。当 bitset 被缓存后，缓存会在那些低于 10,000 个文档（或少于 3% 的总索引数）的段（segment）中被忽略。



> 从 5.0 版本开始，string 类型被废弃，引入了 keyword 、text 两种类型。
主要区别是keyword（ 32766 字节） 不支持全文搜索，只能是使用精确匹配进行查询，比如 term 查询；text（无限制） 默认支持全文搜索。
理论上非评分查询 先于 评分查询执行。非评分查询任务旨在降低那些将对评分查询计算带来更高成本的文档数量，从而达到快速搜索的目的。



***缓存模块——基础***
索引有不同的内置缓存模块。 它们包括 过滤器（filter）, 字段（field） 和其它。
+	过滤器缓存：负责缓存过滤后的结果
	+	节点过滤器缓存（默认配置，多个分片共享一个缓存）
		+	LRU 数据删除策略:当缓存区满了以后,最近最少使用的数据被删除，以腾出空间给新的数据存放。
	+	索引过滤器缓存（一个分片配置一个缓存）
+	字段数据缓存
	+	字段数据缓存的主要用在当以某一个字段排序或faceting上
	
	
***过滤器缓存***

bitsets 缓存以增量方式更新。当我们索引新文档时，只需将那些新文档加入已有 bitset，而不是对整个缓存一遍又一遍的重复计算。


缓存是独立于搜索请求。这就意味着，一旦被缓存，一个缓存可以被用于多个搜索请求。

```
# 下面例子中的查询，它查找满足以下任意一个条件的电子邮件：
	# 在收件箱中，且没有被读过的
	# 不在 收件箱中，但被标注重要的

# 在以下复合查询中，两个过滤器是相同的，所以会使用同一 bitset 。
GET /inbox/emails/_search
{
  "query": {
      "constant_score": {
          "filter": {
              "bool": {
                 "should": [
                    { "bool": {
                          "must": [
                             { "term": { "folder": "inbox" }}, 
                             { "term": { "read": false }}
                          ]
                    }},
                    { "bool": {
                          "must_not": {
                             "term": { "folder": "inbox" } 
                          },
                          "must": {
                             "term": { "important": true }
                          }
                    }}
                 ]
              }
            }
        }
    }
}
```

***自动缓存***
Elasticsearch 会基于使用频次自动缓存查询。如果一个非评分查询在最近的 256 次查询中被使用过（次数取决于查询类型），那么这个查询就会作为缓存的候选。但是，并不是所有的片段都能保证缓存 bitset 。只有那些文档数量超过 10,000 （或超过总文档数量的 3% )才会缓存 bitset 。因为小的片段可以很快的进行搜索和合并，这里缓存的意义不大。

一旦缓存了，非评分计算的 bitset 会一直驻留在缓存中直到它被剔除。剔除规则是基于 LRU 的：一旦缓存满了，最近最少使用的过滤器会被剔除。

### 2. 全文搜索

全文搜索最重要的是解决一个问题：怎样在全文字段中搜索到最相关的文档。
文搜索两个最重要的方面是：

相关性（Relevance）
+	它是评价查询与其结果间的相关程度，并根据这种相关程度对结果排名的一种能力，这种计算方式可以是 TF/IDF 方法（参见 相关性的介绍）、地理位置邻近、模糊相似，或其他的某些算法。
分析（Analysis）
+	它是将文本块转换为有区别的、规范化的 token 的一个过程，（参见 分析的介绍） 目的是为了（a）创建倒排索引以及（b）查询倒排索引。

[结构化搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/structured-search.html)



##### 2.1 文本查询

和一些特殊的完全不会对文本进行操作的查询（如 bool 或 function_score ）不同，文本查询可以划分成两大家族：

1. 基于词项的查询

+	如 term 或 fuzzy 这样的底层查询不需要分析阶段，它们对单个词项进行操作。用 term 查询词项 Foo 只要在倒排索引中查找 准确词项 ，并且用 TF/IDF 算法为每个包含该词项的文档计算相关度评分 `_score`（无须考虑词项是如何存入索引的）。

2. 基于全文的查询

像 match 或 query_string 这样的查询是高层查询，它们了解字段映射的信息：
+	如果查询 日期（date） 或 整数（integer） 字段，它们会将查询字符串分别作为日期或整数对待。
+	如果查询一个（ not_analyzed ）未分析的精确值字符串字段，它们会将整个查询字符串作为单个词项对待。
+	但如果要查询一个（ analyzed ）已分析的全文字段，它们会先将查询字符串传递到一个合适的分析器，然后生成一个供查询的词项列表。一旦组成了词项列表，这个查询会对每个词项逐一执行底层的查询，再将结果合并，然后为每个文档生成一个最终的相关度评分。

> 当我们想要查询一个具有精确值的 not_analyzed 未分析字段之前，需要考虑，是否真的采用评分查询，或者非评分查询会更好。
单词项查询通常可以用是、非这种二元问题表示，所以更适合用过滤。


##### 2.2 匹配查询


***match***

匹配查询 `match` 是个 核心 查询。无论需要查询什么字段， match 查询都应该会是首选的查询方式。它是一个高级 全文查询 ，这表示它既能处理全文字段，又能处理精确字段。


添加一些测试数据：

```
# 使用 bulk API 创建一些新的文档和索引

DELETE /qw

PUT /qw
{ "settings": { "number_of_shards": 1 }}  

POST /qw/_bulk
{ "index": { "_id": 1 }}
{ "title": "The quick brown fox" }
{ "index": { "_id": 2 }}
{ "title": "The quick brown fox jumps over the lazy dog" }
{ "index": { "_id": 3 }}
{ "title": "The quick brown fox jumps over the quick dog" }
{ "index": { "_id": 4 }}
{ "title": "Brown fox brown dog" }
```

单个词查询：

```
GET /qw/_search
{
    "query": {
        "match": {
            "title": "QUICK!"
        }
    }
}

查询步骤：
+	检查字段类型
+	分析查询字符串
+	查找匹配文档
+	为每个文档评分(_score )
```

> term 查询计算每个文档相关度评分 _score ，这是种将词频（term frequency，即词 quick 在相关文档的 title 字段中出现的频率）和反向文档频率（inverse document frequency，即词 quick 在所有文档的 title 字段中出现的频率），以及字段的长度（即字段越短相关度越高）相结合的计算方式.


***多词查询***

match 查询可以简化多词查询：

```
GET /qw/_search
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
        }
    }
}
```

因为 match 查询必须查找两个词（ ["brown","dog"] ），它在内部实际上先执行两次 term 查询，然后将两次查询的结果合并作为最终结果输出。为了做到这点，它将两个 term 查询包入一个 bool 查询中


我们可以用任意查询词项匹配文档可能会导致结果中出现不相关的长尾,搜索包含所有词项的文档。也就是说，不去匹配 brown OR dog ，而通过匹配 brown AND dog 找到所有文档，从而提高精度。实例如下：

```
# 默认是 or
GET /qw/_search
{
    "query": {
        "match": {
            "title": {      
                "query":    "BROWN DOG!",
                "operator": "and"
            }
        }
    }
}

```

我们也可以更精确的控制匹配度。match 查询支持 `minimum_should_match ` 最小匹配参数，这让我们可以指定必须匹配的词项数用来表示一个文档是否相关。我们可以将其设置为某个具体数字，更常用的做法是将其设置为一个百分数:

```
GET /qw/_search
{
  "query": {
    "match": {
      "title": {
        "query":                "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
```

##### 2.3 布尔匹配


多词 match 查询：
```
{
    "match": { "title": "brown fox"}
}
 
```
可以使用以下语句代替

```

{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

使用 and 操作符：

```
{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
```

可以使用以下语句代替

```


{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
```


minimum_should_match 在bool中的使用：

```
{
    "match": {
        "title": {
            "query":                "quick brown fox",
            "minimum_should_match": "75%"
        }
    }
}
```

可以使用以下语句代替
```

{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }},
      { "term": { "title": "quick" }}
    ],
    "minimum_should_match": 2 
  }
}
```

##### 2.4 提升权重

假设想要查询关于 “full-text search（全文搜索）” 的文档，但我们希望为提及 “Elasticsearch” 或 “Lucene” 的文档给予更高的 权重 ，这里 更高权重 是指如果文档中出现 “Elasticsearch” 或 “Lucene” ，它们会比没有的出现这些词的文档获得更高的相关度评分 `_score` ，也就是说，它们会出现在结果集的更上面。

```
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {
                    "content": { 
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [ 
                { "match": { "content": "Elasticsearch" }},
                { "match": { "content": "Lucene"        }}
            ]
        }
    }
}
```

这样，结果就必须包含“full text search”的词组，且应当包含"Elasticsearch"或者"Lucene"。但是，在实际使用中，这两个词的权重是不同的。我们可以通过调整`boost` 来修改词权重，从而获得更高的相关度。

```
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {  
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 
                    }
                }}
            ]
        }
    }
}
```

+	默认的 boost 值 为1;
+	Elasticsearch有最高的 boost 值 3;
+	Lucene比使用默认值的更重要，但它的重要性不及 Elasticsearch 语句.

> 
boost 参数被用来提升一个语句的相对权重（ boost 值大于 1 ）或降低相对权重（ boost 值处于 0 到 1 之间），但是这种提升或降低并不是线性的，换句话说，如果一个 boost 值为 2 ，并不能获得两倍的评分 _score。

***错误的相关度****

Elasticsearch 默认使用的相似度算法，这个算法叫做 词频/逆向文档频率 或 TF/IDF 。词频是计算某个词在当前被查询文档里某个字段中出现的频率，出现的频率越高，文档越相关。 逆向文档频率 将 某个词在索引内所有文档出现的百分数 考虑在内，出现的频率越高，它的权重就越低。

由于性能原因， Elasticsearch 不会计算索引内所有文档的 IDF 。相反，每个分片会根据 该分片 内的所有文档计算一个本地 IDF 。理想状态下，由于文档是均匀分布存储的，两个分片的 IDF 是相同的。但是在实际应用中，初始阶段数据量很少，数据分布均匀但词分布不均匀。本地和全局的 IDF 的差异随着索引里文档数的增多才会渐渐消失。

由此可以看出，多分片的情况下，数据量过少会导致各分片的`IDF`不同，最终导致某些低相关的结果排在高相关的前面。


> 在测试阶段，可以使用`?search_type=dfs_query_then_fetch`解决这个问题。 dfs 是指 分布式频率搜索（Distributed Frequency Search） ， 它告诉 Elasticsearch ，先分别获得每个分片本地的 IDF ，然后根据结果再计算整个索引的全局 IDF 。




##### 2.5 控制分析

查询只能查找倒排索引表中真实存在的项，所以保证文档在索引时与查询字符串在搜索时应用相同的分析过程非常重要，这样查询的项才能够匹配倒排索引中的项。（文档、查询字符串所用的分析器应当相同）

。每个字段都可以有不同的分析器，既可以通过配置为字段指定分析器，也可以使用更高层的~类型（type）~、索引（index）或节点（node）的默认配置

***默认分析器***

分析器可以从三个层面进行定义：
+	按字段（per-field）
+	按索引（per-index）
+	全局缺省（global default）

Elasticsearch 会按照以下顺序依次处理，直到它找到能够使用的分析器。索引时的顺序如下：	

+	字段映射里定义的 analyzer
+	索引设置中名为 default 的分析器
+	standard 标准分析器


搜索时的分析器判定顺序：

+	查询定义的 analyzer 
+	字段映射里定义的 analyzer
+	索引设置中名为 default 的分析器
+	standard 标准分析器


为了区分（索引与搜索），Elasticsearch 也支持一个可选的 search_analyzer 映射，它仅会应用于搜索时（ analyzer 还用于索引时）。还有一个等价的 default_search 映射，用以指定索引层的默认配置。

一个搜索时的 完整 顺序会是下面这样：

+	查询自己定义的 analyzer
+	字段映射里定义的 search_analyzer
+	字段映射里定义的 analyzer
+	索引设置中名为 default_search 的分析器
+	索引设置中名为 default 的分析器
+	standard 标准分析器
