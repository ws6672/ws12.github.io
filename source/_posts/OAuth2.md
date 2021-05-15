---
title: OAuth2
date: 2020-03-26 14:02:12
tags: [微信小程序]
---

OAuth是用于授权的行业标准协议,OAuth 2.0致力于简化客户端开发人员的工作，同时为Web应用程序，桌面应用程序，移动电话和客厅设备提供特定的授权流程。该规范及其扩展正在IETF OAuth工作组内开发。目前的吧去那边是2.0，即 OAuth2。 

### 应用场景
很多网站都支持第三方登陆，为了使用这个服务，需要用户进行授权。传统的授权方法是通过输入用户的账号以及密码，这样的做法有以下几个严重的缺点：
+	该网站会保存用户的密码，这样很不安全；用户没法限制"云冲印"获得授权的范围和有效期。
+	用户如果修改密码，会使得其他所有获得用户授权的第三方应用程序全部失效。
+	只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有被密码保护的数据泄漏。

### OAuth Scopes
`Scopes`是OAuth 2.0中的一种机制，用于限制应用程序对用户帐户的访问。应用程序可以请求一个或多个范围，然后在同意屏幕中将此信息呈现给用户，并且颁发给该应用程序的访问令牌将限于所授予的范围。OAuth没有为`Scopes`定义任何特定的值，因为它高度依赖于服务的内部体系结构和需求。

### OAuth2术语
+	Third-party application：第三方应用程序，本文中又称"客户端"（client），即上一节例子中的"云冲印"。
+	HTTP service：HTTP服务提供商，本文中简称"服务提供商"，即上一节例子中的Google。
+	Resource Owner：资源所有者，本文中又称"用户"（user）。
+	User Agent：用户代理，本文中就是指浏览器。
+	Authorization server：授权服务器，即服务提供商专门用来处理授权、颁发访问令牌的服务器。
+	Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。


### OAuth授权类型
OAuth Grant Types 即OAuth授权类型，包括以下几种类型：
+	授权码(Authorization Code)
	+	拓展：PKCE（代码交换认证密匙）
+	(Client Credentials)
+	(Device Code)
+	(Refresh Token)
+	(Implicit Flow)
+	(Password Grant)


***授权码***
+	通过使用授权服务器获得授权码
+	作为客户端和资源所有者之间的中介，取消直接从资源所有者（用户）获取，客户端向资源所有者请求授权，将会定向到授权服务器（通过其[ RFC2616 ]中定义的用户代理，该代理继而指导资源所有者将授权码返回给客户端。

请求参数：

grant_type=client_credentials - 
&client_id=xxxxxxxxxx 
&client_secret=xxxxxxxxxx
&scope=xxx

***PKCE***
PKCE（RFC 7636）是对授权码流的扩展，以防止某些攻击并能够安全地从公共客户端执行OAuth交换。它主要由移动和JavaScript应用程序使用，但是该技术也可以应用于任何客户端。
该技术会在客户端生成一个密匙，然后当通过授权码获取接入令牌（Access Token）的时候，会再次使用该密匙。

1.启动代码验证
当原生应用程序开始授权请求时，客户端首先创建所谓的“代码验证程序” ，而不是立即启动浏览器。这是使用随机字符串A-Z，a-z，0-9和标点字符-._~（连字符，期间，下划线和波浪线）来进行加密，长字符43和128之间。

2.授权请求
参数如下：
```
response_type = code - 表示您的服务器希望收到授权代码
client_id = - 客户端ID
redirect_uri = - 表示授权完成后返回用户的URL，例如org.example.app://redirect
state = 1234zyx - （初始字符串）应用程序生成的随机字符串，稍后您将验证该字符串
code_challenge = XXXXXXXXX - （加密后的字符串）如前所述生成的代码验证，
code_challenge_method = S256 - plain或者S256，取决于验证是普通验证者字符串还是字符串的SHA256哈希。如果省略此参数，则服务器将采用plain。
```
3.返回授权码
请求通过后，授权服务器应识别code_challenge请求中的参数，并将其与其生成的授权码相关联。将其与授权码一起存储在数据库中。
如果授权服务器要求公共客户端使用PKCE，并且授权请求缺少代码验证，则服务器应返回错误响应，error=invalid_request并且应该使用error_description或者error_uri解释错误的性质。

3.1错误响应
如果授权服务器要求公共客户端使用PKCE，并且授权请求缺少代码验证，则服务器应返回错误响应，error=invalid_request并且应该使用error_description或者error_uri解释错误的性质。

4.授权码交换
```
grant_type = authorization_code - 表示此令牌请求的授权类型
code - 服务端返回的授权码
redirect_uri - 初始授权请求中使用的重定向URL
client_id - 应用程序注册的客户端ID
code_verifier - 代码验证程序（code_challenge）。
```
5.服务端响应
如果验证程序与期望值匹配，则服务器可以正常继续，发出访问令牌并进行适当响应。如果出现问题，则服务器会响应invalid_grant错误。




***客户端模式（Client Credentials）***

***设备码***
***刷新令牌***


+	~旧版：隐式流~
+	~旧版：密码授予~


> [Which OAuth 2.0 Flow Should I Use?](https://auth0.com/docs/api-auth/which-oauth-flow-to-use)
[OAuth教程–使用PKCE保护移动应用程序](https://www.dazhuanlan.com/2019/12/24/5e01e8e3c136f/)