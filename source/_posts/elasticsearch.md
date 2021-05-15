---
title: ES（零）入门
date: 2020-07-29 18:49:31
tags: [elasticsearch]
---

ElasticSearch是一个非常好用的实时分布式搜索和分析引擎，可以帮助我们快速的处理大规模数据，也可以用于全文检索，结构化搜索以及分析等；基于Lucene开发的搜索服务器，具有分布式多用户的能力；基于Restful Web接口，能够达到实时搜索、稳定、可靠、快速、高性能、安装使用方便；同时它的横向扩展能力非常强，不需要重启服务。



### 1.安装

由于是基于Java开发，所以需要配置JDK，不同的ElasticSearch需要不同的JDK版本。我安装的是最新的 ES7（内置JDK），只需要配置即可。

##### 1.1 相关资源

[ELK官网](https://www.elastic.co/cn/downloads/elasticsearch)
[ELK官网文档](https://www.elastic.co/guide/index.html)
[ELK中文文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
[ELK-API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/transport-client.html)

注：elaticsearch不能用root账号运行,需要使用非root账号

##### 1.2 单节点配置

***ES&JDK 版本要求***

查看linux是不是64位：`getconf LONG_BIT`；

ES 7.x 不需要本地 JDK 环境支持：
+	ES 5，安装需要 JDK 8 以上
+	ES 6.5，安装需要 JDK 11 以上
+	ES 7.2.1，内置了 JDK 12

[下载地址](https://www.elastic.co/cn/downloads/elasticsearch)，通过putty的pscp工具传输Windows文件到linux

```
>pscp C:\Users\wsz\Downloads\Compressed\elasticsearch-7.8.0-linux-x86_64.tar.gz cen@192.168.223.128:/home/cen
```

***解压***

```
	~]$ tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz 
	~]$ sudo mv elasticsearch-7.8.0 /usr/local/
	~]$ cd /usr/local/
	local]$ sudo mv elasticsearch-7.8.0 elasticsearch
	local]$ cd elasticsearch/
```


***修改配置文件***
 
```
elasticsearch]$ vim config/elasticsearch.yml


	# 设置集群名称
	cluster.name: my-es
	# 设置集群-结点名称
	node.name: node-1
	# 设置数据路径
	path.data: /usr/local/elasticsearch/data
	# 设置日志路径
	path.logs: /usr/local/elasticsearch/logs
	# 设置当前的ip地址,通过指定相同网段的其他节点会加入该集群中
	network.host: 0.0.0.0
	discovery.seed_hosts: []
	cluster.initial_master_nodes: ["node-1"]
	bootstrap.memory_lock: false

	# 设置网络端口
	http.port: 9200

elasticsearch]$ mkdir data
elasticsearch]$ mkdir logs
```


***JDK版本冲突处理***

在Linux系统中，安装了JDK8，但是ES7配置了内置的JDK12,所以需要修改启动脚本，命令如下：

```
vim bin/elasticsearch

	#设置使用内置JDK
	export JAVA_HOME=/usr/local/elasticsearch/jdk
	export PATH=$JAVA_HOME/bin:$PATH

	#添加jdk判断
	if [ -x "$JAVA_HOME/bin/java" ]; then
		JAVA="/usr/local/elasticsearch/jdk/bin/java"
	else
		JAVA=`which java`
	fi
:wq
```

***错误处理***

+	Elasticsearch start error:Native controller process has stopped - no new native processes can be started;
+	[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535](用户创建文件的权限过低，至少需要65536);

相关命令：
```
	su root
	vim /etc/security/limits.conf

	sudo vim /etc/security/limits.conf

		cen soft nofile 65536
		cen hard nofile 131072
		cen soft nproc 65536
		cen hard nproc 65536

	sudo vim /etc/security/limits.d/90-nproc.conf

		cen soft nproc 4096
		
	sudo vim  /etc/sysctl.conf 

		vm.max_map_count=280530

	sudo sysctl -p

	su cen
```

***启动与关闭***

启动：
```
# 前台启动
./bin/elasticsearch
# 后台启动
./bin/elasticsearch -d

# 测试
elasticsearch]$ curl http://127.0.0.1:9200
	{
	  "name" : "node-1",
	  "cluster_name" : "my-es",
	  "cluster_uuid" : "_na_",
	  "version" : {
		"number" : "7.8.0",
		"build_flavor" : "default",
		"build_type" : "tar",
		"build_hash" : "757314695644ea9a1dc2fecd26d1a43856725e65",
		"build_date" : "2020-06-14T19:35:50.234439Z",
		"build_snapshot" : false,
		"lucene_version" : "8.5.1",
		"minimum_wire_compatibility_version" : "6.8.0",
		"minimum_index_compatibility_version" : "6.0.0-beta1"
	  },
	  "tagline" : "You Know, for Search"
	}
```


关闭：

```
# 非安全关闭
ps -aux|grep elasticsearch
kill -9 端口号

# 安全关闭
# 通过 elasticsearch-head关闭节点
```



### 2. 可视化界面

原生的ES只能通过控制台交互，体验较差。实际使用中，最好配置可视化界面，便于检测与使用。

##### 2.1 elasticsearch-head
elasticsearch-head 是用于监控Elasticsearch 状态的客户端插件，包括数据可视化、执行增删改查操作等。

安装方式
+	通过插件把它集成到es（首选，但是 Elasticsearch 5.x, 6.x, and 7.x不支持该插件了）
+	安装成一个独立webapp。

功能：
+	显示集群的拓扑,并且能够执行索引和节点级别操作
+	搜索接口能够查询集群中原始json或表格格式的检索数据
+	能够快速访问并显示集群的状态

对于Elasticsearch 5.x：不支持插件。独立运行 elasticsearch-head


***安装npm、nodejs***
[nodejs](https://nodejs.org/en/download/)


```
	pscp C:\Users\wsz\Downloads\node-v12.18.2-linux-x64.tar.xz cen@192.168.223.128:/home/cen
	tar -xvf node-v12.18.2-linux-x64.tar.xz
	mv  node-v12.18.2-linux-x64 nodejs
	sudo mv nodejs /usr/local/
	sudo vim /etc/profile

		PATH=$PATH:/usr/local/nodejs/bin
	:wq
	source /etc/profile
	npm -version
```


***安装[elasticsearch-head](https://github.com/mobz/elasticsearch-head#running-with-built-in-server)***

```
	sudo git clone git://github.com/mobz/elasticsearch-head.git
	cd elasticsearch-head
	sudo cnpm install
	sudo cnpm run start
```

***sudo: npm:找不到命令***

```
	#查看npm命令
	#sudo 执行的命令是在 /usr/bin 目录下
	which npm 
	/usr/local/nodejs/bin/npm


	#添加链接
	cd /usr/bin
	sudo ln -s /usr/local/nodejs/bin/npm
	sudo ln -s /usr/local/nodejs/bin/node
```

***nodejs下载缓慢***

```
	#自动使用国内镜像
	npm install -gd cnpm --registry=http://registry.npm.taobao.org
	sudo ln -s /usr/local/nodejs/bin/cnpm
```



***elasticsearch-head插件连接不上elasticsearch***

修改elasticsearch配置文件
```
	vim config/elasticsearch.yml 

		# 连接elasticsearch-head
		http.cors.enabled: true
		http.cors.allow-origin: "*"
	:wq

	ps -aux|grep elasticsearch
	kill -9 端口号
```


> 注：想安装npm，就必须要装node.js，

##### 2.2 kibana

es官方提供了可视化、分析及管理平台-kibana,[kibana下载地址](https://www.elastic.co/cn/downloads/kibana)


***解压***
```
	pscp  C:\Users\wsz\Downloads\Compressed\kibana-7.8.0-linux-x86_64.tar.gz cen@192.168.223.128:/home/cen

	tar -zxvf kibana-7.8.0-linux-x86_64.tar.gz
	mv kibana-7.8.0-linux-x86_64 kibana
	sudo mv kibana /usr/local/
```

***配置***

```
	cd /usr/local/kibana
	vim config/kibana.yml 

		server.port: 5601
		server.host: "0.0.0.0"
		elasticsearch.hosts: ["http://localhost:9200"]
		elasticsearch.requestTimeout: 60000
		i18n.locale: "zh-CN"
		
	:wq
```

***启动与验证***

```
	./bin/kibana 
	top

	#验证
	http://localhost:5601
```

### 3. 分词器配置

分词器 接受一个字符串作为输入，将这个字符串拆分成独立的词或 语汇单元（token） （可能会丢弃一些标点符号等字符），然后输出一个 语汇单元流（token stream）。

由于ES的分词器对中文不太友好，所以我们需要配置一个中文分词器。


##### 3.1 组成与分类

***分析器组成***

+	Character Filters（文本过滤器，去除HTML）
+	Tokenizer（按照规则切分，比如空格）
+	TokenFilter（将切分后的词进行处理，比如转成小写）

***内置分词器***

+	`whitespace`分词器：按空白字符 —— 空格、tabs、换行符等等进行简单拆分。
+	`letter `分词器：转换为小写。
+	`standard `分词器：使用 Unicode 文本分割算法来寻找单词之间的界限，并且输出所有界限之间的内容。


在中文环境中，我们可以自行[IK(中文分词器)](https://github.com/medcl/elasticsearch-analysis-ik)来处理中文。

##### 3.2 安装与配置

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



### 4. 基础入门

##### 4.1 核心概念

+	集群（cluster）
	+	相当于节点集合
+	节点（Node），相当于`数据库`
+	索引（index）
	+	分片（shards）
		+	单个索引超过节点容量，将数据进行类似表分区的操作
		+	用户透明
	+	副本（Replicas）
		+	分片的副本，为了高可用性
+	类型（type），相当于`表`
	+	ES7以后被废除了，只剩下_doc作为唯一的数据类型
+	文档（Document），相当于`行数据`
	+	JSON格式的数据
+	Field：相当于MySQL的列
+	Mapping：相当于Schema（模式）。schema就是数据库对象的集合，这个集合包含了各种对象如：表、视图、存储过程、索引等。
+	DSL：相当于数据库的SQL	



##### 4.2 逻辑分层

elasticsearch 逻辑上可分为：
+	集群层
+	索引层
+	分片层
+	存储引擎层（lucene）


***集群与节点***

+	集群(cluster)是一组具相同 `cluster.name` 的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一 个节点也可以组成一个集群。
+	节点(node)是一个运行着的Elasticsearch实例。
	+	停止节点
		+	当Elasticsearch在前台运行，可以使用 Ctrl-C 快捷键终止
		+	`curl -XPOST http://localhost:9200/_shutdown`


***索引(indices)***

+	一个索引(index)就像是传统关系数据库中的数据库
+	索引（动词「索引一个文档」）表示把一个文档存储到索引（名词）里，以便它可 以被检索或者查询。
+	传统数据库为特定列增加一个索引，例如B-Tree索引来加速检索。 Elasticsearch和Lucene使用一种叫做倒排索引(inverted index)的数据结构来达到相同目的.


***分片***
（空）

***对象与文档***

程序中大多的实体或对象能够被序列化为包含键值对的JSON对象：
+	键(key)是字段(field)或属 性(property)的名字，
+	值(value)可以是字符串、数字、布尔类型、另一个对象、值数组或者 其他特殊类型，比如表示日期的字符串或者表示地理位置的对象

对象(Object)是一个JSON结构体——类似于哈希、hashmap、字典或者关联数组；对象 (Object)中还可能包含其他对象(Object)。 在Elasticsearch中，文档(document)这个术语有着 特殊含义。它特指最顶层结构或者根对象(root object)序列化成的JSON数据（以唯一ID标识 并存储于Elasticsearch中）。

Document（文档）结构如下：
+	元数据（metadata）
	+	_index(逻辑上的命名空间):文档在哪存放
		+	在Elasticsearch中，每一个字段的数据都是默认被索引的。也就是说，每个字段专门有一个`反向索引`用于快速检索。
	+	~_type:文档表示的对象类别~
	+	_id:文档唯一标识;定义或ES生成
		+	自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally unique identifiers, 或者叫 UUIDs
+	数据


##### 4.3 字段的数据类型

`_index` 类似于传统的数据库，而ES基于`倒排索引`对数据的每一个字段配置了索引 。为了便于的检索，在ES7.X中已经去除了`_type(类似于表)`的概念。但数据的每一个字段还是存在一个`TYPE`，可以通过配置优化搜索。


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


***确切值(Exact values) vs. 全文文本(Full text)***

Elasticsearch中的数据可以大致分为两种类型：

1. 确切值
+	确切值是确定的，正如它的名字一样。比如一个date或用户ID，也可以包含更多的字符串比 如username或email地址。
+	确切值查询要么匹配要么不匹配
	
2. 全文文本
+	全文文本，从另一个角度来说是文本化的数据(常常以人类的语言书写)，比如一篇推文 (Twitter的文章)或邮件正文。全文文本常常被称为 `非结构化数据`。
+	查询全文文本，需要先分词再建立倒排索引，然后基于查询条件衡量`匹配程度（相关性）`



##### 4.4 ES7 与 ES6 的区别
1. es6时，官方就提到了es7会删除type，并且es6时已经规定每一个index只能有一个type。在es7中使用默认的_doc作为type，官方说在8.x版本会彻底移除type。
2. api请求方式也发送变化，对索引的文档进行操作的时候，默认使用的Type是 _doc。例如：获取索引`GET index/_doc/id`,其中index和id为具体的值
3. 默认节点名称为主机名，默认分片数为1，不再是5
4. 使用Weak-AND算法优化查询算法
5. 为提升性能默认不在支持全文检索

***新增功能***

1. 新增应用程序主动检测功能，搭配对应版本的kibana，用户可监测应用服务的健康状态
2. 新增间隔查询（Intervals Queries），用户可设置多字符串在文档中出现的先后顺序进行检索
3. 自带jdk
4. JVM引入了新的circuit breaker（熔断）机制 




##### 4.5 交互方式

+	Java API
	+	两个Java客户端都通过9300端口与集群交互，使用Elasticsearch传输协议(Elasticsearch Transport Protocol)。集群中的节点之间也通过9300端口进行通信。如果此端口未开放，你的 节点将不能组成集群。
	+	节点客户端（Node client）：节点客户端以无数据节点(none data node)身份加入集群，换言之，它自己不存储任何数据， 但是它知道数据在集群中的具体位置，并且能够直接转发请求到对应的节点上。
	+	传输客户端（Transport client）：不加入集群，只是简单转发请求 给集群中的节点。


+	REST API(JSON，HTTP)
	+	```
		curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
		
			VERB 适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。
			PROTOCOL http 或者 https
			HOST Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。
			PORT 端口
			PATH API 的终端路径
			QUERY_STRING 任意可选的查询字符串参数
			BODY JSON 格式的请求体 
	```

> 注1：Elasticsearch7.x及以上的版本不使用TYpe的原因：ES使用的倒排索引比关系型数据库的B-Tree索引快，倒排索引的生成是基于 Index，而去除Type是为了更好的构建倒排索引。
注2：Java客户端所在的Elasticsearch版本必须与集群中其他节点一致，否则，它们可能互相 无法识别。

 
##### 4.6 特性

***面向文档***

应用中的对象很少只是简单的键值列表，更多时候它拥有复杂的数据结构，比如包含日期、 地理位置、另一个对象或者数组。Elasticsearch不仅仅是存储，还会索引(index)每个文档的内容使之可以被搜索；可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。

ELasticsearch使用Javascript对象符号(JavaScript Object Notation)，也就是JSON，作为 文档序列化格式。
```
	{
	  "_index": "index",
	  "_type": "_doc",
	  "_id": "222",
	  "_version": 1,
	  "_seq_no": 0,
	  "_primary_term": 1,
	  "found": true,
	  "_source": {
		"text": "阿伯蛇加速跳"
	  }
	}
```


##### 4.7 curl

我们可以通过Linux的curl命令来使用 `elasticsearch`。
+	curl是一个利用URL规则在命令行下工作的文件传输工具，可以说是一款很强大的http命令行工具。它支持文件的上传和下载，是综合传输工具，但按传统，习惯称url为下载工具。


***语法***
`# curl [option] [url]`

***常见参数***

```
-A/--user-agent <string>              设置用户代理发送给服务器
-b/--cookie <name=string/file>    cookie字符串或文件读取位置
-c/--cookie-jar <file>                    操作结束后把cookie写入到这个文件中
-C/--continue-at <offset>            断点续转
-D/--dump-header <file>              把header信息写入到该文件中
-e/--referer                                  来源网址
-f/--fail                                          连接失败时不显示http错误
-o/--output                                  把输出写到该文件中
-O/--remote-name                      把输出写到该文件中，保留远程文件的文件名
-r/--range <range>                      检索来自HTTP/1.1或FTP服务器字节范围
-s/--silent                                    静音模式。不输出任何东西
-T/--upload-file <file>                  上传文件
-u/--user <user[:password]>      设置服务器的用户和密码
-w/--write-out [format]                什么输出完成后
-x/--proxy <host[:port]>              在给定的端口上使用HTTP代理
-#/--progress-bar                        进度条显示当前的传送状态
```

***使用***

```
curl -XPOST "http://127.0.0.1:9200/_reindex?pretty" -H "Content-Type:application/json" -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}'

```

 

