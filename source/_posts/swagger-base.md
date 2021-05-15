---
title: swagger(一)基础入门
date: 2019-08-20 22:42:24
tags: [javaweb]
---
# 基础
API功能的发展是不可避免的，但维护API文档的头痛并非必须如此。Swagger工具可以帮助您完成生成和维护API文档的工作，确保您的文档在API发展过程中保持最新状态。


在Swagger团队加入SmartBear并且该规范于2015年被捐赠给OpenAPI计划之后，Swagger规范被重命名为OpenAPI规范（OAS），已成为定义RESTful API的因素标准。

OAS合同描述了API的作用，它的请求参数和响应对象，所有这些都没有任何代码实现的指示。使用OAS定义的Web服务可以相互通信，而不管它们内置的语言，因为OAS是语言无关的和机器可读的。

优势：
+ 生成API文档
+ 在线对接口进行测试

## Maven包
```
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.7.0</version>
</dependency>

<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.7.0</version>
</dependency>
```

## Swagger2 的常用注解

```
- @Api()用于类； 
    表示标识这个类是swagger的资源 
- @ApiOperation()用于方法； 
    表示一个http请求的操作 
- @ApiParam()用于方法，参数，字段说明； 
    表示对参数的添加元数据（说明或是否必填等） 
- @ApiModel()用于类 
    表示对类进行说明，用于参数用实体类接收 
- @ApiModelProperty()用于方法，字段 
    表示对model属性的说明或者数据操作更改 
- @ApiIgnore()用于类，方法，方法参数 
    表示这个方法或者类被忽略 
- @ApiImplicitParam() 用于方法 
    表示单独的请求参数 
- @ApiImplicitParams() 用于方法，包含多个 @ApiImplicitParam
```
