---
title: springboot（一）入门
date: 10001000-08-15 14:12:58
tags: [javaweb]
---


### 1. SpringBoot 快速入门

##### 1.1 什么是 SpringBoot

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域(rapid application development)成为领导者。

> Spring 诞生时是 Java 企业版（Java Enterprise Edition，JEE，也称 J2EE）的
轻量级代替品。无需开发重量级的 Enterprise JavaBean（EJB），Spring 为企业级
Java 开发提供了一种相对简单的方法，通过依赖注入和面向切面编程，用简单的Java 对象（Plain Old Java Object，POJO）实现了 EJB 的功能。
虽然 Spring 的组件代码是轻量级的，但它的配置却是重量级的。


***Spring配置 发展三阶段***

第一阶段：xml配置
在Spring 1.x时代，使用Spring开发满眼都是xml配置的Bean，随着项目的扩大，我们需要把xml配置文件放到不同的配置文件里，那时需要频繁的在开发的类和配置文件之间进行切换

第二阶段：注解配置
在Spring 2.x 时代，随着JDK1.5带来的注解支持，Spring提供了声明Bean的注解（例如@Component、@Service），大大减少了配置量。主要使用的方式是应用的基本配置（如数据库配置）用xml，业务配置用注解

第三阶段：java配置
Spring 3.0 引入了基于 Java 的配置能力，这是一种类型安全的可重构配置方式，可以代替 XML。我们目前刚好处于这个时代，Spring4.x和Spring Boot都推荐使用Java配置。


`Spring Boot` 四个核心

+	自动配置：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置
+	起步依赖：告诉Spring Boot需要什么功能，它就能引入需要的库。
+	命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
+	执行器：让你能够深入运行中的Spring Boot应用程序，一套究竟。

`Spring Boot`比起`Spring`，最大的好处是易于使用，是开发微服务、小项目的利器。

##### 1.2 使用IDEA构建项目

使用IDEA的好处是可以通过勾选配置组件，不用担心相关包以及版本冲突，

***环境***

+	JDK8
+	IDEA 20019
+	SpringBoot 2.2.3

***使用IDEA配置***

![springboot项目建立](/iamge/springboot/init.png)
![springboot 文件](/iamge/springboot/text1.png)



***项目结构***

项目里面基本没有代码，除了几个空目录外，还包含如下几样东西。

```
com
  +- springboot
    +- src
		+- main
		|  +- java（存放代码）
		|  |
		|  |
		|  +- resources
		|  |  +- static（静态文件 图标/图片）
		|  |  +- templates（模板 HTML文件）
		|  |  +- application.yml  配置属性
		|  |  +- application.profiles  配置属性
		|
		|
		+- test（测试代码）
		+- pom.xml Maven的构建说明文件

```

***pom 配置***

+	spring-boot-starter-parent（父级依赖）：是一个特殊的首发，它用来提供相关的Maven默认依赖，使用它之后，常用的包依赖可以省去版本标签。
+	spring-boot-starter-xx（起步依赖）：Spring Boot提供了很多“开箱即用”的依赖模块，都是以spring-boot-starter-xx作为命名的

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
	// 1. 配置父级依赖
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.springboot</groupId>
    <artifactId>example</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>example</name>
    <description>Demo project for Spring Boot（20200816）</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
		// 2. 配置启动依赖
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
		// 3. 配置其它依赖
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>


    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>


```

***Application.yml 配置***

```
server:
  port: 8888
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/toolweb?serverTimezone=UTC&useSSL=false
    username: root
    password: root
```

***启动器***
```
package com.springboot.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// 该注解表示这是一个 SpringBoot启动类
@SpringBootApplication
public class ExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }

}

```




***控制器***

```
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }

}
```

***测试***

启动主程序，打开浏览器访问`http://localhost:8888/hello`，可以看到页面输出Hello World


##### YML

YAML (YAML Ain't a Markup Language)YAML不是一种标记语言，通常以.yml为后缀的文件，是一种直观的能够被电脑识别的数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，一种专门用来写配置文件的语言。可用于如： Java，C/C++, Ruby, Python, Perl, C#, PHP等。

***语法***

1. 约定格式

+	k: v 表示键值对关系，冒号后面必须有一个空格
+	使用空格的缩进表示层级关系，空格数目不重要，只要是左对齐的一列数据，都是同一个层级的
+	大小写敏感
+	缩进时不允许使用Tab键，只允许使用空格。
+	松散表示，java中对于驼峰命名法，可用原名或使用-代替驼峰，如java中的lastName属性,在yml中使用lastName或 last-name都可正确映射。


2. 键值对的书写

+	字面值
	+	字符串默认不用加上单引号或者双绰号；
	+	"": 双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
		+	"a\nb"===> a换行b
	+	''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据
+	日期
	+	date: 2020/08/15
+	对象(属性和值)、Map(键值对)
	+	```
	people:
		name: ws
		age: 1000
	people: {name:ws,age: 1000}
	```
+	数组、list、set
	+	```
	pets:
		- dog
		- pig
		- cat
	pets: [dog,pig,cat]
	```
+	数组对象、list对象、set对象
	+	```
	peoples:
    - name: ws
      age: 22
    - name: lisi
      age: 1000
    - {name: wangwu,age: 18}
	```

3. 文档块
+	对于生产、测试环境等可以使用不同的配置，可以写在一个文件中，使用`---`隔开
	+	```
	server:
	  port: 8081
	spring:
	  profiles:
		active: test #激活

	---
	server:
	  port: 8080
	spring:
	  profiles: dev #生产环境
	
	---
	server:
	  port: 8084
	spring:
	  profiles: test  #测试环境
	  
	```

### 2. SpringBoot WEB

前面，我们是使用IDEA快速搭建了一个 RESTful Service 的微服务项目，现在要快速配置一个WEB界面

##### 2.1 基础


***约定配置***

Spring Boot默认提供静态资源目录位置需置于classpath下，目录名需符合如下规则：

```
/static
/public
/resources
/META-INF/resources
```

***模板引擎***

`@RestController`返回的是JSON格式的数据，如果需要渲染页面，可以通过模板引擎实现。

Spring Boot提供了默认配置的模板引擎主要有以下几种：

+	`Thymeleaf`
+	FreeMarker
+	Velocity
+	Groovy
+	Mustache

> Spring Boot建议使用这些模板引擎，避免使用JSP，若一定要使用JSP将无法实现Spring Boot的多种特性

##### 2.2 Thymeleaf

Thymeleaf是面向Web和独立环境的现代服务器端Java模板引擎，能够处理HTML，XML，JavaScript，CSS甚至纯文本。 
Thymeleaf的主要目标是提供一个优雅和高度可维护的创建模板的方式。为了实现这一点，它建立在自然模板的概念上，将其逻辑注入到模板文件中，不会影响模板被用作设计原型。这改善了设计的沟通，弥合了设计和开发团队之间的差距。 
Thymeleaf也从一开始就设计了Web标准 (特别是HTML5)允许您创建完全验证的模板，

***引用***

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

***application.yml 配置***

```
spring: 
  thymeleaf:
    prefix: classpath:/templates/ # 前缀
    suffix: .html 	#后缀
    encoding: UTF-8 # 编码
    mode: HTML5 # 模板模式
    cache: false # 关闭模板缓存，实时更新
    content-type: text/html
```

***示例***

```
// 控制器

@Controller
public class IndexController {
    @RequestMapping("/")
    public String index(ModelMap map) {
        // 加入一个属性，用来在模板中读取
        map.addAttribute("host", "https://www.cnblogs.com");
        // 对应src/main/resources/templates/index.html
        return "index";
    }
}
---

// index.html

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>
</head>
<body>
    <h1 th:text="${host}">Hello World</h1>
</body>
</html>

```

***Thymeleaf IDEA 中语法无法识别***

在`<html>` 中添加 `xmlns:th="http://www.thymeleaf.org"`

> Thymeleaf主要以属性的方式加入到html标签中，浏览器在解析html时，当检查到没有的属性时候会忽略，所以Thymeleaf的模板可以通过浏览器直接打开展现，这样非常有利于前后端的分离。

*** Error resolving template [index], template might not exist or might not be accessible by any of the configured Template Resolvers***

可能的原因如下：
+	路径配置错误，正确的配置 `prefix: classpath:/templates/ # 前缀`
+	注解不是`@RestController`, 而是`@Controller`
+	返回值是`"/index"`,去掉斜杆

### 3. RESTful API与单元测试


***RESTful***

基于 RESTful 原则设计的 API 就是RESTful API。该原则如下：
+	一个URI代表一种资源
+	客户端和服务器之间，传递这种资源的某种表现层
+	客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。


##### 3.1 RESTful API 实例

RESTful API具体设计如下：
![RESTful API](/image/springboot/restful.png)

***实体***

```
public class User { 
 
    private Long id; 
    private String name; 
    private Integer age;  
    // 省略setter和getter 
}
```

***操作接口***

```
@RestController 
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下 
public class UserController { 
 
    // 创建线程安全的Map 
    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>()); 
 
    @RequestMapping(value="/", method=RequestMethod.GET) 
    public List<User> getUserList() { 
        // 处理"/users/"的GET请求，用来获取用户列表 
        // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递 
        List<User> r = new ArrayList<User>(users.values()); 
        return r; 
    } 
 
    @RequestMapping(value="/", method=RequestMethod.POST) 
    public String postUser(@ModelAttribute User user) { 
        // 处理"/users/"的POST请求，用来创建User 
        // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数 
        users.put(user.getId(), user); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.GET) 
    public User getUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息 
        // url中的id可通过@PathVariable绑定到函数的参数中 
        return users.get(id); 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.PUT) 
    public String putUser(@PathVariable Long id, @ModelAttribute User user) { 
        // 处理"/users/{id}"的PUT请求，用来更新User信息 
        User u = users.get(id); 
        u.setName(user.getName()); 
        u.setAge(user.getAge()); 
        users.put(id, u); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE) 
    public String deleteUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的DELETE请求，用来删除User 
        users.remove(id); 
        return "success"; 
    } 
}
```


> Representational State Transfer: REST



[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)