---
title: 面试（三）SSM
date: 2020-05-31 21:17:15
tags: ms
---

### 一、SpringMVC

***SpringMVC 的工作原理***

1. 用户发送请求至前端控制器DispatcherServlet。
2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。
3. 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4. DispatcherServlet调用HandlerAdapter处理器适配器。
5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。
6. Controller执行完成返回ModelAndView。
7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
9. ViewReslover解析后返回具体View.
10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。 
11. DispatcherServlet响应用户。


***SpringMVC 常用注解都有哪些***
+	@requestMapping 用于请求 url 映射。
+	@RequestBody 注解实现接收 http 请求的 json 数据，将 json 数据转换为 java 对象。
+	@ResponseBody 注解实现将 controller 方法返回对象转化为 json 响应给客户。

***如何开启注解处理器和适配器？***
配置`<mvc:annotation-driven>`

***如何解决 get 和 post 乱码问题***
解决 post 请求乱码:
+	我们可以在 web.xml 里边配置一个 CharacterEncodingFilter 过滤器。
解决 get 请求的乱码:
+	修改 tomcat 配置文件添加编码与工程编码一致。
+	对参数进行重新编码


### 二、Spring
***谈谈你对 Spring 的理解***
Spring 是一个开源框架，为简化企业级应用开发而生。Spring 可以是使简单的 JavaBean 实现以前只有 EJB 才能实现的功能。Spring 是一个 IOC 和 AOP 容器框架。

Spring 容器的主要核心是：
+	控制反转——框架控制对象（IOC）：
	+	传统的 java 开发模式中，当需要一个对象时，我们会自己使用 new 或者 getInstance 等直接或者间接调用构造方法创建一个对象。而在 spring 开发模式中，spring 容器使用了工厂模式为我们创建了所需要的对象，不需要我们自己创建了，直接调用 spring 提供的对象就可以了，这是控制反转的思想。
+	依赖注入——通过配置初始化对象（DI）：
	+	spring 使用 javaBean 对象的 set 方法或者带参数的构造方法为我们在创建所需对象时将其属性自动设置所需要的值的过程，就是依赖注入的思想。
+	面向切面编程（AOP）：
	+	AOP是对OOP的补充和完善,它将散落在系统中的非核心业务关注点集中成切面。AOP 底层是动态代理

***Spring 中的设计模式***
单例模式——spring 中两种代理方式，若目标对象实现了若干接口，spring 使用 jdk 的java.lang.reflect.Proxy类代理。若目标兑现没有实现任何接口，spring 使用 CGLIB 库生成目标类的子类。单例模式——在 spring 的配置文件中设置 bean 默认为单例模式。

模板方式模式——用来解决代码重复的问题。

***Spring 的常用注解***

@Required 表示必须进行初始化
@Autowired 自动注入
@Qualifier 用于消除特定bean自动装配歧义


***spring bean 生命周期***

定义：在配置文件里面用<bean></bean>来进行定义。
初始化
调用
销毁

***请描述一下 Spring 的事务***

+	声明式事务
+	注解事务@Transactional

***Spring WEB 模块***

Spring 的 WEB 模块是构建在 application context 模块基础之上，提供一个适合 web 应用的上下文。

***有哪些不同类型的 IOC（依赖注入）方式？***

+	Set 注入
+	构造器注入
+	静态工厂的方法注入
+	实例工厂的方法注入

***Spring 支持的几种 bean 的作用域***

+	singleton : bean 在每个 Spring ioc 容器中只有一个实例。
+	prototype：一个 bean 的定义可以有多个实例
+	request：每次 http 请求都会创建一个 bean，该作用域仅在基于 web 的 Spring ApplicationContext 情形下有效。
+	session ：在一个 HTTP Session 中 ， 一 个 bean 定义对应一个实例。该作用域仅在基于 web 的Spring ApplicationContext 情形下有效。
+	global-session：在一个全局的 HTTP Session 中，一个 bean 定义对应一个实例。该作用域仅在基于 web 的Spring ApplicationContext 情形下有效。
+	缺省的 Spring bean 的作用域是 Singleton。

***Spring 框架中的单例 bean 是线程安全的吗? ***
Spring 框架中的单例 bean 不是线程安全的。

***什么是 Spring 的内部 bean？***
当一个 bean 仅被用作另一个 bean 的属性时，它能被声明为一个内部 bean，为了定义 inner bean，在
Spring 的 基于 XML 的 配置元数据中，可以在 <property/>或 <constructor-arg/> 元素内使用<bean/> 元素，内
部 bean 通常是匿名的，它们的 Scope 一般是 prototype。

***在 Spring 中如何注入一个 java 集合***
Spring 提供以下几种集合的配置元素：
<list>类型用于注入一列值，允许有相同的值。
<set> 类型用于注入一组值，不允许有相同的值。
<map> 类型用于注入一组键值对，键和值都可以为任意类型。
<props>类型用于注入一组键值对，键和值都只能为 String 类型。

***bean的自动装配***
无须在 Spring 配置文件中描述 javaBean 之间的依赖关系（如配置<property>、<constructor-arg>）。IOC 容
器会自动建立 javabean 之间的关联关系


### 三、Mybatis
***Mybatis 中#和$的区别？***

+	`#`相当于对数据 加上 双引号，`$`相当于直接显示数据.
+	`#`方式能够很大程度防止 sql 注入;$方式无法防止 Sql 注入。
+	.$方式一般用于传入数据库对象，例如传入表名.一般能用#的就别用$.

例如：
```
order by #user_id#
解析为：order by "11"

order by $user_id$
解析为：order by 11

```

***Mybatis 的编程步骤是什么样的***
1、创建 SqlSessionFactory
2、通过 SqlSessionFactory 创建 SqlSession
3、通过 sqlsession 执行数据库操作
4、调用 session.commit()提交事务
5、调用 session.close()关闭会话

***使用 MyBatis 的 mapper 接口调用时有哪些要求***
+	Mapper.xml 文件中的 namespace 即是 mapper 接口的`类路径`
+	Mapper 接口`方法名`和 mapper.xml 中定义的每个 sql 的 id 相同
+	Mapper 接口方法的`输入参数类型`和 mapper.xml 中定义的每个 sql 的 parameterType 的类型相同
+	Mapper 接口方法的`输出参数类型`和 mapper.xml 中定义的每个 sql 的 resultType 的类型相同

***Mybatis 中一级缓存与二级缓存***
一级缓存:   HashMap 本地缓存，其存储作用域为 Session 
二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为
Mapper(Namespace)，并且可自定义存储源

