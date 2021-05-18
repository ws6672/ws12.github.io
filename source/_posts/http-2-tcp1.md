---
title: 计算机网络（二） HTTP协议概述
date: 2020-04-13 10:45:28
tags: [http]
---

超文本传输​​协议（HTTP）是一个用于传输超媒体文档（例如 HTML）的应用层协议。它是为 Web 浏览器与 Web 服务器之间的通信而设计的，但也可以用于其他目的。


HTTP 遵循经典的客户端-服务端模型，客户端打开一个连接以发出请求，然后等待它收到服务器端响应。HTTP 是无状态协议，这意味着服务器不会在两个请求之间保留任何数据（状态）。该协议虽然通常基于 TCP/IP 层，但可以在任何可靠的传输层上使用；也就是说，不像 UDP，它是一个不会静默丢失消息的协议。RUDP——作为 UDP 的可靠化升级版本——是一种合适的替代选择。

> 客户端和服务端通过交换各自的消息（与数据流正好相反）进行交互。由像浏览器这样的客户端发出的消息叫做 requests，被服务端响应的消息叫做 responses。


### 一、http协议中的角色

+	客户端/user-agent: 就是任何能够为用户发起行为的工具，一般是由浏览器扮演。浏览器*总是*作为发起请求的实体，而不是服务器。
+	Web服务端：Server只是虚拟意义上代表一个机器：它可以是共享负载（负载均衡）的一组服务器组成的计算机集群，也可以是一种复杂的软件，通过向其他计算机（如缓存，数据库服务器，电子商务服务器 ...）发起请求来获取部分或全部资源。
+	代理（Proxies）：它作为一个中介的角色，用于隐藏、缓存、过滤、负载均衡、认证、日志记录等行为。


### 二、HTTP协议的本质

+	简单的。虽然下一代HTTP/2协议将HTTP消息封装到了帧（frames）中，HTTP大体上还是被设计得简单易读。
+	可扩展的。 HTTP/1.0中的 `headers`可以用于拓展协议，只要服务端与客户端语义一致。
+	无状态，有会话的。 HTTP本质是无状态的，使用 Cookies 可以创建有状态的会话。两个请求之间是没有联系的，前一个请求下单，后一个请求付钱，但第二个请求是拿不到第一个请求的数据的，而是通过Cookies保存。

***是否需要面向连接***

HTTP并不需要其底层的传输层协议是面向连接的，只需要它是可靠的，或不丢失消息的（至少返回错误）。因为TCP是可靠的，而UDP不是。所以，HTTP依赖于面向连接的TCP，而不是UDP。但是，面向连接不是必须的。

### 三、HTTP控制什么

+	缓存。文档如何缓存能通过HTTP来控制。服务端能告诉代理和客户端哪些文档需要被缓存，缓存多久，而客户端也能够命令中间的缓存代理来忽略存储的文档。
+	开放同源限制。同源限制是指如果两个 URL 的 protocol、port (如果有指定的话)和 host 都相同的话，则这两个 URL 是同源。而HTTP可以通过修改头部来放开限制。
+	认证。可以保护网页，只让特地用户访问。使用Authenticate相似的头部即可，或用HTTP Cookies来设置指定的会话。
+	代理和隧道。通常情况下，服务器和/或客户端是处于内网的，对外网隐藏真实 IP 地址。因此 HTTP 请求就要通过代理越过这个网络屏障。
+	会话 。虽然基本的HTTP是无状态协议，但使用HTTP Cookies允许你用一个服务端的状态发起请求，这就创建了会话。



### 四、HTTP流

+	打开一个TCP连接
+	发送一个HTTP报文
	+	在http/2.0中，报文被封装在帧中，取代了http流
+	读取服务端返回的报文信息
+	关闭连接或者为后续请求重用连接。

### 五、http报文


***请求***

![http请求](/image/http/httpqq.png)

+	Method
	+	GET
		+	该GET方法请求表示指定资源。使用的请求GET应仅检索数据。
	+	HEAD
		+	该HEAD方法要求一个与GET请求相同的响应，但没有响应主体。
	+	POST
		+	该POST方法用于将实体提交给指定的资源，通常会导致状态更改或对服务器产生副作用。
	+	PUT
		+	该PUT方法用请求有效负载替换目标资源的所有当前表示形式。
	+	DELETE
		+	该DELETE方法删除指定的资源。
	+	CONNECT
		+	该CONNECT方法建立到由目标资源标识的服务器的隧道。
	+	OPTIONS
		+	该OPTIONS方法用于描述目标资源的通信选项。
	+	TRACE
		+	该TRACE方法沿到目标资源的路径执行消息环回测试。
	+	PATCH
		+	该PATCH方法用于对资源进行部分修改。
+	Path 要获取的资源的路径
	+	通常是上下文中就很明显的元素资源的URL，它没有protocol （http://），domain（developer.mozilla.org），或是TCP的port（HTTP一般在80端口）。
+	Version	协议版本号
+	headers 可选头部
+	对于一些像POST这样的方法，报文的body就包含了发送的资源，这与响应报文的body类似。

***响应***

![http响应](/image/http/httpxy.png)
响应报文包含了下面的元素：

+	HTTP协议版本号。
+	一个状态码（status code），来告知对应请求执行成功或失败，以及失败的原因。
+	一个状态信息，这个信息是非权威的状态码描述信息，可以由服务端自行设定。
+	HTTP headers，与请求头部类似。
+	可选项，比起请求报文，响应报文中更常见地包含获取的资源body。

### http缓存

重用已获取的资源能够有效的提升网站与应用的性能。Web 缓存能够减少延迟与网络阻塞，进而减少显示某个资源所用的时间。借助 HTTP 缓存，Web 站点变得更具有响应性。


缓存分类
+	私有缓存	只能用于单独用户
+	共享缓存	共享缓存存储的响应能够被多个用户使用

缓存类型
+	浏览器缓存
+	代理缓存
+	网关缓存
+	CDN缓存
+	反向代理缓存
+	负载均衡器缓存


*** 缓存目标 ***

然而常见的 HTTP 缓存只能存储 GET 响应，对于其他类型的响应则无能为力。缓存的关键主要包括request method和目标URI（一般只有GET请求才会被缓存）。

*** 缓存控制 ***

`Cache-control 头`：HTTP/1.1定义的 Cache-Control 头用来区分对缓存机制的支持情况， 请求头和响应头都支持这个属性。通过它提供的不同的值来定义缓存策略。
禁止缓存	`Cache-Control: no-store`
强制确认缓存	`Cache-Control: no-cache`
私有缓存	`Cache-Control: private`
公共缓存	`Cache-Control: public`
缓存过期	
	（请求）`Cache-Control: max-age=31536000`（max-age是距离请求发起的时间的秒数）
	（响应）Expires 响应头包含日期/时间， 即在此时候之后，响应过期。
缓存验证确认	`Cache-Control: must-revalidate`

。。。待续


### 参考文章
> [HTTP概述](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview)