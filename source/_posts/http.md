---
title: Http 协议
date: 2021-01-09 00:06:22
tags: [http]
---

### 一、概述

超文本传输协议（英文：HyperText Transfer Protocol，缩写：HTTP）是一种用于分布式、协作式和超媒体信息系统的应用层协议。HTTP是万维网的数据通信的基础。

HTTP 协议的主要特点可概括如下：
+	支持客户/服务器模式。
+	简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有 GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因而通信速度很快。
+	灵活：HTTP 允许传输任意类型的数据对象（传输的类型由 Content-Type 加以标记）
+	无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
+	无状态：HTTP 协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。


HTTP 消息由客户端到服务器的请求和服务器到客户端的响应组成。请求消息和响应消息都是由开始行（对于请求消息，开始行就是请求行，对于响应消息，开始行就是状态行），消息报头（可选），空行（只 有 CRLF 的行），消息正文（可选）组成。


### 二、URL定义

HTTP URL (URL 是一种特殊类型的 URI，包含了用于查找某个资源的足够的信息)的格式如下：
```
http://host[:port][abs_path]
```

http各部分含义：
+	http 表示要通过 HTTP 协议来定位网络资源
+	host 表示合法的 Internet 主机域名或者 IP 地址
+	port 指定一个端口号，为空则使用缺省端口 80
+	abs_path 指定请求资源的 URI，如果不存在则为“/”

### 三、请求

http 请求由三部分组成，分别是：

1. 请求行

请求行以一个方法符号开头，以空格分开，后面跟着请求的 URI 和协议的版本，格式如下：`Method Request-URI HTTP-Version CRLF`。

+	Method	请求方法
	+	GET	请求获取 Request-URI 所标识的资源
	+	POST	在 Request-URI 所标识的资源后附加新的数据
	+	HEAD	请求获取由 Request-URI 所标识的资源的响应消息报头
	+	PUT	请求服务器存储一个资源，并用 Request-URI 作为其标识
	+	DELETE	请求服务器删除 Request-URI 所标识的资源
	+	TRACE	请求服务器回送收到的请求信息，主要用于测试或诊断
	+	CONNECT	保留将来使用
	+	OPTIONS	请求查询服务器的性能，或者查询与资源相关的选项和需求

+	Request-URI	统一资源标识符
+	HTTP-Version 表示请求的HTTP 协议版本
+	CRLF 回车和换行（除了作为结尾的 CRLF 外，不允许出现单独的 CR 或 LF 字符）。


例如：
```
GET /form.html HTTP/1.1 (CRLF)
```

2. 消息报头
3. 请求正文


### 四、响应

在接收和解释请求消息后，服务器返回一个 HTTP 响应消息。HTTP 响应也是由三个部分组成，分别如下：

1.	状态行

状态行格式如下：
```
HTTP-Version Status-Code Reason-Phrase CRLF

-- HTTP-Version 表示服务器 HTTP 协议的版本
-- Status-Code 表示服务器发回的响应状态代码
-- Reason-Phrase 表示状态代码的文本描述
```

状态代码有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：

+	1xx：指示信息--表示请求已接收，继续处理
+	2xx：成功--表示请求已被成功接收、理解、接受
+	3xx：重定向--要完成请求必须进行更进一步的操作
+	4xx：客户端错误--请求有语法错误或请求无法实现
+	5xx：服务器端错误--服务器未能实现合法的请求


常见状态代码、状态描述、说明：

+	200 OK //客户端请求成功
+	400 Bad Request //客户端请求有语法错误，不能被服务器所理解
+	401 Unauthorized // 请 求 未 经 授 权 ， 这 个 状 态 代 码 必 须 和 WWW-Authenticate 报
+	//头域一起使用
+	403 Forbidden //服务器收到请求，但是拒绝提供服务
+	404 Not Found //请求资源不存在，eg：输入了错误的 URL
+	500 Internal Server Error //服务器发生不可预期的错误
+	503 Server Unavailable // 服 务 器 当 前 不 能 处 理 客 户 端 的 请 求 ， 一 段 时 间 后 ，


2.	消息报头
3.	响应正文




### 五、消息报头

HTTP 消息报头包括普通报头、请求报头、响应报头、实体报头。
每一个报头域都是由名字+“：”+空格+值 组成，消息报头域的名字是大小写无关的

1. 普通报头

在普通报头中，有少数报头域用于所有的请求和响应消息，但并不用于被传输的实体，只用于传输的消息.

+	Cache-Control 用于指定缓存指令，缓存指令是单向的（响应中出现的缓存指令在请求中未必会出现），且是独立的（一个消息的缓存指令不会影响另一个消息处理的缓存机制），HTTP1.0 使用的类似的报头域为 Pragma。请求时的缓存指令包括：no-cache（用于指示请求或响应消息不能缓存）、no-store、max-age、max-stale、min-fresh、only-if-cached;响 应 时 的 缓 存 指 令 包 括 ：public、 private、no-cache、 no-store 、no-transform、must-revalidate、proxy-revalidate、max-age、s-maxage.
	+	例如：```
		response.sehHeader("Cache-Control","no-cache");
		//response.setHeader("Pragma","no-cache");
		```
+	Date 普通报头域表示消息产生的日期和时间。
+	Connection 普通报头域允许发送指定连接的选项。例如指定连接是连续，或者指定“close”选项，通知服务器，在响应完成后，关闭连接。


2. 请求报头

请求报头允许客户端向服务器端传递请求的附加信息以及客户端自身的信息。

+	Accept 请求报头域用于指定客户端接受哪些类型的信息。eg：`Accept：image/gif`，表明客户端希望接受 GIF 图象格式的资源；`Accept：text/html`，表明客户端希望接受 html 文本
+	Accept-Charset 请 求 报 头 域 用 于 指 定 客 户 端 接 受 的 字 符 集 。 eg ：`Accept-Charset:iso-8859-1,gb2312`.如果在请求消息中没有设置这个域，缺省是任何字符集都可
以接受。
+	Accept-Encoding 请求报头域类似于 Accept，但是它是用于指定可接受的内容编码。eg：`Accept-Encoding:gzip.deflate`.如果请求消息中没有设置这个域服务器假定客户端对各种内容编码都可以接受。
+	Accept-Language 请 求 报 头 域 类 似 于 Accept ， 但 是 它 是 用 于 指 定 一 种 自 然 语 言 。 eg ：`Accept-Language:zh-cn`.如果请求消息中没有设置这个报头域，服务器假定客户端对各种语言都可以接受。
+	Authorization 请求报头域主要用于证明客户端有权查看某个资源。当浏览器访问一个页面时，如果收到服务器的响应代码为 401（未授权），可以发送一个包含 Authorization 请求报头域的请求，要求服务器对其进行验证。
+	Host 请求报头域主要用于指定被请求资源的 Internet 主机和端口号，它通常从 HTTP URL 中提取
出来的，eg：`Host：www.guet.edu.cn`
+	User-Agent 请求报头域允许客户端将它的操作系统、浏览器和其它属性告诉服务器


3. 响应报头
响应报头允许服务器传递不能放在状态行中的附加响应信息，以及关于服务器的信息和对Request-URI 所标识的资源进行下一步访问的信息。

+	Location 响应报头域用于重定向接受者到一个新的位置。Location 响应报头域常用在更换域名的
时候。
+	Server 响应报头域包含了服务器用来处理请求的软件信息。与 User-Agent 请求报头域是相对应
的。eg:`Server：Apache-Coyote/1.1`
+	WWW-Authenticate 响应报头域必须被包含在 401（未授权的）响应消息中，客户端收到 401 响应消息时候，并发送 Authorization 报头域请求服务器对其进行验证时，服务端响应报头就包含该报头域。


4. 实体报头（正文）

请求和响应消息都可以传送一个实体。一个实体由实体报头域和实体正文组成，但并不是说实体报头域和实体正文要在一起发送，可以只发送实体报头域。实体报头定义了关于实体正文。

+	Content-Encoding 实体报头域被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文的附加内容的编码，因而要获得 Content-Type 报头域中所引用的媒体类型，必须采用相应的解码机制。Content-Encoding 这样用于记录文档的压缩方法，eg：`Content-Encoding：gzip`
+	Content-Language 实体报头域描述了资源所用的自然语言。没有设置该域则认为实体内容将提供给所有的语言阅读者。eg：`Content-Language:da`
+	Content-Length 实体报头域用于指明实体正文的长度，以字节方式存储的十进制数字来表示。
+	Content-Type 实体报头域用语指明发送给接收者的实体正文的媒体类型。
+	Last-Modified 实体报头域用于指示资源的最后修改日期和时间。
+	Expires 实体报头域给出响应过期的日期和时间。为了让代理服务器或浏览器在一段时间以后更新缓存中(再次访问曾访问过的页面时，直接从缓存中加载，缩短响应时间和降低服务器负载)的页面，我们可以使用 Expires 实体报头域指定页面过期的时间。eg：`Expires：Thu，15 Sep 2006 16:23:12 GMT`

