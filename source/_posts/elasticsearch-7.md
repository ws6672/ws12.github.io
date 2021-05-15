---
title: ES（七）处理人类语言——文本特征抽取
date: 2020-08-14 15:48:39
tags: [elasticsearch]
---

ES的全文搜索除了识别词汇，还需要懂得识别词意。


### 1. 归一化词元

把文本切割成词元(token)只是这项工作的一半。为了让这些词元(token)更容易搜索, 这些词元(token)需要被 归一化(normalization)--这个过程会去除同一个词元(token)的无意义差别，例如大写和小写的差别。归一化这个过程中，我们通过相同的方式去处理所有的词语，使词元更加规范。

用的最多的语汇单元过滤器(token filters)是 lowercase 过滤器，它的功能正和你期望的一样；它将每个词元(token)转换为小写形式：。例如：

```
GET /_analyze?tokenizer=standard&filters=lowercase
The QUICK Brown FOX! 

词语识别：The, QUICK, Brown, FOX
归一化：the, quick, brown, fox
```

***去掉变音符号***

英语用变音符号(例如 ´, ^, 和 ¨) 来强调单词—​例如 rôle, déjà, 和 däis —​但是是否使用他们通常是可选的. 其他语言则通过变音符号来区分单词。对于西方语言，可以用 asciifolding 字符过滤器来实现这个功能。 实际上，它不仅仅能去掉变音符号。它会把Unicode字符转化为ASCII来表示。

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "folding": {
          "tokenizer": "standard",
          "filter":  [ "lowercase", "asciifolding" ]
        }
      }
    }
  }
}

GET /my_index?analyzer=folding
My œsophagus caused a débâcle 


# 保留原词意思,在子域中去除变音
PUT /my_index/_mapping/my_type
{
  "properties": {
    "title": { 
      "type":           "string",
      "analyzer":       "standard",
      "fields": {
        "folded": { 
          "type":       "string",
          "analyzer":   "folding"
        }
      }
    }
  }
}

```

##### 1.1 Unicode

当Elasticsearch在比较词元(token)的时候，它是进行字节(byte)级别的比较。 换句话说，如果两个词元(token)被判定为相同的话，他们必须是相同的字节(byte)组成的。然而，Unicode允许你用不同的字节来写相同的字符。

如果你的数据有多个来源，就会有可能发生这种状况：因为相同的单词使用了不同的编码，导致一个形式的 déjà 不能和它的其他形式进行匹配。

幸运的是，这里就有解决办法。这里有4种Unicode 归一化形式 (normalization forms) : nfc, nfd, nfkc, nfkd，它们都把Unicode字符转换成对应标准格式，把所有的字符 进行字节(byte)级别的比较。

> Unicode归一化形式 (Normalization Forms)
规范 (canonical) 模式—nfc 和 nfd&—把连字作为单个字符，例如 ﬃ 或者 œ 。 兼容 (compatibility) 模式—nfkc 和 nfkd—将这些组合的字符分解成简单字符的等价物，例如： f + f + i 或者 o + e.


可以使用 icu_normalizer 语汇单元过滤器(token filters) 来保证你的所有词元(token)是相同模式：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "nfkc_normalizer": { 
          "type": "icu_normalizer",
          "name": "nfkc"
        }
      },
      "analyzer": {
        "my_normalizer": {
          "tokenizer": "icu_tokenizer",
          "filter":  [ "nfkc_normalizer" ]
        }
      }
    }
  }
}
```

***关于排序***
归一化后可以进行排序，那么字符串的排序方式大致如下：
+	大小写敏感排序（not_analyzed ）
+	Unicode 排序
+	按语言排序
+	自定义排序

每种语言都可以使用复数域来支持对同一个域进行多规则排序：

### 2. 词干提取(单词归类)

词干提取 试图移除单词的变化形式之间的差别，从而达到将每个词都提取为它的词根形式。 例如 foxes 可能被提取为词根 fox ，移除单数和复数之间的区别跟我们移除大小写之间的区别的方式是一样的。但是，词干提取也有可能遇到以下两个问题：
+	词干弱提取 就是无法将同样意思的单词缩减为同一个词根。
+	词干过度提取 就是无法将不同含义的单词分开.

> 词形还原（词意归类）：原词是一组相关词的规范形式，或词典形式paying 、 paid 和 pays 的原词是 pay 。 通常原词很像与其相关的词，但有时也不像is 、 was 、 am 和 being 的原词是 be 。
词形还原，很像词干提取，试图归类相关单词，但是它比词干提取先进一步的是它企图按单词的 词义 ，或意义归类。 同样的单词可能表现出两种意思—例如， wake 可以表现为 to wake up 或 a funeral 。然而词形还原试图区分两个词的词义，词干提取却会将其混为一谈。
词形还原是一种更复杂和高资源消耗的过程，它需要理解单词出现的上下文来决定词的意思。实践中，词干提取似乎比词形还原更高效，且代价更低。


##### 2.1 词干提取器

***算法词干提取器***
Elasticsearch 中的大部分 stemmers （词干提取器）是基于算法的，它们提供了一系列规则用于将一个词提取为它的词根形式，例如剥离复数词末尾的 s 或 es 。提取单词词干时并不需要知道该词的任何信息。

> 
kstem token filter 是一款合并了词干提取算法和内置词典的英语分词过滤器。为了避免模糊词不正确提取，这个词典包含一系列根词单词和特例单词。 kstem 分词过滤器相较于 Porter 词干提取器而言不那么激进。

可以使用 ​`porter_stem`​ 词干提取器或直接使用 `kstem` 分词过滤器，或使用 `snowball` 分词过滤器创建一个具体语言的 Snowball 词干提取器。所有基于算法的词干提取器都暴露了用来接受 语言 参数的统一接口： `stemmer token filter` 。

***字典词干提取器***

字典词干提取器 在工作机制上与 算法化词干提取器 完全不同。 不同于应用一系列标准规则到每个词上，字典词干提取器只是简单地在字典里查找词。理论上可以给出比算法化词干提取器更好的结果。

用途：
+	返回不规则形式如 feet 和 mice 的正确词干
+	区分出词形相似但词义不同的情形，比如 organ and organization

比较：
+	字典词干提取器对于字典中不存在的词无能为力。而一个基于算法的词干提取器，则会继续应用之前的相同规则，结果可能正确或错误。
+	字典词干提取器需要加载所有词汇、 所有前缀，以及所有后缀到内存中，这会显著地消耗内存；算法化词干提取器通常更简单、轻量和快速

> Elasticsearch 提供了基于词典提取词干的 hunspell 语汇单元过滤器（token filter）. Hunspell hunspell.github.io 是一个 Open Office、LibreOffice、Chrome、Firefox、Thunderbird 等众多其它开源项目都在使用的拼写检查器。
[.oxt 后缀的文件](https://extensions.openoffice.org/)
[.xpi 扩展文件](http://mzl.la/157UORf)
[.zip 文件](http://download.services.openoffice.org/contrib/dictionaries/)


***选择***

对于中文来说，词干提取就不需要的。因为中文的特点是字形一致，字意复杂；但是，也存在多语言并用的情况，可以配置一个英文的词干提取器。


***阻止提取与自定义***

```

# 阻止词干提取
 PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "no_stem": {
          "type": "keyword_marker",
          "keywords": [ "skies" ] 
        }
      },
      "analyzer": {
        "my_english": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "no_stem",
            "porter_stem"
          ]
        }
      }
    }
  }
}

# 自定义提取
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "custom_stem": {
          "type": "stemmer_override",
          "rules": [ 
            "skies=>sky",
            "mice=>mouse",
            "feet=>foot"
          ]
        }
      },
      "analyzer": {
        "my_english": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "custom_stem", 
            "porter_stem"
          ]
        }
      }
    }
  }
}
```


[词干提取算法](https://www.elastic.co/guide/cn/elasticsearch/guide/current/algorithmic-stemmers.html)

### 3. 停用词

停用词：顾名思义，被配置为停用词的词语会被过滤掉，不会被加入到倒排索引中，所以不参与全文搜索

一种最简单的减少索引大小的方法就是 索引更少的词。 以及有些词要比其他词更重要，只索引那些更重要的词来可以大大减少索引的空间。我们可以把高频词（High-frequency terms）以及低频词（Low-frequency terms）添加到停用词表中。

停用词的优缺点:
+	更大的磁盘空间，更好的性能
+	使用停用词减少了搜索的选择


##### 3.1 使用

移除停用词的工作是由 stop 停用词过滤器完成的，可以通过创建自定义的分析器来使用它。但是，也有默认配置停用词的分析器：

+	语言分析器
	+	standard 标准分析器
+	默认使用空的停用词列表：_none_ ，实际上是禁用了停用词
	+	pattern 模式分析器
+	默认使用空的停用词列表：为 _none_ ，与 standard 分析器类似。

```
# 指定停用词
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": { 
          "type": "standard", 
          "stopwords": [ "and", "the" ] 

        }
      }
    }
  }
}
# 停用词会被过滤掉，但两个词项的位置不会变化


# 设置特定语言的停用词
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": { 
          "type": "standard", 
          "stopwords": "_english_" 
        }
      }
    }
  }
}

# 通过文件使用

PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english": {
          "type":           "english",
          "stopwords_path": "stopwords/english.txt" 
        }
      }
    }
  }
}

```

更新分析器的停用词列表有多种方式：
+	分析器在创建索引时
+	当集群节点重启时候
+	或者关闭的索引重新打开的时候。
+	如果使用 stopwords_path 参数指定停用词的文件路径，只需更新文件即可。

***cutoff_frequency ***

在查询字符串中的词项可以分为更重要（低频词）和次重要（高频词）这两类。 只与次重要词项匹配的文档很有可能不太相关。实际上，我们想要文档能尽可能多的匹配那些更重要的词项。

match 查询接受一个参数 cutoff_frequency ，从而可以让它将查询字符串里的词项分为低频和高频两组。低频组（更重要的词项）组成 bulk 大量查询条件，而高频组（次重要的词项）只会用来评分，而不参与匹配过程。通过对这两组词的区分处理，我们可以在之前慢查询的基础上获得巨大的速度提升。

> 删除停用词是能显著降低位置信息所占空间的一种方式。 一个被删除停用词的索引仍然可以使用短语查询，因为剩下的词的原始位置仍然被保存着


### 4. 同义词

词干提取是通过简化他们的词根形式来扩大搜索的范围，同义词 通过相关的观念和概念来扩大搜索范围。用户搜索 “美国” 并且期望找到包含 美利坚合众国 、 美国 、 美洲 、或者 美国各州 的文档。 然而，他们不希望搜索到关于 国事 或者 政府机构 的结果。

区分不同的概念对于人类是很简单的，而对于纯粹的机器是很棘手的事情，需要事先配置同义词，且启用同义词会降低性能。所以，同义词应该只在必要的时候使用。

> 同义词扩大了一个匹配文件的范围。正如 词干提取 或者 部分匹配 ，同义词的字段不应该被单独使用，而应该与一个针对主字段的查询操作一起使用，这个主字段应该包含纯净格式的原始文本。 在使用同义词时，参阅 多数字段 的解释来维护相关性。

##### 4.1 语法

***同义词格式***

1. 使用`逗号分隔`语法
同义词最简单的表达形式是 逗号分隔：`"jump,leap,hop"`

如果遇到这些词项中的任何一项，则将其替换为所有列出的同义词。例如：
```
原始词项:   取代:
────────────────────────────────
jump            → (jump,leap,hop)
leap            → (jump,leap,hop)
hop             → (jump,leap,hop)
```


2. 使用 `=>` 语法
使用 => 语法，可以指定一个词项列表（在左边），和一个或多个替换（右边）的列表：
`"u s a,united states,united states of america => usa"`

```
原始词项:   取代:
────────────────────────────────
u s a           → (usa)
united states   → (usa)
```

3. 如果多个规则指定同一个同义词且顺序无关，它们将被合并在一起; 否则使用最长匹配。例如：
`"united states            => usa"`
`"united states of america => usa" `

如果这些规则相互冲突，Elasticsearch 会将 United States of America 转换为词项 (usa),(of),(america) 。否则，会使用最长的序列，即最终得到词项 (usa) 。


***扩展与收缩***

通过 简单扩展 ，我们可以把同义词列表中的任意一个词扩展成同义词列表 所有 的词：`"jump,hop,leap"`
通过 简单收缩 ，把 左边的多个同义词映射到了右边的单个词：`"leap,hop => jump"`。它必须同时应用于索引和查询阶段，以确保查询词项映射到索引中存在的同一个值。

类型扩展是完全不同于简单收缩 或扩张， 并不是平等看待所有的同义词，而是扩大了词的意义，使被拓展的词更为通用。以这些规则为例：
```
"cat    => cat,pet",
"kitten => kitten,cat,pet",
"dog    => dog,pet"
"puppy  => puppy,dog,pet"
```
***分析链***

假设我们有一个分析器，它由 standard 分词器、 lowercase 的语汇单元过滤器、 synonym 的语汇单元过滤器组成。文本 `U.S.A.` 的分析过程，看起来像这样的：

```
original string（原始文本）                       → "U.S.A."
standard           tokenizer（分词器）            → (U),(S),(A)
lowercase          token filter（语汇单元过滤器）  → (u),(s),(a)
synonym            token filter（语汇单元过滤器）  → (usa)
```

如果用`U.S.A.`的同义词查询，它永远不会匹配任何东西，因为， my_synonym_filter 看到词项的时候，句号已经被移除了，并且字母已经被小写了。

***大小写敏感***

通常，我们把同义词过滤器放置在 lowercase 语汇单元过滤器之后，因此，所有的同义词 都是小写。 但有时会导致奇怪的合并。例如， CAT 扫描和一只 cat 有很大的不同，或者 PET （正电子发射断层扫描）和 pet 。 就此而言，姓 Little 也是不同于形容词 little 的 (尽管当一个句子以它开头时，首字母会被大写)。

如果根据使用情况来区分词义，则需要将同义词过滤器放置在 lowercase 筛选器之前。当然，这意味着同义词规则需要列出所有想匹配的变化（例如， Little、LITTLE、little ）。


***多词同义词和短语查询***

为了能使 短语查询 正常工作， Elasticsearch 需要知道每个词在初始文本中的位置。多词同义词会严重破坏词的位置信息，尤其当新增的同义词标记长度各不相同的时候。

例如：
```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "usa,united states,u s a,united states of america"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_synonyms&text=
The United States is wealthy

Pos 1:  (the)
Pos 2:  (usa,united,u,united)
Pos 3:  (states,s,states)
Pos 4:  (is,a,of)
Pos 5:  (wealthy,america)
```


避免这种混乱的方法是使用 简单收缩， 用单个词项表示所有的同义词， 然后在查询阶段，就只需要针对这单个词进行查询了



##### 4.2 语汇单元过滤器（synonym token filter）

语汇单元过滤器可以使用配置文件配置同义词或者在配置中写明，允许在分析过程中方便地处理同义词。


***实例***


1. 配置文件

```
mkfile conf/analysis/synonym.txt

# 同义词格式如下，“i-pod, i pod” 的同义词是 “ipod”
i-pod, i pod => ipod,
sea biscuit, sea biscit => seabiscuit
ipod, i-pod, i pod => ipod, i-pod, i pod
ipod, i-pod, i pod => ipod
foo => foo bar
foo => baz
foo => foo bar, baz

:wq
```

2. 配置同义词

```
# 通过文件使用同义词
PUT /test_index
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "synonym": {
            "tokenizer": "whitespace",
            "filter": [ "synonym" ]
          }
        },
        "filter": {
          "synonym": {
            "type": "synonym",
            "synonyms_path": "analysis/synonym.txt"
          }
        }
      }
    }
  }
}
```

### 5. 拼写错误

一个好的人性化的搜索，应当具备纠错功能，因为有时候用户也不知道他想要的是什么。Fuzzy matching 允许查询时匹配错误拼写的单词，而语音语汇单元过滤器可以在索引时用来进行 近似读音 匹配。

***模糊性***

模糊匹配是指对待相似的两个词语，会被认为是同一个词。

> 在1965年，Vladimir Levenshtein 开发出了 Levenshtein distance， 用来度量从一个单词转换到另一个单词需要多少次单字符编辑。他提出了三种类型的单字符编辑：
一个字符 替换 另一个字符： _f_ox → _b_ox
插入 一个新的字符：sic → sic_k_
删除 一个字符：b_l_ack → back

Elasticsearch 指定了 fuzziness 参数支持对最大编辑距离的配置，默认为 ２；80% 的拼写错误可以对原始字符串用 `单次编辑`进行修正, 所以把最大 fuzziness 设置为 1 ，你可以得到更好的结果和更好的性能。

##### 5.1 模糊查询（fuzzy）

fuzzy 查询是 term 查询的模糊等价。 也许你很少直接使用它，但是理解它是如何工作的，可以帮助你在更高级别的 match 查询中使用模糊性。

```
# 测试数据
POST /my_index/_bulk
{ "index": { "_id": 1 }}
{ "text": "Surprise me!"}
{ "index": { "_id": 2 }}
{ "text": "That was surprising."}
{ "index": { "_id": 3 }}
{ "text": "I wasn't surprised."}

# fuzzy 查询是一个词项级别的查询，所以它不做任何分析。

GET /my_index/_search
{
  "query": {
    "fuzzy": {
      "text": "surprize"
    }
  }
}

{
  "took" : 169,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.9559981,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.9559981,
        "_source" : {
          "text" : "Surprise me!"
        }
      },
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.69983494,
        "_source" : {
          "text" : "I wasn't surprised."
        }
      }
    ]
  }
}

# 更好的参数
GET /my_index/_search
{
  "query": {
    "fuzzy": {
      "text": {
        "value": "surprize",
        "fuzziness": 1
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.9559981,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.9559981,
        "_source" : {
          "text" : "Surprise me!"
        }
      }
    ]
  }
}

```

***优化***

fuzzy 查询的工作原理是给定原始词项及构造一个 编辑自动机— 像表示所有原始字符串指定编辑距离的字符串的一个大图表。

模糊查询使用这个自动机依次高效遍历词典中的所有词项以确定是否匹配。 一旦收集了词典中存在的所有匹配项，就可以计算匹配文档列表。

一个编辑距离 2 的模糊查询能够匹配一个非常大数量的词项同时执行效率会非常糟糕。 下面两个参数可以用来限制对性能的影响：

+	`prefix_length`
	+	不能被 “模糊化” 的初始字符数。 大部分的拼写错误发生在词的结尾，而不是词的开始。 例如通过将 prefix_length 设置为 3 ，你可能够显著降低匹配的词项数量。
+	`max_expansions`
	+	如果一个模糊查询扩展了三个或四个模糊选项，这些新的模糊选项也许是有意义的。如果它产生 1000 个模糊选项，那么就基本没有意义了。 设置 max_expansions 用来限制将产生的模糊选项的总数量。模糊查询将收集匹配词项直到达到 max_expansions 的限制。


##### 5.2 模糊匹配查询

match 查询支持开箱即用的模糊匹配：

```
GET /my_index/_search
{
  "query": {
    "match": {
      "text": {
        "query":     "SURPRIZE ME!",
        "fuzziness": "AUTO",
        "operator":  "and"
      }
    }
  }
}
```


同样， multi_match 查询也支持 fuzziness ，但只有当执行查询时类型是 best_fields 或者 most_fields ：
```
GET /my_index/_search
{
  "query": {
    "multi_match": {
      "fields":  [ "text", "title" ],
      "query":     "SURPRIZE ME!",
      "fuzziness": "AUTO"
    }
  }
}
```

> match 和 multi_match 查询都支持 prefix_length 和 max_expansions 参数。

***评分***

拼写错误文档比拼写正确的相关度更高，因为错误拼写出现在更少的文档中！

> 注1：模糊匹配不应用于参与评分—​只能在有拼写错误时扩大匹配项的范围。
注2：可以通过语音匹配扩大模糊匹配的范围

### 6. 参考文章

> [归一化](https://www.elastic.co/guide/cn/elasticsearch/guide/current/token-normalization.html)
[词干提取](https://www.elastic.co/guide/cn/elasticsearch/guide/current/stemming.html)
[停用词](https://www.elastic.co/guide/cn/elasticsearch/guide/current/stopwords.html)
[同义词](https://www.elastic.co/guide/cn/elasticsearch/guide/current/synonyms.html)
[拼写错误](https://www.elastic.co/guide/cn/elasticsearch/guide/current/fuzzy-matching.html)