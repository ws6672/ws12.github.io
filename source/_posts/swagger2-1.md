---
title: swagger2 通过OpenAPI文件生成JAVA代码
date: 2021-06-20 23:25:38
tags: [springboot]
---


# 一、使用Swagger－codegen生成代码(推荐)

Swagger CodeGen是一个REST 客户端生成工具，它可以从Open API的规范定义文件中生成对应的REST Client代码。推荐的OpenAPI 文档名字通常为openapi.json 或者 openapi.yaml，所以这个工具支持json或者是yaml后缀的文件。可以在官方下载相应[工具包](https://repo1.maven.org/maven2/io/swagger/swagger-codegen-cli/2.4.20/swagger-codegen-cli-2.4.20.jar)， 使用[官方例子](https://petstore.swagger.io/v2/swagger.json) 进行测试。


例如：
```java
java -jar swagger-codegen-cli-3.0.26.jar generate -i D:\cos.yaml --api-package com.simtek.cosimcloud.openapi.api --model-package com.simtek.cosimcloud.openapi.model --invoker-package com.simtek.cosimcloud.openapi.invoker --group-id com.simtek --artifact-id cosimCloud --artifact-version 0.0.1-SNAPSHOT -l spring --library spring-mvc -o cos
```

+	-i 表示openapi文件地址
+	–api-package, –model-package, –invoker-package 指定了生成类的路径
+	–group-id, –artifact-id, –artifact-version 指定生成的maven 项目的属性
+	-l 指明生成的代码编程语言
+	–library 指定了实际的实现框架
+	-o 输出文件跟目录

Swagger Codegen 支持如下的Java 库：

+	jersey1 – Jersey1+Jackson
+	jersey2 – Jersey2+Jackson
+	feign – OpenFeign+Jackson
+	okhttp-gson – OkHttp+Gson
+	retrofit (Obsolete) – Retrofit1/OkHttp+Gson
+	retrofit2 – Retrofit2/OkHttp+Gson
+	rest-template – Spring RestTemplate+Jackson
+	rest-easy – Resteasy+Jackson

如果 library选择的是`spring-mvc`，那么生成的包是swagger，可以直接复制到项目中去。

# 二、使用 OpenAPI Generator 生成 REST 客户端

OpenAPI Generator 是 Swagger Codegen 的一个分支，能够从任何 OpenAPI 规范 2.0/3.x 文档生成多个客户端。Swagger Codegen 由 SmartBear 维护，而 OpenAPI Generator 由一个社区维护，该社区包括 Swagger Codegen 的 40 多位顶级贡献者和模板创建者作为创始团队成员。

1. 在linux安装

```
npm install @openapitools/openapi-generator-cli -g
wget https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli/4.2.3/openapi-generator-cli-4.2.3.jar \
  -O openapi-generator-cli.jar

```


2. 在windows安装

下载[相关JARb包](https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli/);
OpenAPI Generator 的选项与 Swagger Codegen 的选项几乎相同；最明显的区别是将-l语言标志替换为-g生成器标志，它将生成客户端的语言作为参数；
```
java -jar openapi-generator-cli.jar generate -i D:\OpenAPI.yaml --api-package com.simtek.cosimcloud.swagger.api --model-package com.simtek.cosimcloud.swagger.model --invoker-package com.simtek.cosimcloud.swagger.invoker --group-id com.simtek --artifact-id cosimCloud --artifact-version 0.0.1-SNAPSHOT -g java -p java8=true --library resttemplate -o cosimCloud
  
-- 列出相关命令：
java -jar openapi-generator-cli.jar config-help -g java

```

支持的库如下所示：
+	jersey1 – Jersey1 +Jackson
+	jersey2 – Jersey2 +Jackson
+	feign – OpenFeign +Jackson
+	okhttp-gson – OkHttp +Gson
+	retrofit (Obsolete) – Retrofit1/OkHttp +Gson
+	retrofit2 – Retrofit2/OkHttp +Gson
+	resttemplate – Spring RestTemplate +Jackson
+	webclient – Spring 5 WebClient +Jackson (OpenAPI Generator only)
+	resteasy – Resteasy +Jackson
+	vertx – VertX +Jackson
+	google-api-client – Google API Client +Jackson
+	rest-assured – rest-assured +Jackson/Gson (Java 8 only)
+	native – Java native HttpClient +Jackson (Java 11 only; OpenAPI Generator only)
+	microprofile – Microprofile client +Jackson (OpenAPI Generator only)

3. 生成Spring Boot项目

添加maven依赖
```
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>spring-swagger-codegen-api-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```




# 导读
> [Generate Spring Boot REST Client with Swagger](https://www.baeldung.com/spring-boot-rest-client-swagger-codegen)
