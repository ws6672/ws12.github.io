---
title: ES（五）深入搜索——匹配
date: 2020-08-07 12:53:06
tags: [elasticsearch]
---

### 1. 多字段（multifield）查询

通常我们需要用多个字符串查询多个字段，也就是说，需要对多个查询语句以及它们相关度评分进行合理的合并。


##### 1.1 多字段查询

用 bool 查询 进行多字段查询：

```
# 多字段查询
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}

# 设置语句优先级，通过设置boost可以设置优先级，值越大比重越大
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { 
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { 
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { 
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}


```

***单字符串查询***

best fields策略（dis_max）：文档在同字段中包含的词越多越好，某一个field中匹配到了尽可能多的关键词，就会被排在前面。
多数字段：其他字段是作为匹配每个文档时提高相关度评分的 信号 ， 匹配字段越多 则越好。
混合字段：单个字段都只能作为整体的一部分（例如姓名）；在这种情况下，我们希望在 任何 这些列出的字段中找到尽可能多的词

***多字段匹配（multi_match）***


multi_match 查询为能在多个字段上反复执行相同查询提供了一种便捷方式。

multi_match 多匹配查询的类型有多种，其中的三种恰巧与 了解我们的数据 中介绍的三个场景对应，即： best_fields 、 most_fields 和 cross_fields （最佳字段、多数字段、跨字段）。


##### 1.2 最佳匹配（best_fields）

best_fields策略（以质取胜）：一个字段中包含的词越多分数越高，而不在乎有多少个字段包含词

默认情况下，查询的类型是 best_fields ，这表示它会为每个字段生成一个 match 查询，然后将它们组合到 dis_max 查询的内部，如下：

```
#  多字段

{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}

# 转换为 multi_match

{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", 
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" 
    }
}

# 模糊匹配
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}

# 提升字段权重
# 可以使用 ^ 字符语法为单个字段提升权重，在字段名称的末尾添加 ^boost
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] 
    }
}
```

> 在 multi_match 查询中避免使用 not_analyzed 字段。


##### 1.3 多数匹配（most_fields） 

most_fields 的策略（以量取胜）：包含词的字段越多分数越高，而不在乎字段包含多少次

```
DELETE /my_index
PUT /my_index
{
    "settings": { "number_of_shards": 1 }, 
    "mappings": {
		"properties": {
			"title": { 
				"type":     "text",
				"analyzer": "english",
				"fields": {
					"keyword":   {
						"type":     "text",
						"analyzer": "standard",
						"ignore_above" : 256
					}
				}
			}
		}
    }
}

PUT /my_index
{"title":"My rabbit jumps"}
PUT /my_index
{"title":"Jumping jack rabbit"}

# 普通查询
GET /my_index/_search
{
   "query": {
        "match": {
            "title": "jumping rabbit"
        }
    }
}

# 多字段查询

GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", 
            "fields": [ "title", "title.std" ]
        }
    }
}

# 添加boot比重参数
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] 
        }
    }
}

# 跨字段实体搜索
{
  "query": {
    "multi_match": {
      "query":       "Poland Street W1V",
      "type":        "most_fields",
      "fields":      [ "street", "city", "country", "postcode" ]
    }
  }
}

```


用 most_fields 这种方式搜索也存在某些问题：

+	它是为多数字段匹配`任意`词设计的，而不是在`所有字段`中找到最匹配的。
+	它不能使用 operator 或 minimum_should_match 参数来降低次相关结果造成的长尾效应。
+	词频对于每个字段是不一样的，而且它们之间的相互影响会导致不好的排序结果。

以上三个源于 most_fields 的问题都因为它是 `字段中心式`（field-centric） 而不是 `词中心式`（term-centric） 的：当真正感兴趣的是匹配词的时候，它为我们查找的是最匹配的 字段 。


字段中心式的问题如下：

+	白化象问题(albino elephant) — 有更多搜索词匹配的文档没有被排在靠前的位置
+	信号冲突(signal discordance) — 多字段 idf 不一，排序难被用户理解


***字段中心式与词中心式的区别***

+	字段中心式：用文本去匹配字段；各个字段分别算分，然后按一定 function 求最终结果
+	词中心式：文本分词；词在不同字段中的分数，通过一定 function 算出（图中为 max）,然后再将各个词的分数结合不同比重计算最后的总和



 
##### 1.4 跨字段匹配（cross_fields） 

cross_fields 策略：质量兼具

cross_fields 使用词中心式（term-centric）的查询方式，这与 best_fields 和 most_fields 使用字段中心式不同。

cross_fields 类型首先分析查询字符串并生成一个词列表，然后它从所有字段中依次搜索每个词。这种不同的搜索方式很自然的解决了 字段中心式 查询三个问题中的二个。剩下的问题是逆向文档频率不同。

```
# 它通过 混合 不同字段逆向索引文档频率的方式解决了词频的问题
# 它会同时在 first_name 和 last_name 两个字段中查找 smith 的 IDF ，然后用两者的最小值作为两个字段的 IDF
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields", 
            "operator":    "and",
            "fields":      [ "first_name", "last_name" ]
        }
    }
}

# 提高字段权重
GET /books/_search
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields",
            "fields":      [ "title^2", "description" ] 
        }
    }
}
```


> 为了让 cross_fields 查询以最优方式工作，所有的字段都须使用相同的分析器，具有相同分析器的字段会被分组在一起作为混合字段使用。


##### 1.5 最佳字段

在下面的例子中， title 和 body 字段是相互竞争的关系，但我们需要找到单个 最佳匹配 的字段。

```
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}

{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}

# 第一个数据 在两个字段都包含‘brown’，而第二个数据只有一个数据包含，所以第一个数据分数更高
# 但是，这个查询中两个字段是相互竞争的关系，所以两字段的分数不应该相互影响。

{
  "hits": [
     {
        "_id":      "1",
        "_score":   0.14809652,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     },
     {
        "_id":      "2",
        "_score":   0.09256032,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     }
  ]
}
```

bool 是如何计算评分的：

+	它会执行 should 语句中的两个查询。
+	加和两个查询的评分。
+	乘以匹配语句的总数。
+	除以所有语句总数（这里为：2）。


***dis_max ***

这里可以使用 dis_max 即分离 最大化查询（Disjunction Max Query） 。分离（Disjunction）的意思是 或（or） ，这与可以把结合（conjunction）理解成 与（and） 相对应。分离最大化查询（Disjunction Max Query）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 ：

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}

{
  "hits": [
     {
        "_id":      "2",
        "_score":   0.21509302,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id":      "1",
        "_score":   0.12713557,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     }
  ]
}
```

可以通过指定 tie_breaker 这个参数将其他匹配语句的评分也考虑其中：

```
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}

tie_breaker  参数提供了一种 dis_max 和 bool 之间的折中选择，它的评分方式如下：

获得最佳匹配语句的评分 _score 。
将其他匹配语句的评分结果与 tie_breaker 相乘。
对以上评分求和并规范化。
```

> tie_breaker 可以是 0 到 1 之间的浮点数，其中 0 代表使用 dis_max 最佳匹配语句的普通逻辑， 1 表示所有匹配语句同等重要。最佳的精确值需要根据数据与查询调试得出，但是合理值应该与零接近（处于 0.1 - 0.4 之间），这样就不会颠覆 dis_max 最佳匹配性质的根本。



### 2. 近似匹配

> 全文搜索被称作是 召回率（Recall） 与 精确率（Precision） 的战场： 召回率 ——返回所有的相关文档； 精确率 ——不返回无关文档。目的是在结果的第一页中为用户呈现最为相关的文档。
为了提高召回率的效果，我们扩大搜索范围——不仅返回与用户搜索词精确匹配的文档，还会返回我们认为与查询相关的所有文档。如果一个用户搜索 “quick brown box” ，一个包含词语 fast foxes 的文档被认为是非常合理的返回结果。


使用 TF/IDF 的标准全文检索将文档或者文档中的字段作一大袋的词语处理。 `match`查询可以告知我们这大袋子中是否包含查询的词条，但却无法告知词语之间的关系。用 `match` 搜索匹配的只是单词，但在实际使用中我们更希望该词组与文档有很高的相关度，而不是分散在文档的几个段落中。

这就是短语匹配或者近似匹配的所属领域。


##### 2.1 短语匹配

***什么是短语匹配***

例如`quick brown fox `就是一个短语，要想匹配这个短语，需要满足以下条件：
```
quick 、 brown 和 fox 需要全部出现在域中。
brown 的位置应该比 quick 的位置大 1 。
fox 的位置应该比 quick 的位置大 2 
```

如果以上任何一个选项不成立，则该文档不能认定为匹配。



***match_phrase***

就像 `match` 查询对于标准全文检索是一种最常用的查询一样，当你想找到彼此邻近搜索词的查询方法时，就会想到 `match_phrase` 查询。

> 本质上来讲，match_phrase 查询是利用一种低级别的 span 查询族（query family）去做词语位置敏感的匹配。 Span 查询是一种词项级别的查询，所以它们没有分词阶段；它们只对指定的词项进行精确搜索。
值得庆幸的是，match_phrase 查询已经足够优秀，大多数人是不会直接使用 span 查询。 然而，在一些专业领域，例如专利检索，还是会采用这种低级别查询去执行非常具体而又精心构造的位置搜索。

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

GET /qw/_search
{
    "query": {
        "match_phrase": {
            "title": "the lazy dog"
        }
    }
}
```

类似 match 查询， match_phrase 查询首先将查询字符串解析成一个词项列表，然后对这些词项进行搜索，但只保留那些包含 全部 搜索词项，且 位置 与搜索词项相同的文档。


> 位置信息可以被存储在倒排索引中，因此 match_phrase 查询这类对词语位置敏感的查询， 就可以利用位置信息去匹配包含所有查询词项，且各词项顺序也与我们搜索指定一致的文档，中间不夹杂其他词项。
在ES2.x，多值需要添加`position_increment_gap`来增大词距，避免被匹配错误。

##### 2.2 邻近匹配

精确短语匹配 或许是过于严格了。也许我们想要包含 “quick brown fox” 的文档也能够匹配 “quick fox,” ， 尽管情形不完全相同。

通过使用 slop 参数，可以将灵活度引入短语匹配中：

```
GET /qw/_search
{
    "query": {
        "match_phrase": {
            "title": {
				"query":"the dog",
				"slop":10
			}
        }
    }
}
```

slop 参数告诉 match_phrase 查询词条相隔多远时仍然能将文档视为匹配。相隔多远的意思是，你需要移动一个词条多少次来让查询和文档匹配.


> 鉴于一个短语查询仅仅排除了不包含确切查询短语的文档， 而 邻近查询 — 一个 slop 大于 0— 的短语查询将查询词条的邻近度考虑到最终相关度 _score 中。 通过设置一个像 50 或者 100 这样的高 slop 值, 你能够排除单词距离太远的文档， 但是也给予了那些单词临近的的文档更高的分数




***使用邻近度提高相关度***

虽然邻近查询很有用， 但是所有词条都出现在文档的要求过于严格了。

我们可以将一个简单的 match 查询作为一个 must 子句。 这个查询将决定哪些文档需要被包含到结果集中。 我们可以用 minimum_should_match 参数去除长尾。 然后我们可以以 should 子句的形式添加更多特定查询。 每一个匹配成功的都会增加匹配文档的相关度。


例如：

```
GET /qw/_search
{
  "query": {
    "bool": {
      "must": {
        "match": { 
          "title": {
            "query":                "the lazy dog",
            "minimum_should_match": "30%"
          }
        }
      },
      "should": {
        "match_phrase": { 
          "title": {
            "query": "the dog",
            "slop":  50
          }
        }
      }
    }
  }
}
```

+	must 子句从结果集中包含或者排除文档。
+	should 子句增加了匹配到文档的相关度评分。

##### 2.3 优化


***性能优化***

短语查询和邻近查询都比简单的 query 查询代价更高。 一个 match 查询仅仅是看词条是否存在于倒排索引中，而一个 match_phrase 查询是必须计算并比较多个可能重复词项的位置。

Lucene nightly benchmarks 表明一个简单的 term 查询比一个短语查询大约快 10 倍，比邻近查询(有 slop 的短语 查询)大约快 20 倍。当然，这个代价指的是在搜索时而不是索引时。



我们可以从每个分片中获得前 K 个结果。 然后会根据它们的最新评分 重新排序。这样就可以提高查询效率：

```
GET /qw/_search
{
    "query": {
        "match": {  
            "title": {
                "query":"the lazy dog",
                "minimum_should_match": "30%"
            }
        }
    },
    "rescore": {
        "window_size": 50, 
        "query": {         
            "rescore_query": {
                "match_phrase": {
                    "title": {
                        "query": "the lazy dog",
                        "slop":  50
                    }
                }
            }
        }
    }
}
```



> 标准全文数据的短语查询通常在几毫秒内完成，因此实际上都是完全可用，即使是在一个繁忙的集群上。在某些特定病理案例下，短语查询可能成本太高了，但比较少见。一个典型例子就是DNA序列，在序列里很多同样的词项在很多位置重复出现。在这里使用高 slop 值会到导致位置计算大量增加。

***同义词***

短语查询和邻近查询都很好用，但仍有缺点：
+	它们过于严格了：为了匹配短语查询，所有词项都必须存在，即使使用了 slop 。
+	用 slop 得到的单词顺序的灵活性也需要付出代价，因为失去了单词对之间的联系，短语的语义可能相去甚远。

我们可以配置相关的同义词插件以及词库，提高搜索的准确度。


 

### 3. 部分匹配

目前为止，介绍的所有查询都是针对整个词的操作。为了能匹配，只能查找倒排索引中存在的词，最小的单元为单个词。

在SQL中，可以通过模糊查询，匹配部分词：

```
 WHERE text LIKE "%the%"
      AND text LIKE "%lazy%"
      AND text LIKE "%dog%" 
```

但是，在某些情况下部分匹配会比较有用，常见的应用如下：

+	匹配邮编、产品序列号或其他（特定前缀）
+	输入即搜索（search-as-you-type） ——在用户键入搜索词过程的同时就呈现最可能的结果
+	匹配如德语或荷兰语这样有长组合词的语言，如： Weltgesundheitsorganisation （世界卫生组织，英文 World Health Organization）。

##### 3.1 邮编与结构化数据

接下来，我会使用美国目前使用的邮编形式（United Kingdom postcodes 标准）来说明如何用部分匹配查询结构化数据。这种邮编形式有很好的结构定义。

我国采用四级六位编码制：
+	前两位表示 省|市|自治区
+	第三位代表 邮区
+	第四位代表 县
+	后两位代表 投递邮局

例如：130021
`“13”代表吉林省，“00”代表省会长春，“21”代表所在投递区。`

添加测试数据:

```
DELETE /my_mail
POST /my_mail/_bulk
{ "index": { "_id": 1 }}
{ "postcode": "130021" }
{ "index": { "_id": 2 }}
{ "postcode": "120021" }
{ "index": { "_id": 3 }}
{ "postcode": "130020" }
{ "index": { "_id": 4 }}
{ "postcode": "131121" }
```

***通过prefix 前缀查询编码***

prefix 查询是一个词级别的底层的查询，它不会在搜索之前分析查询字符串，它假定传入前缀就正是要查找的前缀。

例如：
```
GET /my_mail/_search
{
    "query": {
        "prefix": {
            "postcode.keyword": "12"
        }
    }
}
```

>  prefix 查询不做相关度评分计算，它只是将所有匹配的文档返回，并为每条结果赋予评分值 1 。它的行为更像是过滤器而不是查询。 prefix 查询和 prefix 过滤器这两者实际的区别就是过滤器是可以被缓存的，而查询不行。
prefix 查询或过滤对于一些特定的匹配是有效的，但使用方式还是应当注意。当字段中词的集合很小时，可以放心使用，但是它的伸缩性并不好，会对我们的集群带来很多压力。可以使用较长的前缀来限制这种影响，减少需要访问的量。

***通配符与正则表达式查询***

与 prefix 前缀查询的特性类似， wildcard 通配符查询也是一种底层基于词的查询，与前缀查询不同的是它允许指定匹配的正则式。
它使用标准的 shell 通配符查询： 
+	`? 匹配任意字符`
+	`* 匹配 0 或多个字符`
+	`[]匹配多个,例如 [0-9] [a-z] [1,2,3]`

wildcard 通配符查询：
```
GET /my_mail/_search
{
    "query": {
        "wildcard": {
            "postcode.keyword": "13*2?"
        }
    }
}

```

regexp 正则式查询允许写出这样更复杂的模式：

```
GET /my_mail/_search
{
    "query": {
        "regexp": {
            "postcode.keyword": "1[0-9].+"
        }
    }
}
```

> regexp 正则查询允许使用正则表达式的相关语法。


##### 3.2 输入即搜索（search-as-you-type）

户已经渐渐习惯在输完查询内容之前，就能为他们展现搜索结果，这就是所谓的 即时搜索（instant search） 或 输入即搜索（search-as-you-type） 。不仅用户能在更短的时间内得到搜索结果，我们也能引导用户搜索索引中真实存在的结果。

在 短语匹配 中，我们引入了 match_phrase 短语匹配查询，它匹配相对顺序一致的所有指定词语，对于查询时的输入即搜索，可以使用 match_phrase 的一种特殊形式， match_phrase_prefix 查询。


例如：

```
# 可以匹配 Johnnie Walker Black Label 和 Johnnie Walker Blue Label 。

GET /qw/_search
{
    "query": {
        "match_phrase_prefix": {
            "title": "The quick br"
        }
    }
}
```

这种查询的行为与 match_phrase 查询一致，不同的是它将查询字符串的最后一个词作为前缀使用.
可以通过设置 max_expansions 参数来限制前缀无限匹配的影响，例如：
```
{
    "match_phrase_prefix" : {
        "brand" : {
            "query":          "johnnie walker bl",
            "max_expansions": 50
        }
    }
}
```


##### 3.3 Ngrams


***N-gram基本原理***

N-gram是计算机语言学和概率论范畴内的概念，是指给定的一段文本或语音中N个项目（item）的序列。项目（item）可以是音节、字母、单词或碱基对。通常N-grams取自文本或语料库。

> 
在整个语言环境中，句子T的出现概率是由组成T的N个item的出现概率组成的，如下公式所示：
+	P(T)=P(W1W2W3Wn)=P(W1)P(W2|W1)P(W3|W1W2)…P(Wn|W1W2…Wn-1)
该公式难以投入实际使用，此时出现马尔科夫模型，该模型认为，一个词的出现仅仅依赖于它前面出现的几个词。这就大大简化了上述公式（`P(W1)P(W2|W1)P(W3|W1W2)…P(Wn|W1W2…Wn-1)≈P(W1)P(W2|W1)P(W3|W2)…P(Wn|Wn-1)`)。
通常采用bigram和trigram进行计算。

20世纪80年代至90年代初,n-gram技术被广泛地用来进行文本压缩,检查拼写错误,加速字符串查找,文献语种识别。90年代,该技术又在自然语言处理自动化领域得到新的应用,如自动分类,自动索引,超链的自动生成,文献检索,无分隔符语言文本的切分等。

目前N-gram最为有用的就是自然语言的自动分类功能。基于n-gram的自动分类方法有两大类,一类是人工干预的分类(Classification),又称分类;一类是无人工干预的分类(Clustering),又称聚类。

人工干预的分类,是指人工预先分好类(如Yahoo!的层次结构类),然后,计算机根据特定算法自动地将新添加到数据库的文献划归某一类。这类方法缺点是,人们须预先具备关于整个文献库和分类的知识。无人工干预的分类,是指计算机自动地识别文献组(集合),人们勿需预先具备关于整个文献库和分类的知识。




例如：将“informationretrieval”视为一段文本，它的5-grams的items依次为：
	`infor,nform,forma,ormat,rmati,matio,ation,tion,ionr,onre,nret,retr,retri,etrie,triev,rieva,ieval`

***edge_ngram 和 ngram***

edge_ngram和ngram是elasticsearch内置的两个分词器(tokenizer)和过滤器（filter）。
+	ngram, 类似一个向右移动的游标窗口, 把窗口中看到的部分内容进行索引.token_chars 默认包含所有字符, 如果设置为 letter, 则对每个词分别执行(不会出现跨词的窗口). 要求 max_gram 比 min_gram 最多长一位

+	edge_ngram 只从每个分段的边缘开始(不会出现词中的窗口). max_gram 可以比 min_gram长任意位


Ngrams是一种将一个标记分成一个单词的每个部分的多个子字符的方法。 
ngram和edge ngram过滤器都允许您指定min_gram以及max_gram设置，这些设置控制单词被分割成的标记的大小。 这可能令人困惑，让我们看一个例子。 

假设分析“autocomplete”这个词, 结果如下

1. `ngrams`解析词（min_gram为2且max_gram为3）:
+	au, to, co, mp, le, te
+	aut, oco, mpl, ete

2. `Edge ngrams`解析词（min_gram设置为2并将max_gram设置为6）：
+	au, aut, auto, autoc, autoco

***Ngrams 在部分匹配的应用***

在搜索之前准备好供部分匹配的数据可以提高搜索的性能。

在索引时准备数据意味着要选择合适的分析链，这里部分匹配使用的工具是 n-gram 。可以将 n-gram 看成一个在词语上 滑动窗口 ， n 代表这个 “窗口” 的长度。如果我们要使用 `n-gram`分析 `quick` 这个词 —— 

它的结果取决于 n 的选择长度：
```
长度 1（unigram）： [ q, u, i, c, k ]
长度 2（bigram）： [ qu, ui, ic, ck ]
长度 3（trigram）： [ qui, uic, ick ]
长度 4（four-gram）： [ quic, uick ]
长度 5（five-gram）： [ quick ]
```

对于输入即搜索（search-as-you-type）这种应用场景，我们会使用一种特殊的 n-gram 称为 边界 n-grams （edge n-grams）。
所谓的边界 n-gram 是说它会固定词语开始的一边，以单词 quick 为例，它的边界 n-gram 的结果为：
```
q
qu
qui
quic
quick
```


***索引时输入即搜索***

设置索引时输入即搜索的第一步是需要定义好分析链

第一步：需要配置一个自定义的 edge_ngram token 过滤器，称为 autocomplete_filter ：
第二步：在一个自定义分析器 autocomplete 中使用上面这个 token 过滤器

```
# 对于这个 token 过滤器接收的任意词项，过滤器会为之生成一个最小固定值为 1 ，最大为 20 的 n-gram 
# 这个分析器使用 standard 分词器将字符串拆分为独立的词，并且将它们都变成小写形式，然后为每个词生成一个边界 n-gram

DELETE /my_index

PUT /my_index
{
  "settings": {
	"analysis": {
	  "analyzer": {
		"my_analyzer": {
		  "tokenizer": "my_tokenizer"
		}
	  },
	  "tokenizer": {
		"my_tokenizer": {
		  "type": "edge_ngram",
		  "min_gram": 2,
		  "max_gram": 10,
		  "token_chars": [
			"letter",
			"digit"
		  ]
		}
	  }
	}
  }
}

POST /my_index/_bulk
{ "index": { "_id": 1 }}
{ "title": "The quick brown fox" }
{ "index": { "_id": 2 }}
{ "title": "The quick brown fox jumps over the lazy dog" }
{ "index": { "_id": 3 }}
{ "title": "The quick brown fox jumps over the quick dog" }
{ "index": { "_id": 4 }}
{ "title": "Brown fox brown dog" }
{ "index": { "_id": 5 }}
{ "title": "中国你好" }
```




测试索引即搜索：

```
GET /my_index/_search
{
    "query": {
        "match": {
            "title": "brown "
        }
    }
}
```

通过这个方式，可以匹配到结果，但是并没有包含“fo”的短语，所以它是进行了前缀匹配


> 使用边界 n-grams 进行输入即搜索（search-as-you-type）的查询设置简单、灵活且快速，但有时候它并不够快，特别是当试图立刻获得反馈时，延迟的问题就会凸显，很多时候不搜索才是最快的搜索方式。
Elasticsearch 里的 completion suggester 采用与上面完全不同的方式，需要为搜索条件生成一个所有可能完成的词列表，然后将它们置入一个 有限状态机（finite state transducer） 内，这是个经优化的图结构。为了搜索建议提示，Elasticsearch 从图的开始处顺着匹配路径一个字符一个字符地进行匹配，一旦它处于用户输入的末尾，Elasticsearch 就会查找所有可能结束的当前路径，然后生成一个建议列表。


***Ngrams 在复合词的应用***

有些人希望在搜索 “Wörterbuch”（字典）的时候，能在结果中看到 “Aussprachewörtebuch”（发音字典）。同样，搜索 “Adler”（鹰）的时候，能将 “Weißkopfseeadler”（秃鹰）包括在结果中。

处理这种语言的一种方式可以用 组合词 token 过滤器（compound word token filter） 将复合词拆分成各自部分，但这种方式的结果质量依赖于组合词字典的质量。另一种方式就是将所有的词用 n-gram 进行处理，然后搜索任何匹配的片段——能匹配的片段越多，文档的相关度越大。
 
 

[近似匹配](https://www.elastic.co/guide/cn/elasticsearch/guide/current/proximity-matching.html)
[部分匹配](https://www.elastic.co/guide/cn/elasticsearch/guide/current/partial-matching.html)
[多字段搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-field-search.html)
[N-gram的原理、用途和研究](https://www.cnblogs.com/cdsj/p/5720391.html	)