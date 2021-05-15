---
title: webservice 使用（一）
date: 2019-11-27 16:08:26
tags: [javaweb]
---


# 一、什么是webservice

Web Service是一个平台独立的，低耦合的，自包含的、基于可编程的web的应用程序，可使用开放的XML（标准通用标记语言下的一个子集）标准来描述、发布、发现、协调和配置这些应用程序，用于开发分布式的互操作的应用程序。现在这种技术使用可能没有那么多了，许多较新的项目选择了使用微服务、springboot、springmvc来发布接口。但是，作为一种风靡一时的技术，还是影响着现在的技术体系。


通常【WebService】拥有一个服务端和多个客户端，服务端通过各种协议返回WSDL文件来告诉客户端它有什么可以调用的方法。客户端通过WSDL文件编写对应的服务端端口，通过RMI等技术来访问服务端以获取数据。服务提供商通过告知使用者或者注册到UDDI服务器来提供WSDL文件。

> WSDL(Web Services Description Language)是一个基于XML的语言，用于描述Web Service及其函数、参数和返回值。。它是WebService客户端和服务器端都能理解的标准格式。因为是基于XML的，所以WSDL既是机器可阅读的，又是人可阅读的。

> UDDI是一种用于描述、发现、集成Web Service的技术，它是Web Service协议栈的一个重要部分。通过UDDI，企业可以根据自己的需要动态查找并使用Web服务，也可以将自己的Web服务动态地发布到UDDI注册中心，供其他用户使用。

# 二、相关协议


RPC(Remote Process Call)是远程进程调用,不管你通过HTTP协议也要，Socket协议也罢，能够调用远程规定好的接口就可称之为RPC。

### 0. webService相关术语
webService 的组成部分是WSDL+SOAP+UDDI。
 
+	soap【简单请求协议】
	+	基于http协议使用xml语言传输数据
	+	XML +	HTTP
	+	组成：
		+	Envelope – 必须的部分。以XML的根元素出现。
		+	Headers – 可选的。
		+	Body – 必须的。在body部分，包含要执行的服务器的方法。和发送到服务器的数据。
	+	XML	【扩展性标记语言】
		+	XML，用于传输格式化的数据，是Web服务的基础。
		+	namespace-命名空间。
		+	xmlns=“http://itcast.cn” 使用默认命名空间。
		+	xmlns:itcast=“http://itcast.cn”使用指定名称的命名空间
+	wsdl【Web服务描述语言】
	+	webservice 接口描述文件，用于约定调用接口的参数
	+	组成
		+	<definitions>  描述文本，父节点
		+	<types>	描述接口的命名空间等
		+	```
			<types>
				<xsd:schema> // 定义服务
					<xsd:import namespace="http://server.webservice.csgsas.qctc.com/" schemaLocation="http://127.0.0.1:8080/zdjd_webservice_server/databaseService?xsd=1"/>
				</xsd:schema>
			</types>
		```
		+	<message>,定义发布的方法
+	UDDI【服务目录】
	+	UDDI是一种用于描述、发现、集成Web Service的技术，它是Web Service协议栈的一个重要部分。
	+	通用描述、发现与集成服务
	
		
REST 是新一代的webservice风格，组成部分是http、json。
		
+	REST
	+	一种风格，对http协议的二次诠释
	+	URL描述资源
	+	JSON交换数据
	+	http描述行为（get、post、put、delete）;
	+	HTTP状态码来表示不同的结果
	
	
### 1. SOAP webService

SOAP是一种数据交换协议规范，是一种轻量的、简单的、基于XML的协议的规范。

> WebService采用HTTP协议传输数据，采用XML格式封装数据（即XML中说明调用远程服务对象的哪个方法，传递的参数是什么，以及服务对象的返回结果是什么）。WebService通过HTTP协议发送请求和接收结果时，发送的请求内容和结果内容都采用XML格式封装，并增加了一些特定的HTTP消息头，以说明HTTP消息的内容格式，这些特定的HTTP消息头和XML内容格式就是SOAP协议(simpleobject access protocol,简单对象访问协议) 。

*1.1 特点*

+	`SOAP = HTTP + XML`
+	SOAP webService有严格的规范和标准，包括安全，事务等各个方面的内容，同时SOAP强调操作方法和操作对象的分离，有WSDL文件规范和XSD文件分别对其定义。
+	适用于对底层传输有较大要求的项目

 
### 2. RESTful webService

REST其实并不是什么协议也不是什么标准，而是一种设计风格,它将Http协议的设计初衷作了诠释.RESTful是不是标准，关注核心是要处理的资源，通过RUL触发执行。是一种比RPC（远程调用）更轻量级、更安全的服务端和客户端交互方式，基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。可有多种实现方式，如java中spring

*2.1 特点*
1. REST是一种架构风格，其核心是面向资源；而webService底层SOAP协议，主要核心是面向活动。
2. 幂等性。某个函数或接口使用相同的参数调用一次或无限次，造成影响相同，不会产生灾难性后果。
3. 安全性。对资源的访问不会导致资源状态发生变化。
4. 适用于偏向资源的、业务较为简单、安全性要求不高的项目。


*2.2 什么是RESTful风格*
RESTful诠释了Http协议，在这种风格中，约定对http不同的方法【get、put、post、delete】对应不同api接口。
```
RESTful 风格
http://127.0.0.1/user/1 GET   查
http://127.0.0.1/user  POST   增 
http://127.0.0.1/user  PUT    改
http://127.0.0.1/user  DELETE 删

# 使用【SOAP webService】时，我们需要为增删改查设置不同的接口；但使用Restful风格则不同，不同的方法不需要使用不同的接口，而是使用http协议提供的方法即可。
```

*2.3 使用*
|HTTP方法|资源操作|幂等|安全||
|:--|:--|:--|:--|
|GET|SELECT|Y|Y||
|POST|INSERT|N|N||
|PUT|UPDATE|Y|N||
|DELETE|DELETE|Y|N||

 


### 3. RESTful 实例

*3.1 代码* 
在这个例子中，是用springboot实现restful风格。
```
	@RestController
	@RequestMapping("/test")
	public class TestController {
		@RequestMapping(method = RequestMethod.POST)
		public String insert(String message) {
			return "insert:"+message;
		}
		@RequestMapping(method = RequestMethod.GET)
		public String select() {
			return  "select:11";
		}
		@RequestMapping(method = RequestMethod.PUT)
		public String update(String message) {
			return "update："+message;
		}
		@RequestMapping(method = RequestMethod.DELETE)
		public String delete(Integer id) {
			return "delete:"+id;
		}
	}
```
*3.2 测试*

```
访问地址：http://localhost:9084/test
#	通过不用的get、put、post、delete可以访问不同的方法
```
![rest-get](/image/webservice/webservice-rest-getpng.png)
 


# 参考文档
> [Web service是什么？——阮一峰](http://www.ruanyifeng.com/blog/2009/08/what_is_web_service.html)
[Rest状态码](https://www.cnblogs.com/gulei/p/9515982.html)