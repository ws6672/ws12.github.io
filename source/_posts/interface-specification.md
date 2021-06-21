---
title: 接口对接规范
date: 2021-06-15 16:45:06
tags:
---


# 一、前后端正常交互的流程

1、评审阶段：产品召集前后端进行需求评审，前后端各自捋清楚自己的业务量以及联调之间工作量，从而进行开发时间评估。

2、开发准备阶段：前后端一起商量需求中需要联调的部分，进行接口的口头协议交流。

3、接口定义阶段：前后端中的一方根据之前的口头协议拟定出一份详细的接口，并书写API文档，完成后由另一方确认。有疑问的地方重新商量直至双方都没有问题。

注意：第一份确认并书写好API的接口基本不会大改。

4、开发阶段：双方根据协商出来的接口为基础进行开发，如在开发过程中发现需要新增或删除一些字段，重复步骤3。

注意：前端在开发过程中记得跟进接口，mock数据进行本地测试。

5、联调阶段：双方独自的工作完成，开始前后端联调，如在联调过程发现有疑问，重复步骤3，直至联调完成。

6、产品体验阶段：将完成的需求交给产品，让其体验，直至产品这边没有问题

7、提测阶段：将完成的需求提给测试人员，让其对该需求进行测试，如发现问题，及时通知开发并让其修改，直至需求没有bug。

8、评审单发布阶段：前后端中的一人进行评审单的拟定，发送给对应的领导，表明需求发布的程序，包括影响到的页面及业务，发布的流程，发布的回滚方案等。

9、发布阶段：前后端双方在保证步骤1-8都没有问题了，进行各自的代码发布，完成后由测试人员在线上进行相应的测试，如果有bug，重复步骤7和9，直至需求成功上线。



# 二、后端接口设计思路

1. 接口类型如何定义

按接口类型整理接口文档：
+	资源接口: 系统涉及到哪些资源, 按照 RESTful 方式定义的细粒度接口
+	操作接口: 页面涉及到哪些操作, 例如修改购物车中商品的数量, 更换优惠券等等, 也可以使用 RESTful 方式来定义
+	页面接口: 页面涉及到太多接口, 如果是一个个地调用, 会需要很多次请求, 有可以影响到前端的性能和用户感知(特别是首屏的体验), 因此可能需要将这些接口的数据合并到一起, 作成一个聚合型接口提供给前端来使用


2. 响应设计

返回的响应体类型推荐为 `Content-Type: application/json; charset=utf-8`, 返回的数据包含在 HTTP 响应体中, 是一个 JSON Object. 该 Object 可能包含 3 个字段:
+	data：业务数据
	+	任意 JSON 数据类型(number/string/boolean/object/array)，推荐始终返回一个 object (即再包一层)以便于扩展字段.
	+	例如: 用户数据应该返回 {"user":{"name":"test"}}, 而不是直接为 {"name":"test"}
+	status：状态码
	+	必须是 >= 0 的 JSON Number 整数.
		+	0 表示请求处理成功, 此时可以省略 status 字段, 省略时和为 0 时表示同一含义.
		+	非 0 表示发生错误时的错误码, 此时可以省略 data 字段, 并视情况输出 statusInfo 字段作为补充信息
+	statusInfo：状态信息，任意 JSON 数据类型.
	+	 object 包含 message 和 detail 字段
		+	message 字段作为接口处理失败时, 给予用户的友好的提示信息, 即所有给用户的提示信息都统一由后端来处理.
		+	detail 字段用来放置接口处理失败时的详细错误信息. 只是为了方便排查错误, 前端无需使用

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": {},
    "status": 0,
    "statusInfo": {
        "message": "给用户的提示信息",
        "detail": "用于排查错误的详细错误信息"
    }
}
```

3. 响应中状态码的定义

对于错误码的规范, 参考行业实践, 大致有两种方案

做显性的类型区分, 快速定位错误的类别, 例如通过字母划分类型: A101, B131
+	Standard ISO Response Codes
固定位数, 设定区间(例如手机号码, 身份证号码)来划分不同的错误类型
+	HTTP Status Code Definitions
+	System Error Codes

4. 响应码示例

```java
package com.simtek.cosimcloud.tool;

/**
 * Http状态码
 */
public enum HttpStatus {
//   成功
    OK(200, "OK"),
    SERVER_ERROR(1000, "服务器错误"),

//  参数错误：1001-1999
    PARAMETER_INVALID(1001, "参数无效"),
    PARAMETER_EMPTY(1002, "参数为空"),
    PARAMETER_TYPE_ERROR(1003, "参数类型错误"),
    PARAMETER_MISSING(1004, "参数缺失"),
    PARAMETER_BAD_CREDENTIALS(1005, "加密错误"),

//  用户错误：2001-2999
    USER_NOT_FOUND(2001, "用户名不存在"),
    USER_LOCKED(2002, "账号被停用"),
    USER_ALREADY_EXISTS(2003, "用户已存在"),
    USER_INCORRECT_PASSWORD(2004, "账号不存在或密码错误"),

//    业务错误：3001-3999
    PROFESSION_ERROR(3001, "业务错误"),

//    系统错误：4001-4999
    SYSTEM_ERROR(4001, "系统繁忙，请稍后重试"),

//    数据错误：5001-5999
    DATA_NOT_FOUND(5001, "数据未找到"),
    DATA_ERROR(5002, "数据有误"),
    DATA_EXSITED(5003, "数据已存在"),
    DATA_SEARCH_ERROR(5004, "查询出错"),

//    接口错误：6001-6999
    INTERFACE_INTERNAL_SYSTEM_CALL_EXCEPTION(6001, "内部系统接口调用异常"),
    INTERFACE_EXTERNAL_SYSTEM_CALL_EXCEPTION(6002, "外部系统接口调用异常"),
    INTERFACE_FORBIDDEN_ACCESS(6003, "接口禁止访问"),
    INTERFACE_ADDRESS_INVALID(6004, "接口地址无效"),
    INTERFACE_REQUEST_TIMED_OUT(6005, "接口请求超时"),
    INTERFACE_LOAD_IS_TOO_HIGH(6006, "接口负载过高"),

//  权限错误：7001-7999
//    JAVA Web Token
    JWT_EXPIRED(7001, "JWT 过期"),
    JWT_SIGNATURE_ERROR(7002, "JWT 错误"),
    JWT_MALFORMED(7003, "JWT 格式错误"),
    USER_FORBIDDEN(7004, "无相关权限，禁止访问"),
    UNAUTHORIZED(7005, "未进行身份认证，无访问权限");



    private  Integer code;
    private  String message;


    HttpStatus(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    public Integer getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

    private  String detail;
    public String getDetail() {
        return detail;
    }

    public HttpStatus setDetail(String detail) {
        this.detail = detail;
        return this;
    }
}

```

# 三、 RESTFUL


1. model与entity

> MVC是模式，EF是ORM，角色不同。MVC里面的Model是C发给V的。这些Model应该被高度优化，仅仅被对应的View用来显示，额外的数据应该被Model层砍掉以节省磁盘访问、内存占用或者数据库带宽。通常情况下，View的数量都会比你数据库的Entity要多，比如用户要求的各种各样的报表，所以对应的Model也应该比数据访问层的Entity多。假设View和EF的实体类完全一一对应，可以不编写额外的Model。但是随着需求的增多，很难一直使用EF的实体类来做Model。
	Entity = Database Table
	Model = Entities + Relations
	DTO = Entity - Uninteresting Fields
	View = DTOs + Relations

Entity是相对数据库而言的，是一个数据实体。而Model是MVC里面的概念，是一个用户所需的最小化视图。

2. Restful中API不同操作的区别


RESTFUL风格的请求如下：

+	GET 获取资源
+	POST 创建会话（用户登录）、创建资源、添加资源
+	PUT 创建资源、更新资源
+	DELETE 删除资源
+	PATCH 更新部分资源

POST与PUT的区别

+	在更新资源的操作上，POST 和 PUT 基本相同。在创建资源时，PUT可以指定资源路径，POST无法指定资源路径
+	PUT是幂等的操作，即重复操作不会产生变化，10次PUT 的创建请求与1次PUT 的创建请求相同，只会创建一个资源，其实后面9次的请求只是对已创建资源的更新，且更新内容与原内容相同，所以不会产生变化。
+	POST 的重复操作截然不同，10次POST请求将会创建10个资源。

以用户模块为例：
```
/user/1 put
/user post
```

PUT请求会查询ID为1的用户，如果不存在就创建；如果存在，就更新；POST请求会直接使用数据创建用户，一次请求表示创建一个用户。


Springboot中相关的注解
+	@GetMapping 
+	@PostMapping 
+	@PutMapping  
+	@DeleteMapping 
+	@PatchMapping 

```
@RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
===>
@GetMapping("/get/{id}")

```


3. RESTFUL URL

3.1 命名规则

URL命名通常有三种，驼峰命名法(serverAddress)，蛇形命名法(server_address)，脊柱命名法(server-address)。由于URL是大小写敏感的，如果用驼峰命名在输入的时候就要求区分大小写，一个是增加输入难度，另外也容易输错，报404（aB）。蛇形命名法用下划线（a_b），在输入的时候需要切换shfit，同时下划线容易被文本编辑器的下划线掩盖，支付宝用的是蛇形命名法，stackoverflow.com和github.com用的是脊柱命名法(a-b)


+	URL请求采用小写字母，数字，部分特殊符号（非制表符）组成。
+	URL请求中不采用大小写混合的驼峰命名方式，尽量采用全小写单词，如果需要连接多个单词，则采用连接符“_”连接单词

3.2 CRUD请求定义规范

第一级Pattern为模块,比如组织管理/orgz, 网格化：/grid；
第二级Pattern为资源分类或者功能请求，优先采用资源分类；如果为资源分类，则按照RESTful的方式定义第三级Pattern；定义类似如下所示：

```
/orgz/members GET 获取成员列表
/orgz/members/120 GET 获取单个成员
/orgz/members POST 创建成员
/orgz/members/120 PUT 修改成员
/orgz/members PUT 批量修改
/orgz/members/120 PATCH 修改成员的部分属性
/orgz/members/120 DELETE 删除成员
```




假设第三季

# 导读
> [前后端接口联调](https://www.zhihu.com/question/61415974/answer/187589565)
[什么时候用Model，什么时候用Entity？](https://www.zhihu.com/question/25256772)
[RESTFUL URL命名原则](https://www.pianshen.com/article/4262562009/)