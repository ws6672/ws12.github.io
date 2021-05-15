---
title: springboot（三）注解
date: 2020-08-17 15:58:28
tags: [javaweb]
---

### 通用注解
+	@Autowired
	+	表示被修饰的类需要注入对象,spring会扫描所有被@Autowired标注的类,然后根据 类型在ioc容器中找到匹配的类注入
+	@Resource(type,name)
	+	默认按 byName
	+	使用name属性，则使用 `byName自动注入策略`; 而使用type属性时则使用 `byType自动注入策略`
	+	如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
+	@Component
	+	把普通pojo实例化到spring容器中，相当于配置文件中的 `<bean id="" class=""/>`
+	@Service
	+	标注服务层
+	@Scope
	+	设置bean作用域
	+	类型
		+	singleton 表示在spring容器中的单例，通过spring容器获得该bean时总是返回唯一的实例
		+	prototype 表示每次获得bean都会生成一个新的对象
		+	request 表示在一次http请求内有效（只适用于web应用）
		+	session 表示在一个用户会话内有效（只适用于web应用）
		+	globalSession 表示在全局会话内有效（只适用于web应用）
+	@Bean
	+	在配置类中使用，即类上需要加上@Configuration注解
+	@Configuration
	+	定义配置类，可替换xml配置文件
+	@ComponentScan
	+	扫描当前包及其子包下被`@Component`，`@Controller`，`@Service`，`@Repository`注解标记的类并纳入到spring容器中进行管理
+	@SpringBootApplication：声明为应用启动器
+	@EnableAutoConfiguration：启用自动配置（类按约定配置装载）

### Spring MVC注解

+	@Controller：标记为控制器
+	@ResponseBody：返回值为JSON数据
+	@RestController：@Controller+@ResponseBody
+	@RequestMapping
	+	处理请求地址映射
	+	value： 指定请求的实际地址
	+	method： 指定请求的method类型， GET、POST、PUT、DELETE等
	+	consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
	+	produces: 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；
	+	params： 指定request中必须包含某些参数值是，才让该方法处理。
	+	headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。
+	@RequesParam
	+	指定用户访问携带的参数（blogs?blogId=1），required默认为true，必须携带
	+	@RequestParam(name="id",required=false)
+	@PathVariable
	+	获取访问路径的值作为函数值
	+	例如：```
	@RequestMapping("/users/{username}")
    @ResponseBody
    public String userProfile(@PathVariable String username){
//        return String.format("user %s", username);
        return "user" + username; 
    }
	```
+	@ControllerAdvice
	+	对所有的Controller进行加强，统一处理异常。
+	@ExceptionHandler
	+	全局异常统一处理
	

