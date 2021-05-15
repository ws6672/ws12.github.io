---
title: Spring Cloud（一）上下文服务
date: 2019-08-23 16:20:40
tags: [微服务]
---
  这一个系列是在进入新公司后，接触到微服务项目后打算开的新坑。因为我有可能要长期出差，维护另外一个项目，可能就没有什么动力学习这个了。我希望这个坑能时时引起我的注意，毕竟轻微强迫症也是强迫症。
  系列的主要内容是对一份官网的文档进行硬翻译，再夹杂一些网上的信息，都是比较基础的知识，干货少。

# Context Services
## Spring Cloud Context——应用上下文服务
 Spring Boot有一个关于如何使用Spring构建应用程序的建议：
 + 将常规配置文件放置在常规位置；
 + 使用于通用管理和监视任务的端点。
 
## Bootstrap Application Context（bootstrap.yml）
+ Spring Cloud应用程序通过创建“引导程序（bootstrap.yml）”的上下文来运行，该上下文是主应用程序的父上下文。
+ 开箱即用，它负责从外部源加载配置属性，还解密本地外部配置文件中的属性。
+ 这两个上下文共享一个环境，它是任何Spring应用程序的外部属性的来源。 
+ 高优先级，因此本地配置无法覆盖,但优先级低于 application.yml.



## 修改 bootstrap.yml (bootstrap.properties)的默认位置或者默认加载文件
通过配置系统属性：
```
spring.cloud.bootstrap.name （默认 bootstrap)
spring.cloud.bootstrap.location
```

## 自定义Bootstrap配置中心
  稍微了解一点微服务的，都知道它有个配置中心的概念，而Spring Cloud也对它做了集成，通过简单配置就可以开启配置中心。但在实际的开发中，有时候需要对它进行定制，以实现一些特殊的功能。所以，官方提供了自定义配置的样例。

默认属性资源是Spring Cloud Config Server，即配置中心。但我们也可以自定义配置中心。
> 注：Spring Cloud Config 是一种用来动态获取Git、SVN、本地的配置文件的一种工具

```
//1.实现接口PropertySourceLocator 
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {
//2.重写locate方法
    @Override
    public PropertySource<?> locate(Environment environment) {
        return new MapPropertySource("customProperty",
                Collections.<String, Object>singletonMap("property.from.sample.custom.source", "worked as intended"));
    }
}
//3.在META-INF/spring.factories下进行配置
//org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator
```


## 范围刷新
当配置发生变化时，标记为*@RefreshScope*和*@Bean*类将得到特殊处理。这就实现了有状态的bean的状态更改。

### 实际用途
一个需要连接数据库的业务，如果数据库主从切换，而在切换的时候，有业务正在进行，那么范围刷新就可以使连接延续，而不至于调用失败。
*@RefreshScope*注解没有继承性，即被注解的Bean的属性对应的bean并不会被刷新。

## Encryption and Decryption（加密与解密）
Config Client有一个Environment预处理器，用于在本地解密属性值。它遵循与Config Server相同的规则，并通过encrypt.*具有相同的外部配置。因此，您可以使用{cipher}*形式的加密值，只要有有效密钥，它们就会在主应用程序上下文获取环境之前被解密。

要在客户端中使用加密功能，您需要在类路径中包含Spring Security RSA（Maven协调“org.springframework.security:spring-security-rsa”），您还需要在JVM中使用完整的JCE扩展.

### JCE
JCE（Java Cryptography Extension）是一组包，它们提供用于加密、密钥生成和协商以及 Message Authentication Code（MAC）算法的框架和实现。
它提供对对称、不对称、块和流密码的加密支持，它还支持安全流和密封的对象。它不对外出口，用它开发完成封装后将无法调用。
+ 1.文件下载
++ JCE6:https://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html
++ JCE7:https://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html
++ JCE8:https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html
+ 2.导入文件到JDK/jre/lib/security下


## Spring Boot Actuator的端口
通过web端点可以对应用程序运行时的内部状态。Actuator提供13个端点，可以分为三大类：*配置端点*、*度量端点*和*其他端点*。

### 对应的Maven依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
### 调用工具——curl
cURL是一个利用URL语法在命令行下工作的文件传输工具，1997年首次发行。它支持文件上传和下载，所以是综合传输工具，但按传统，习惯称cURL为下载工具。cURL还包含了用于程序开发的libcurl。

```
$ curl localhost:8080/env
```