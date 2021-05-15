---
title: Spring Cloud（二）配置中心
date: 2019-08-23 16:41:33
tags: [微服务]
---


在微服务项目中，如果说注册中心是管理者，那么配置中心就是户口簿。它记录着微服务体系中，各个模块的信息。
 
# 云配置

Spring Cloud Config 是一套为分布式系统中的基础设施和微服务应用提供集中化配置的管理方案，它分为服务端与客户端两个部分：
+ 服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置仓库并为客户端提供配置信息。
+ 客户端则是微服务架构中的各个微服务应用或基础设施，它们通过指定的配置中心来管理服务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。

在微服务的体系中，为了便于对多个微服务应用的配置进行管理，可以基于Spring Cloud Config构建配置中心，便于配置文件的统一修改、分发以及版本管理。

*特点*
+ 抽象映射，,通用性好。
+ 默认使用git存储，支持版本管理

> 虽然Git可以进行托管，但是在正式的开发环境中，需要放置到服务器上


*配置文件的启动顺序*
配置文件的优先规则也与常规启动应用程序相同：
+ 活动配置文件优先于默认配置文件；
+ 如果有多个配置文件，则引用最后一个配置文件。

## Spring Cloud Config支持本地存储
spring.profiles.active=native，Config Server会默认从应用的src/main/resource目录下检索配置文件。
另外也可以通过spring.cloud.config.server.native.searchLocations=file:D:/properties/属性来指定配置文件的位置



## 分布式配置中心

### 需要的包
+ spring核心
+ Spring Cloud
+ Spring Cloud Config

*使用步骤*
1. 构建项目，推荐使用IDEA的Spring initializr
2. 开启配置服务支持，@EnableConfigServer
3. 在yml添加如下配置

#### Application.yml
```
server:
  port: 9901
spring:
  application:
    name: spring-cloud-config-server
  profiles: #默认从应用的src/main/resource目录下检索配置文件
    active: native #读取本地文件
  cloud:
    config:
      server:
        native:
          search-locations: file:/E://wsz6672/bdqy/eboss/config-repo/config #重设本地文件的路径
        bootstrap: true
#        git: # 使用git的数据源
#          uri: file://gitee.com/liuge1988/spring-cloud-demo/         # 配置git仓库的地址
#          search-paths: config-repository                             # git仓库地址下的相对地址，可以配置多个，用,分割。
#          username: username                                          # git仓库的账号
#          password: password
```

> search-locations：file://${user.home}/config-repo，${user.home}可以是git仓库的地址或者是本地的地址。

入口
```
@SpringBootApplication //开启服务
@EnableConfigServer //启用配置服务器
public class ScfApplication {
    public static void main(String[] args) {
        SpringApplication.run(ScfApplication.class, args);
    }

}
```



客户端进行资源请求的规则如下
```
• /{application}/{profile}[/{label}]
• /{application}-{profile}.yml
• /{label}/{application}-{profile}.yml
• /{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

+ 其中application是配置文件名称，映射到客户端的spring.application.name；
+ profile是属性，映射到客户端的spring.active.profiles，可以用逗号分割。Profile 分为dev test  prod ，有默认、开发、测试、产品四个级别
+ label是配置的版本号，git中默认是master，如果包含特殊字符，需要用“_”分割


## 配置中心客户端
这里，不做过多介绍，专注于配置与使用。

### pom导包

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
</dependencies>
```

### Boostrap.yml


```
server:
  port: 1114

spring:
  application:
    name: zhny-base #读取配置的名称
  cloud:
    config:
       #uri: localhost:1113
      profile: dev #读取配置的版本号
      discovery:
        enabled: true #默认直接查找配置中心，配置后通过注册中心找配置中心再获取配置
        service-id: config-cloud #配置中心的名字，可以在配置中心集群时实现负载均衡
#注册到注册中心
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:1110/eureka/ #注册中心的域名
```


### 控制器

```
@RestController
public class TestController {
    @Value(value = "${common.org_id}")
    private String org_id;
    @RequestMapping("/")
    public String test () {
        if (org_id.isEmpty()) {
            return "error";
        }
        return org_id;
    }
}

```

## 属性覆盖

用途：在实际开发中，多个分布式系统可能使用同一套公有的配置文件，但有些属性的参数值是当前系统所不匹配的，通过这个配置就可以在公有配置文件的基础上进行调整。

```
spring:
  cloud:
    config:
      server:
        overrides:
          foo: bar
```

## Health指标
Config Server附带一个运行状况指示器，用于检查配置的EnvironmentRepository是否正常工作。默认情况下，它会向EnvironmentRepository询问名为app的应用程序，默认配置文件以及EnvironmentRepository实现提供的默认标签。
您可以通过设置*spring.cloud.config.server.health.enabled = false*来禁用运行状况指示器。


## Security
您可以以任何对您有意义的方式（从物理网络安全到OAuth2承载令牌）自由地保护您的Config服务器，而Spring Security和Spring Boot可以轻松完成任何操作。
要使用默认的Spring Boot配置的HTTP Basic安全性，只需在类路径中包含Spring Security（例如，通过spring-boot-starter-security）。默认值是“user”的用户名和随机生成的密码，这在实践中不会非常有用，因此我们建议您配置密码（通过security.user.password）并加密。
