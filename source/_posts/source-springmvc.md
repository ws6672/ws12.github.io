---
title: springmvc源码阅读（一）
date: 2020-06-26 12:51:46
tags: [javaweb,源码]
---


# 1.入门

### 1.1 什么是SpringMVC

Spring Web MVC是基于Servlet API构建的原始Web框架，从一开始就已包含在Spring框架中。正式名称为“Spring Web MVC”，来自其源模块（spring-webmvc）的名称，但通常称为“Spring MVC”。

Spring Framework 5.0引入了一个反应式堆栈Web框架，与Spring Web MVC类似，其名称是“Spring WebFlux”，该名称也基于其源模块（spring-webflux）。



### 1.2 DispatcherServlet

与其他许多Web框架一样，Spring MVC围绕前端控制器模式进行设计Servlet，命名为`DispatcherServlet`。Central提供了用于请求处理的共享算法，而实际工作是由可配置的委托组件执行的。`DispatcherServlet`使用Spring配置文件来获取请求映射、进行视图解析、异常处理、委托组件等工作。

##### 1.2.1 通过web.xml配置



配置如下：

```
	<web-app>

		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		</listener>

		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/app-context.xml</param-value>
		</context-param>

		<servlet>
			<servlet-name>app</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value></param-value>
			</init-param>
			<load-on-startup>1</load-on-startup>
		</servlet>

		<servlet-mapping>
			<servlet-name>app</servlet-name>
			<url-pattern>/app/*</url-pattern>
		</servlet-mapping>

	</web-app>
```

##### 1.2.2 上下文环境

`Root WebApplicationContext` 在多个`DispatcherServlet`（或其他Servlet）实例之间共享,每个实例都有其自己的子`WebApplicationContext`配置.

`Root WebApplicationContext` 通常包含基础结构bean，例如需要在多个Servlet实例之间共享数据存储库和业务服务。这些Bean被有效地继承，并且可以在Servlet特定的子级中重写（即重新声明），该子级`WebApplicationContext`通常包含给定本地的 Bean `Servlet`。

![DispatcherServlet上下文环境](/image/source/springmvc/dispatcherServletContext.png)

##### 1.2.3 内置的Bean

+	***HandlerMapping***
	+	将请求与拦截器列表一起映射到处理程序，以 进行预处理和后期处理。映射基于某些标准，具体标准因HandlerMapping 实现而异。
	+	主要实现
		+	RequestMappingHandlerMapping （支持带@RequestMapping注释的方法）
		+	SimpleUrlHandlerMapping （维护URI路径的显式注册）。
+	***HandlerAdapter***
	+	帮助DispatcherServlet调用映射到请求的处理程序，而不管实际如何调用该处理程序，用于保护细节不被暴露
+	HandlerExceptionResolver
	+	解决异常的策略，可能将它们映射到处理程序
+	***ViewResolver***
	+	 将从处理程序返回的基于逻辑的视图名称解析为实际的View
+	LocaleResolver，LocaleContextResolver
	+	解决Locale一个客户正在使用的并且可能是其时区的问题，以便能够提供国际化的视图
+	ThemeResolver
	+	应用Spring Web MVC框架主题来设置应用程序的整体外观
+	MultipartResolver
	+	借助一些多部分解析库来解析多部分请求的抽象（例如，浏览器表单文件上传）
+	FlashMapManager
	+	通常用于通过重定向将属性从一个请求传递到另一个请求
	
##### 1.2.4 通过容器配置
在Servlet 3.0+环境中，您可以选择以编程方式配置Servlet容器，以替代方式或与web.xml文件结合使用。以下示例注册一个DispatcherServlet：

```
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```

***

# 2. 前置配置

在查看源码前，需要先配置一个简单的项目。

### 2.1 相关资源

[下载项目]()
[官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)

***通过maven下载文档***

![通过maven下载文档](/image/source/springmvc/mavendocument.png)



### 2.2 项目配置

本项目使用的是IDEA,需要配置`File->Project Structure`中的Facets（页面）、Artifacts（打包方式）

项目目录结构如下：
![项目结构](/image/source/springmvc/xmjg.png)



***1. 定义包 pom.xml***

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.eg</groupId>
    <artifactId>springmvc-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <spring.version>4.3.7.RELEASE</spring.version>
        <mybatis.version>3.4.6</mybatis.version>
        <mysql.version>8.0.18</mysql.version>
        <c3p0.version>0.9.1.2</c3p0.version>
    </properties>

    <dependencies>
        <!--  spring 基本配置  -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>

<!--        spring-webmvc是spring-web的实现，包含，不必定义-->
<!--        <dependency>-->
<!--            <groupId>org.springframework</groupId>-->
<!--            <artifactId>spring-web</artifactId>-->
<!--            <version>${spring.version}</version>-->
<!--        </dependency>-->

        <!--  springmvc 基本配置  -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>


        <!-- mybatis/spring包 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.2</version>
        </dependency>

        <!--  mybatis 基本配置  -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <!--mysql数据库驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>

        <!--log4j日志包 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.6.1</version>
        </dependency>

        <!--druid 连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.7</version>
        </dependency>




    </dependencies>

<!--    配置tomcat插件-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                </configuration>
            </plugin>
        </plugins>
        <finalName>generator</finalName>
<!--IDEA不会编译src的java目录下的xml文件,需要配置-->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>

</project>
```
***2. 配置web项目 WEB.xml***

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

<!--    配置spring-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring/applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>


<!--    避免编码乱码-->
    <filter>
        <filter-name>error Encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>error Encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

<!--    配置初始页面-->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>

</web-app>
```


***3. 配置spring applicationContext.xml***

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
          http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context-3.0.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-3.0.xsd" default-autowire="byName">

    <context:annotation-config />

<!--    包扫描-->
    <context:component-scan base-package="com.eg.springmvc">
<!--        Spring 不扫描@Controller-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

<!--    2.1 引入配置文件-->
    <context:property-placeholder location="classpath:confg/db.properties"/>

    <!-- 1) 获得数据库连接池对象，并交由 spring 同一管理 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!-- 连接数据库的驱动，连接字符串，用户名和登录密码-->
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/eg?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false&amp;serverTimezone=UTC&amp;rewriteBatchedStatements=true"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>

<!--        &lt;!&ndash; 数据池中最大连接数和最小连接数&ndash;&gt;-->
<!--        <property name="maxActive" value="${jdbc.max}"/>-->
<!--        <property name="minIdle" value="${jdbc.min}"/>-->
    </bean>

<!--2.3 配置会话工厂-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--配置实体类前缀-->
        <property name="typeAliasesPackage" value="com.eg.springmvc.pojo"/>
        <!-- 映射mapper文件 -->
        <property name="mapperLocations" value="classpath*:com/eg/springmvc/mapper/*.xml"/>
    </bean>
<!--2.4 扫描mapper所在的包 为mapper创建实现类-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.eg.springmvc.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

<!--2.5 配置事务-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--支持注解驱动的事务管理，指定事务管理器 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>

```



***4.配置springmvc springmvc.xml***
```
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/mvc
http://www.springframework.org/schema/mvc/spring-mvc.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd">
    <!--组件扫描 只扫描controller-->
    <context:component-scan base-package="com.eg.springmvc.controller"></context:component-scan>
    <!--配置mvc注解驱动-->
    <mvc:annotation-driven/>

    <!--内部资源视图解析器-->
    <bean id="resourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!--开放静态资源访问权限-->
    <mvc:default-servlet-handler/>

</beans>
```

### 2.3 阅读技巧

+	进入普通文件ctrl+shift+n:
+	类查询
	+	ctrl+f12:查看该类方法
	+	ctrl+n:快速进入类
	+	ctrl+alt+u:查看类结构图
	+	ctrl+shift+alt+u:查看类结构图，这些类不能进入

+	方法引用
	+	alt+f7:查看方法引用位置
	+	ctrl+alt+b:跳转到方法实现处

+	书签
	+	ctrl+f11:显示bookmark标记情况，土黄色代表该字符已被占用
	+	ctrl+标记编号 快速回到标记处
	+	shift+f11:显示所有书签
	+	shift+f11：普通书签
+	alt+f8：启用Evaluate窗口

***
	
# 3. 参考文章
> [官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)
[SpringMVC源码阅读入门](https://www.cnblogs.com/Java-Starter/p/10304896.html)