---
title: springboot（二）配置与启动
date: 2020-08-17 13:53:19
tags: [javaweb]
---



### 1. 基本配置

##### 1.1 Banner

Banner是指横幅之类的，有趣的是每次`SpringBoot`都会输出以下的图案：
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
```

而我们自定义这个图案。

***语法***

```
banner.txt配置
　　${AnsiColor.BRIGHT_RED}：设置控制台中输出内容的颜色
　　${application.version}：用来获取MANIFEST.MF文件中的版本号
　　${application.formatted-version}：格式化后的${application.version}版本信息
　　${spring-boot.version}：Spring Boot的版本号
　　${spring-boot.formatted-version}：格式化后的${spring-boot.version}版本信息
```

***自定义banner***

1. 编写文件`banner.txt`
```
${AnsiColor.BRIGHT_YELLOW}
////////////////////////////////////////////////////////////////////
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//            佛祖保佑       永不宕机      永无BUG                //
////////////////////////////////////////////////////////////////////
```

2. 定义`application.yml`
```
spring: 
  banner:
    charset: UTF-8
    location: banner.txt
```

***关闭banner***
```
@SpringBootApplication
public class ExampleApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(ExampleApplication.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.run(args);
    }
}
```

##### 1.2 全局配置文件

SpringBoot使用的全局配置文件为：
+	application.properties
+	application.yml

配置一个即可，yml格式比properties格式的简洁一些。配置文件路径是`\src\main\resources`。

***properties 实例***

```
server.port=8888
server.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
server.datasource.url=mysql://localhost:3306/toolweb?serverTimezone=UTC&useSSL=false
server.datasource.username=root
server.datasource.password=root
```
***yml 实例***

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


***使用XML文件***

XML文件配置如下：`@ImportResource({"classpath:some-context.xml","classpath:another-context.xml"})`


### 2.  外部配置

外部配置包括命令行参数配置、常规属性配置以及类型安全的配置。

##### 2.1 命令行参数配置

默认情况下，SpringApplication会将所有命令行配置参数（以'--'开头，比如`--server.port=9000`）转化成一个property，并将其添加到Spring Environment中，命令行属性总是优先于其他属性源。

阻止属性添加到环境：`SpringApplication.setAddCommandLineProperties(false)`

springboot项目可以基于jar包运行，打开jar的程序可以通过下面命令行运行：`java -jar xxx.jar`

修改tomcat端口：`java -jar xxx.jar --server.port=9090`

##### 2.2 常规属性配置

***application.properties ***
```
book.author=wangyunfei
book.name=spring boot
```

***入口类***
通过`@Value`注入属性
```
@RestController
@SpringBootApplication
public class DemoApplication {

    @Value("${book.author}")
    private String bookAuthor;

    @Value("${book.name}")
    private String bookName;

    @RequestMapping("/")
    String index(){
        return "book name is :" + bookName + " and book author is: " + bookAuthor;
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}


```

##### 2.3 类型安全的配置

通过`@ConfigurationProperties`将`properties`属性和一个`Bean`及其相关属性关联、从而实现类型安全的配置

```
author.name=wyf
author.age=32

@Component
@ConfigurationProperties(prefix = "author")
public class Author {
}

@RestController
@SpringBootApplication
public class DemoApplication {

    @Value("${book.author}")
    private String bookAuthor;

    @Value("${book.name}")
    private String bookName;

    @Autowired
    Author author;
	......
}
```

### 3. 其它相关配置

***配置的优先级***

1. Spring Boot 所提供的配置优先级顺序比较复杂。按照优先级从高到低的顺序，具体的列表如下所示：
+	命令行参数（优先级最高，有利于实际环境调整参数）
+	通过 System.getProperties() 获取的 Java 系统参数。
+	操作系统环境变量。
+	从 java:comp/env 得到的 JNDI 属性。
+	通过 RandomValuePropertySource 生成的`“random.*”`属性。
+	应用 Jar 文件之外的属性文件。(通过spring.config.location参数)
+	应用 Jar 文件内部的属性文件。
+	在应用配置 Java 类（包含“@Configuration”注解的 Java 类）中通过“@PropertySource”注解声明的属性文件。
+	通过“SpringApplication.setDefaultProperties”声明的默认属性。

2. 在同一目录下，properties配置优先级 > YAML配置优先级、

3. SpringBoot配置文件可以放置在多种路径下，不同路径下的配置优先级有所不同。
可放置目录(优先级从高到低)，高优先级的配置会覆盖低优先级的配置：
```
	file:./config/ (当前项目路径config目录下);
	file:./ (当前项目路径下);
	classpath:/config/ (类路径config目录下);
	classpath:/ (类路径config下).
```

##### 3.1 profile
Profile主要用于区分不同的环境。

***@Profile***
在某个类、或者方法上添加`@Profile`注解，指定具体的profile环境标签，那么只有在该profile处于active的情况下该类、方法才会被加载、执行。

```
@Profile({"dev","test"})
public class Xxx{
    
    @Profile({"dev"})
    @Bean
    public Xxx xxx(){
        return new Xxx();
    }
}
```

***多环境配置***

使用properties配置文件实现多环境配置，可以通过添加多个application-{profile}.properties来实现。比如：
`application-dev.properties,application-test.properties`

使用YAML实现多环境配置要简单的多，只需要一个文件即可，application.yml.例如：

```
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

我们也可以通过命令行来激活相关配置，而且优先级更高.例如：`--spring.profiles.active=dev`

##### 3.2 日志

> 为了保证服务的高可用，发现问题一定要即使，解决问题一定要迅速，所以生产环境一旦出现问题，预警系统就会通过邮件、短信甚至电话的方式实施多维轰炸模式，确保相关负责人不错过每一个可能的bug。

Spring Boot默认使用LogBack日志系统，如果不需要更改为其他日志系统如Log4j2等，则无需多余的配置，LogBack默认将日志打印到控制台上。新建的Spring Boot项目一般都会引用`spring-boot-starter`或者`spring-boot-starter-web`，这两个依赖中已经包含了对于`spring-boot-starter-logging`的依赖，所以，无需额外添加依赖。

> Spring Boot默认使用LogBack日志系统，默认的日志级别为INFO。

***存储日志***
默认文件名为spring.log，默认路径在当前项目下

```
logging:
  file:
    name: springboot-example.log # 配置日志文件名
```

***日志级别***

日志级别总共有TRACE < DEBUG < INFO < WARN < ERROR < FATAL ，且级别是逐渐提供，如果日志级别设置为INFO，则意味TRACE和DEBUG级别的日志都看不到。

```
# 配置级别为 warn，INFO级别以下的信息无法查看，只显示（WARN、ERROR、FATAL ）级别的信息
logging:
  file:
    name: springboot-example.log
  level:
    root: warn
  
```

***定义日志格式***

springboot支持自定义日志输出格式，例如：
```
# 定制日志输出格式
logging.pattern.console=%d{yyyy/MM/dd-HH:mm:ss} [%thread] %-5level %logger- %msg%n  
logging.pattern.file=%d{yyyy/MM/dd-HH:mm} [%thread] %-5level %logger- %msg%n
```

符号含义如下：
```
%d{HH:mm:ss.SSS}——日志输出时间
%thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用
%-5level——日志级别，并且使用5个字符靠左对齐
%logger- ——日志输出者的名字
%msg——日志消息
%n——平台的换行符
```

### 4. 核心注解

##### 4.1 @SpringBootApplication

@SpringBootApplication是一个复合注解，源码如下

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "nameGenerator"
    )
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```


可以看到该注解中包含了以下三个重要注解:
+	`@SpringBootConfiguration`
	+	继承自`@Configuration`，二者功能也一致，标注当前类是配置类，并会将当前类内声明的一个或多个以`@Bean`注解标记的方法的实例纳入到srping容器中，并且实例名就是方法名。
	+	也就是说，定义了该注解的类中，可以通过在方法上定义`@Bean` 配置文件
+	`@EnableAutoConfiguration`
	+	启用自动配置
	+	根据添加的jar包来配置项目的默认配置
+	`@ComponentScan`
	+	扫描组件, 类似于 Spring的配置 `<context:component-scan>`
	+	扫描当前包及其子包下被`@Component`，`@Controller`，`@Service`，`@Repository`注解标记的类并纳入到spring容器中进行管理
	

***关闭自动配置***
例如以下配置，会取消自动配置`DataSourceAutoConfiguration`

```
	@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
```


##### 4.2 @Enable* 系列注解

> `@enable*`是springboot中用来启用某一个功能特性的一类注解。其中包括我们常用的几个注解：
用于开启自动注入的注解`@EnableAutoConfiguration`
开启异步方法的 注解`@EnableAsync`
开启将配置文件中的属性以bean的方式注入到IOC容器的注解`@EnableConfigurationProperties`


***@EnableAutoConfiguration***

它的源码如下:
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}


// 由Spring提供的，用来导入配置类的，在配置类中定义的Bean(@Bean)，可通过@Autowired注入到容器中，也就是可以被扫描到
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
    Class<?>[] value();
}

```
`@Import`用来导入一个或多个class，这些类会注入到spring容器中，或者配置类，配置类里面定义的bean都会被spring容器托管。通过`@Import`注入了`AutoConfigurationImportSelector`类。



### 5. 启动原理


SpringBoot项目会自动创建一个启动器，名字一般是`XXXApplication`。启动器源码如下，通过`SpringApplication`构造实例，通过`RUN()`方法启动项目。

```
@SpringBootApplication
public class ExampleApplication {
    public static void main(String[] args) {
		SpringApplication.run(ExampleApplication.class, args);
    }
}
```


##### 5.1 SpringApplication 构造器

SpringApplication 的作用：
1、创建一个恰当的ApplicationContext实例（取决于类路径）
2、注册 CommandLinePropertySource，将命令行参数公开为Spring属性
3、刷新应用程序上下文，加载所有单例bean
4、触发全部CommandLineRunner bean

构造器如下：

```
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.sources = new LinkedHashSet();
	this.bannerMode = Mode.CONSOLE;
	this.logStartupInfo = true;
	this.addCommandLineProperties = true;
	this.addConversionService = true;
	this.headless = true;
	this.registerShutdownHook = true;
	this.additionalProfiles = new HashSet();
	this.isCustomEnvironment = false;
	this.lazyInitialization = false;
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
	this.webApplicationType = WebApplicationType.deduceFromClasspath();
	this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
	this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
	this.mainApplicationClass = this.deduceMainApplicationClass();
}
```

主要是 deduceFromClasspath、getSpringFactoriesInstances、deduceMainApplicationClass 几个方法


***deduceFromClasspath***

源码如下，用于推测WEB应用类型

```
static WebApplicationType deduceFromClasspath() {
	if (ClassUtils.isPresent("org.springframework.web.reactive.DispatcherHandler", (ClassLoader)null) && !ClassUtils.isPresent("org.springframework.web.servlet.DispatcherServlet", (ClassLoader)null) && !ClassUtils.isPresent("org.glassfish.jersey.servlet.ServletContainer", (ClassLoader)null)) {
		return REACTIVE;
	} else {
		String[] var0 = SERVLET_INDICATOR_CLASSES;
		int var1 = var0.length;

		for(int var2 = 0; var2 < var1; ++var2) {
			String className = var0[var2];
			if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
				return NONE;
			}
		}

		return SERVLET;
	}
}
// 用于判断类能否加载，路径是否存在既定类
public static boolean isPresent(String className, @Nullable ClassLoader classLoader) {
	try {
		forName(className, classLoader);
		return true;
	} catch (IllegalAccessError var3) {
		throw new IllegalStateException("Readability mismatch in inheritance hierarchy of class [" + className + "]: " + var3.getMessage(), var3);
	} catch (Throwable var4) {
		return false;
	}
}
```

从源码上看，主要是判断应用类型是否为`NONE`、`SERVLET`或者`REACTIVE`，这三种类型都是`WebApplicationType`中定义的。含义如下：
+	SERVLET：传统 MVC Web项目
+	REACTIVE：响应式Web项目(WebFlux)
+	NONE：非web项目


***getSpringFactoriesInstances***

源码如下，主要做了几件事：
+	`loadFactoryNames`加载工程名称
+	`createSpringFactoriesInstances`根据名字、类型创建工厂实例
+	`AnnotationAwareOrderComparator.sort(instances)` 对工厂实例进行排序

```
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
	ClassLoader classLoader = this.getClassLoader();
	Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
	List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
	AnnotationAwareOrderComparator.sort(instances);
	return instances;
}
```

***deduceMainApplicationClass***

从当前堆栈跟踪列表中获取main方法所在的类名
```
private Class<?> deduceMainApplicationClass() {
	try {
		StackTraceElement[] stackTrace = (new RuntimeException()).getStackTrace();
		StackTraceElement[] var2 = stackTrace;
		int var3 = stackTrace.length;

		for(int var4 = 0; var4 < var3; ++var4) {
			StackTraceElement stackTraceElement = var2[var4];
			if ("main".equals(stackTraceElement.getMethodName())) {
				return Class.forName(stackTraceElement.getClassName());
			}
		}
	} catch (ClassNotFoundException var6) {
	}

	return null;
}
```


##### 5.2 SpringApplication.run() 方法


该方法源码如下，主要是 `prepareEnvironment`、`createApplicationContext`、`prepareContext`三个方法

```
    public ConfigurableApplicationContext run(String... args) {
		//  记录请求执行时间，便于构建日志
        StopWatch stopWatch = new StopWatch();
		
        stopWatch.start();

        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
		
		// 1. 配置属性
        this.configureHeadlessProperty();
		
		//  利用loadFactoryNames方法从路径MEAT-INF/spring.factories中找到所有的SpringApplicationRunListener
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
		
		//  启动监听器
        listeners.starting();

        Collection exceptionReporters;
        try {
			// 封装参数到ApplicationArguments对象
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			
			// 触发监听器(调用每个监听器的environmentPrepared()方法)；存在命令行参数就进行配置
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
			
			// 定义 spring.beaninfo.ignore
            this.configureIgnoreBeanInfo(environment);
			// 获取Banner
            Banner printedBanner = this.printBanner(environment);
			
			// 2. 初始化 context
            context = this.createApplicationContext();
			
			// 解析META-INF/spring.factories文件下的几个类
            exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
			
			// 3. 准备上下文
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
			
			
            stopWatch.stop();
			
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }
		
            listeners.started(context);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, exceptionReporters, listeners);
            throw new IllegalStateException(var10);
        }

        try {
            listeners.running(context);
            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var9);
        }
    }
	
```

***configureHeadlessProperty***

用来设置java.awt.headless 属性是true 还是false, java.awt.headless是J2SE的一种模式用于在缺少显示屏、键盘或者鼠标时的系统配置，很多监控工具如jconsole 需要将该值设置为true
```

    private void configureHeadlessProperty() {
        System.setProperty("java.awt.headless", System.getProperty("java.awt.headless", Boolean.toString(this.headless)));
    }
```


***prepareEnvironment***

准备环境
+	配置环境
+	附加命令行参数

```
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments) {

    // 获取或创建环境
	ConfigurableEnvironment environment = this.getOrCreateEnvironment();
	
	// 配置环境：配置PropertySources和activeProfiles
	this.configureEnvironment((ConfigurableEnvironment)environment, applicationArguments.getSourceArgs());
	

	ConfigurationPropertySources.attach((Environment)environment);
	
	// listeners环境准备
	listeners.environmentPrepared((ConfigurableEnvironment)environment);
	// 将环境绑定到SpringApplication
	this.bindToSpringApplication((ConfigurableEnvironment)environment);
	
	
	// environment转换为StandardEnvironment对象
	if (!this.isCustomEnvironment) {
		environment = (new EnvironmentConverter(this.getClassLoader())).convertEnvironmentIfNecessary((ConfigurableEnvironment)environment, this.deduceEnvironmentClass());
	}
	
    // 配置PropertySources对它自己的递归依赖
	ConfigurationPropertySources.attach((Environment)environment);
	return (ConfigurableEnvironment)environment;
}


protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
	if (this.addConversionService) {
		ConversionService conversionService = ApplicationConversionService.getSharedInstance();
		environment.setConversionService((ConfigurableConversionService)conversionService);
	}
	// 将命令行参数公开为Spring属性
	this.configurePropertySources(environment, args);
	// 设置Profiles，命令行存在则启用命令行的配置
	this.configureProfiles(environment, args);
}


```


***createApplicationContext***

初始化context。通过构造器中初始化的`webApplicationType`变量来确定项目类型，从而初始化恰当的 context

```
	protected ConfigurableApplicationContext createApplicationContext() {
        Class<?> contextClass = this.applicationContextClass;
        if (contextClass == null) {
            try {
				// 判断项目类型
                switch(this.webApplicationType) {
					case SERVLET:
						contextClass = Class.forName("org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext");
						break;
					case REACTIVE:
						contextClass = Class.forName("org.springframework.boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext");
						break;
					default:
						contextClass = Class.forName("org.springframework.context.annotation.AnnotationConfigApplicationContext");
                }
            } catch (ClassNotFoundException var3) {
                throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", var3);
            }
        }

        return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
    }
```


***prepareContext***

准备上下文：
+   设置上下文的environment
+	将命令行参数转换为bean
+	打印启动日志
+	懒加载判断
+	将bean加载到应用上下文中

```
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
	// 设置上下文的environment，将context的 environment 切换为SpringApplication的 environment
	context.setEnvironment(environment);
	// 应用上下文后处理
	this.postProcessApplicationContext(context);
	// 应用环境初始化
	this.applyInitializers(context);
	
	//  上下文准备
	listeners.contextPrepared(context);
	
	// // 打印启动日志和启动应用的Profile
	if (this.logStartupInfo) {
		this.logStartupInfo(context.getParent() == null);
		this.logStartupProfileInfo(context);
	}

	// 获取配置bean工厂
	ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
	// 注册cmd参数
	beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
	// 注册 banner
	if (printedBanner != null) {
		beanFactory.registerSingleton("springBootBanner", printedBanner);
	}


	if (beanFactory instanceof DefaultListableBeanFactory) {
		((DefaultListableBeanFactory)beanFactory).setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
	}

	// 懒加载
	if (this.lazyInitialization) {
		context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
	}

	Set<Object> sources = this.getAllSources();
	Assert.notEmpty(sources, "Sources must not be empty");
	
	// 将bean加载到应用上下文中
	this.load(context, sources.toArray(new Object[0]));
	
	// 向上下文中添加ApplicationListener，并广播ApplicationPreparedEvent事件
	listeners.contextLoaded(context);
}
```


[spring-boot-2.0.3启动源码篇五 - run方法(四)之prepareContext](https://www.cnblogs.com/youzhibing/p/9697825.html)