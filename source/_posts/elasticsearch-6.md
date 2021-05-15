---
title: ES（五）处理人类语言——词汇识别
date: 2020-08-12 23:12:30
tags: [elasticsearch]
---

全文搜索是一场 `查准率` 与 `查全率` 之间的较量—查准率即尽量返回较少的无关文档，而查全率则尽量返回较多的相关文档。 尽管能够精准匹配用户查询的单词，但这仍然不够，我们会错过很多被用户认为是相关的文档。 因此，我们需要把网撒得更广一些，去搜索那些和原文不是完全匹配但却相关的单词。

ES 为很多世界流行语言提供良好的、简单的、开箱即用的语言分析器,可以分析常见语言，包括且不限于英语、日语、中文等等。

分析器的功能如下
+	文本拆分为单词：`The quick brown foxes → [ The, quick, brown, foxes]`
+	大写转小写：`The → the`
+	移除常用的 停用词：`[ The, quick, brown, foxes] → [ quick, brown, foxes]`
+	将变型词（例如复数词，过去式）转化为词根：`foxes → fox`


在我们可以操控单个单词之前，需要先将文本切分成单词，这也意味着我们需要知道 单词 是由什么组成的。

> Elasticsearch 的内置分析器都是全局可用的，不需要提前配置.
默认的分析器是 standard（标准）分析器
 

### 1. 语言分析器

默认的中文分析器效果差，在实际使用中一般需要配置 IK 分析器。


#####  1.1 中文分析器（IK）
***安装与配置***


[IK(下载)](https://github.com/medcl/elasticsearch-analysis-ik/releases)。

```
	#安装配置
	bin/elasticsearch-plugin install  https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.8.0/elasticsearch-analysis-ik-7.8.0.zip

	# 重启es，查看日志如下,加载了分词器插件
		...l
		[2020-07-18T10:12:50,682][INFO ][o.e.p.PluginsService     ] [node-1] loaded plugin [analysis-ik]
		...

	# 测试
	# 添加数据
	curl -XPOST http://localhost:9200/index/_create/ -H 'Content-Type:application/json' -d' {"text":"阿伯蛇加速跳"}'
	curl -XGET http://localhost:9200/index/_search
```


***配置停用词***

```
vim 

:wq
```


##### 1.2 混合语言处理

> 德语的词干提取规则跟英语，法语，瑞典语等是不一样的。 为不同的语言提供同样的词干提规则 将会导致有的词的词根找的正确，有的词的词根找的不正确，有的词根本找不到词根。 甚至是将不同语言的不同含义的词切为同一个词根，合并这些词根的搜索结果会给用户带来困恼。

正确的处理多语言类型文档是非常困难的。如果对所有的域使用 standard （标准）分析器， 那么你的文档会变得不利于搜索。混合语言域包含多种语言，那么每包含一种语言就需要配置对应的词干提取器。 

但是，提供多种的词干提取器轮流切分同一份文档的结果很有可能得到一堆垃圾，因为下一个词干提取器会尝试切分一个已经被缩减为词干的单词。

每种语言都有自己的语法，可以为同一份文本提供多种词干提取器，提取器所使用的文本应当相互独立。



多语言文档类型如下

+	每个文档对应一种语言（index-per-language）
+	每个域对应一种语言（field-per-language）
+	混合语言域

***每份文档一种语言( index-per-language )***

文档有多种语言，一种语言配置一种索引。例如

```
PUT /blogs-en
{
  "mappings": {
    "post": {
      "properties": {
        "title": {
          "type": "string", 
          "fields": {
            "stemmed": {
              "type":     "string",
              "analyzer": "english" 
            }
}}}}}}

PUT /blogs-fr
{
  "mappings": {
    "post": {
      "properties": {
        "title": {
          "type": "string", 
          "fields": {
            "stemmed": {
              "type":     "string",
              "analyzer": "french" 
            }
}}}}}}

# 查询
GET /blogs-*/post/_search 
{
    "query": {
        "multi_match": {
            "query":   "deja vu",
            "fields":  [ "title", "title.stemmed" ] 
            "type":    "most_fields"
        }
    },
    "indices_boost": { 
        "blogs-en": 3,
        "blogs-fr": 2
    }
}

```

这样配置的好处是：干净且灵活。新语言很容易被添加 — 仅仅是创建一个新索引—​因为每种语言都是彻底的被分开， 我们不用遭受在 混合语言的陷阱 中描述的词频和词干提取的问题。

***每个域一种语言（field-per-language）***
每个 field （域）有自己的主语言, 并包含一些其他语言的片段。一个文档可以有多个field.

例如

```
PUT /movies
{
  "mappings": {
    "movie": {
      "properties": {
        "title": { 
          "type":       "string"
        },
        "title_br": { 
            "type":     "string",
            "analyzer": "brazilian"
        },
        "title_cz": { 
            "type":     "string",
            "analyzer": "czech"
        },
        "title_en": { 
            "type":     "string",
            "analyzer": "english"
        },
        "title_es": { 
            "type":     "string",
            "analyzer": "spanish"
        }
      }
    }
  }
}

PUT /movies
{
   "title":     "Fight club",
   "title_br":  "Clube de Luta",
   "title_cz":  "Klub rváčů",
   "title_en":  "Fight club",
   "title_es":  "El club de la lucha",
   ...
}

GET /movies/_search
{
    "query": {
        "multi_match": {
            "query":    "club de la lucha",
            "fields": [ "title*", "title_es^2" ], 
            "type":     "most_fields"
        }
    }
}

```

添加新语言：分析器只能在索引创建时被装配。有一个变通的方案，你可以先关闭这个索引 close ，然后使用 update-settings API ，重新打开这个索引，但是关掉这个索引意味着得停止服务一段时间。



***混合语言域***

混合语言的处理方式：
+	切分到不同的域（根据语言切分文本）
	+	如果文档来自第三方资源且没经过语言归类，或者是不正确的归类。这种情况下，可以使用学习算法来归类你文档的主语言。
	+	例如，使用 `chromium-compact-language-detector` 工具包
+	 多次分析(analyze-multiple times)
	+	处理数量有限的语言， 你可以使用多个域，每种语言都分析文本一次。
	+	```
		PUT /movies
		{
		  "mappings": {
			"title": {
			  "properties": {
				"title": { 
				  "type": "string",
				  "fields": {
					"de": { 
					  "type":     "string",
					  "analyzer": "german"
					},
					"en": { 
					  "type":     "string",
					  "analyzer": "english"
					}
				  }
				}
			  }
			}
		  }
		}
		# 主域 title 使用 standard （标准）分析器
		# 每个子域提供不同的语言分析器来对 title 域文本进行分析。
	```
+	使用 n-grams
	+	```
		PUT /movies
		{
		  "settings": {
			"analysis": {...} 
		  },
		  "mappings": {
			"title": {
			  "properties": {
				"title": {
				  "type": "string",
				  "fields": {
					"de": {
					  "type":     "string",
					  "analyzer": "german"
					},
					"en": {
					  "type":     "string",
					  "analyzer": "english"
					},
					"fr": {
					  "type":     "string",
					  "analyzer": "french"
					},
					"es": {
					  "type":     "string",
					  "analyzer": "spanish"
					},
					"general": { 
					  "type":     "string",
					  "analyzer": "trigrams"
					}
				  }
				}
			  }
			}
		  }
		}
	```

### 2. 词汇识别

中文博大精深，一词多音是很常见的，所以要识别词汇是没那么简单的。还好，Elasticsearch 为很多语言提供了专用的分析器， 其他特殊语言的分析器以插件的形式提供，`IK`就是一个中文分析器。

然而并不是所有语言都有专用分析器，而且有时候你甚至无法确定处理的是什么语言。这种情况，可以使用标准工具包进行简单的分析。

***标准分析器***

分析器分析的过程就是将文本转换为标记（tokens）或术语的过程，任何全文检索的字符串域都默认使用 standard 分析器。 

如果我们想要自定义分析器 ，可以按照如下定义方式重新实现标准分析器：

```
PUT /my_analyzer
{
	"settings": {
		"number_of_shards":1
	},
	"mappings": {
		"properties": {
			"title": {
				"type":"text",
				"analyzer": {
					"type":      "custom",
					"tokenizer": "standard",
					"filter":  [ "lowercase", "stop" ]
				}
			}
		}
	}
}
```

***标准分词器（tokenizer）***

分词器 接受一个字符串作为输入，将这个字符串拆分成独立的词或 语汇单元（token） （可能会丢弃一些标点符号等字符），然后输出一个 语汇单元流（token stream） 。
 
> 分词器在字符过滤器之后工作，用于把文本分割成多个标记（Token），一个标记基本上是词加上一些额外信息，分词器的处理结果是标记流，它是一个接一个的标记，准备被过滤器处理。

##### 2.1 分析器

分析器可以包含三个部分：字符过滤器（character filters）、分词器（tokenizer）和标记过滤器（token filters）。一个分析器不一定这三个部分都有，但是一般会包含分词器。


工作流程：
+	字符过滤器对分析（analyzed）文本进行过滤和处理，例如从原始文本中移除HTML标记，根据字符映射替换文本等；
+	过滤之后的文本被分词器接收，分词器把文本分割成标记流；
+	然后，标记过滤器对标记流进行过滤处理
+	最终，过滤之后的标记流被存储在倒排索引中


1. 字符过滤器

+	`html_strip`：HTML字符过滤器（HTML Strip Char Filter）去除HTML元素
+	`mapping`：映射字符过滤器（Mapping Char Filter）接收键值的映射，每当遇到与键相同的字符串时，它就用该键关联的值替换它们。
+	`pattern_replace`：模式替换过滤器（Pattern Replace Char Filter）使用正则表达式匹配并替换字符串中的字符

例如：
```

POST _analyze
{
  "tokenizer": "keyword",
  "char_filter": ["html_strip"],
  "text":"<p>I&apos;m so <b>happy</b>!</p>"
}

DELETE char_filter_mapping
PUT /char_filter_mapping
{
	"settings": {
		"analysis" : {
			"char_filter" : {
				"my_mapping" : {
					"type" : "mapping",
					"mappings" : [
					  "沙雕 => sd"
					]
				}
			},
			"analyzer" : {
				"custom_with_char_filter" : {
					"tokenizer" : "standard",
					"char_filter" : ["my_mapping"]
				}
			}
		}
	}
}
 
PUT pattern_test5
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "pattern_replace",
          "pattern": "(\\d+)-(?=\\d)",
          "replacement": "$1_"
        }
      }
    }
  }
}
```



2. 分词器
+	`whitespace`分词器：按空白字符 —— 空格、tabs、换行符等等进行简单拆分。
+	`letter`分词器：字母分词器。
+	`standard`分词器： 标准分词器类型是standard，用于大多数欧洲语言，使用Unicode文本分割算法对文档进行分词。
+	`lowercase`分词器： 在非字母位置上分割文本，并把分词转换为小写形式
+	`classic `分词器： 基于语法规则对文本进行分词，对英语文档分词非常有用


3. 标记过滤器（Token Filter）

+	小写标记过滤器（Lowercase）：用于把标记转换为小写形式
+	停用词标记过滤器（Stopwords）：类型是stop，用于从标记流中移除停用词（例如：这，是，不是，那）
+	词干过滤器（Stemmer）：类型是stemmer，用于把词转换为其词根形式存储在倒排索引，能够减少标记
+	同义词过滤器（Synonym）：类型是synonym，在分析阶段，基于同义词规则，把词转换为其同义词存储在倒排索引中





##### 2.2 系统预定义的分析器

1. 标准分析器（Standard）

分析器类型是standard，由标准分词器（Standard Tokenizer），标准标记过滤器（Standard Token Filter），小写标记过滤器（Lower Case Token Filter）和停用词标记过滤器（Stopwords Token Filter）组成。参数stopwords用于初始化停用词列表，默认是空的。

2. 简单分析器（Simple）

分析器类型是simple，实际上是小写标记分词器（Lower Case Tokenizer），在非字母位置上分割文本，并把分词转换为小写形式，功能上是Letter Tokenizer和 Lower Case Token Filter的结合（Combination），但是性能更高，一次性完成两个任务。

3. 空格分析器（Whitespace）

分析器类型是whitespace，实际上是空格分词器（Whitespace Tokenizer)。

4. 停用词分析器（Stopwords）

分析器类型是stop，由小写分词器（Lower Case Tokenizer）和停用词标记过滤器（Stop Token Filter）构成，配置参数stopwords 或 stopwords_path指定停用词列表。

5. 雪球分析器（Snowball）

分析器类型是snowball，由标准分词器（Standard Tokenizer），标准过滤器（Standard Filter），小写过滤器（Lowercase Filter），停用词过滤器（Stop Filter）和雪球过滤器（Snowball Filter）构成。参数language用于指定语言。


> [elasticsearch 分析器 分词器](https://www.cnblogs.com/gmhappy/p/9472377.html)


##### 2.3  ICU 分析器

Elasticsearch的 ICU 分析器插件 使用 国际化组件 Unicode (ICU) 函数库（详情查看 site.project.org ）提供丰富的处理 Unicode 工具。 这些包含对处理亚洲语言特别有用的 icu_分词器 ，还有大量对除英语外其他语言进行正确匹配和排序所必须的分词过滤器。

icu_分词器 和 标准分词器 使用同样的 Unicode 文本分段算法， 只是为了更好的支持亚洲语，添加了泰语、老挝语、中文、日文、和韩文基于词典的词汇识别方法，并且可以使用自定义规则将缅甸语和柬埔寨语文本拆分成音节。

*** 安装***

安装ICU:
`sudo bin/elasticsearch-plugin install analysis-icu`

查看：
`./bin/elasticsearch-plugin list`

删除：
`sudo bin/elasticsearch-plugin remove analysis-icu` 

***使用***

```
GET _analyze
{
  "text": ["他是一个前端开发工程师"],
  "analyzer": "icu_analyzer"
}
```


> [ICU 分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/icu-plugin.html)


