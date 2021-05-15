---
title: lombok
date: 2020-03-28 15:18:10
tags: [javaweb]
---

`lombok`  是一个可以通过简单的注解形式来帮助我们简化消除一些必须有但显得很臃肿的Java代码的工具，通过使用对应的注解，可以在编译源码的时候生成对应的方法.



### 配置

IDEA 相关插件：[Lombok](https://plugins.jetbrains.com/plugin/index?xmlId=Lombook%20Plugin)

相关依赖
```
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.16.18</version>
	<scope>provided</scope>
</dependency>
```


### 使用

使用方式如下：
```
@Data
public class Url implements Serializable {
    private Long id;
    private String url;
    private Date createdDate;
    private Date expiresDate;
}
```

### 常用的几个注解
+	@Data 类注解，包含了：@Getter @Setter @ToString @EqualsAndHashCode RequiredArgsConstructor注解；生成相关的get、set、equals、hashCode 以及无参构造器
+	@Setter 属性注解，生成相应set方法
+	@Getter 属性注解，生成相应get方法
+	@Log 类注解, 实例名称都是`log`
	+	@CommonsLog , `Creates log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);`
	+	@Log `Creates log = java.util.logging.Logger.getLogger(LogExample.class.getName());
	+	@Log4j `Creates log = org.apache.log4j.Logger.getLogger(LogExample.class);`
	+	@Log4j2 `Creates log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);`
	+	@Slf4j `Creates log = org.slf4j.LoggerFactory.getLogger(LogExample.class);`
	+	@XSlf4j `Creates log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);`
+	@AllArgsConstructor	类注解,：用在类上，自动生成无参构造和使用所有参数的构造函数以及把所有@NonNull属性作为参数的构造函数，如果指定staticName = “of”参数，同时还会生成一个返回类对象的静态工厂方法，比使用构造函数方便很多
+	@NoArgsConstructor 用来指定无参数构造器
+	@EqualsAndHashCode equals、hashCode方法
+	@NonNull 属性注解，空值测试
+	@Builder 建造者模式
+	@Cleanup 在当前变量不在有效范围内的时候，对其进行自动的资源回收
+	@ToString 类注解，生成toString()方法
+	@RequiredArgsConstructor  用来指定参数（采用静态方法of访问）
+	@Value 与@Data批注类似，但是它创建不可变的对象，即不包含set方法
+	@SneakyThrows 方法注解，用于生成包含try-catch 异常处理块
+	@Synchronized 处理线程安全问题



### 注解调试配置
-	Preferences | Build, Execution, Deployment | Compiler | Annotation Processors
	-	勾选`Enable annotation processing`
-	Preferences | Build, Execution, Deployment | Compiler 
	-	Shared build process VM options
		-	-Xdebug -agentlib:jdwp = transport = dt_socket,server = y,suspend = n,address = 8000


### 不推荐使用的理由
+	如果项目组中有一个人使用了Lombok，那么其他人就必须也要安装IDE插件。否则就没办法协同开发。
+	在代码中大量使用Lombok，就导致代码的可读性会低很多，而且也会给代码调试带来一定的问题。
+	如果我们需要升级到某个新版本的JDK的时候，若其中的特性在Lombok中不支持的话就会受到影响。
+	如果我们定义的一个jar包中使用了Lombok，那么就要求所有依赖这个jar包的所有应用都必须安装插件，这种侵入性是很高的。


### 参考文章
> [为什么有的程序员不推荐使用Lombok！](http://blog.itpub.net/69908877/viewspace-2676272/)
