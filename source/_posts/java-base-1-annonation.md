---
title:  java基础（一）annonation
date: 2019-11-13 17:22:55
tags: [java]
---

本文通过jax-wx包中的一个注解【@WebServiceClient】来阐述注解的基础、使用以及和反射的搭配使用。

# 一、基础

###  1. @Override，@Deprecated，@SuppressWarnings，@Inherited

+	@Override 表示重写父类方法
+	@Deprecated 表示方法已过时
+	@SuppressWarnings 表示忽略警告
+	@Inherited：如果子类继承了被 Inherited 修饰的注解，则子类也自动拥有父类中的注解

### 2. @Retention
这是一个元注解，用于定义存在范围。

+	@Retention(RetentionPolicy.SOURCE) 保留在源码中，不会被编译
+	@Retention(RetentionPolicy.CLASS) 会在class中被编译，但不会被虚拟机运行，编译时注解
+	@Retention(RetentionPolicy.RUNTIME) 会被虚拟机运行，运行时注解

> 元注解，修饰注解的注解

### 3. @Target 
元注解，用于定义注解的使用位置，有一个枚举类型的熟悉ElementType。

```
// 声明的位置
public enum ElementType {
    /** Class, interface (包括注解),  enum 声明 */
    TYPE,
    /** 熟悉声明 */
    FIELD,

    /** 方法声明 */
    METHOD,

    /** 形参声明 */
    PARAMETER,

    /** 构造器声明 */
    CONSTRUCTOR,

    /** 局部变量声明 */
    LOCAL_VARIABLE,

    /** 注解类型声明 */
    ANNOTATION_TYPE,

    /** 包声明 */
    PACKAGE,

    /**
     * 类型参数声明
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

### 4. @Documented 元注解
@Documented 注解表明这个注解应该被 javadoc工具记录. 默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中，是一个标记注解，没有成员。

# 二、实战

### 1. 实例

以下是关于注解的一个小例子，取自【javax.xml.ws】的注解【WebServiceClient】
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME) 
@Documented
public @interface WebServiceClient {
  /**
   *  The local name of the Web service.
  **/
  String name() default "";

  /**
   *  The namespace for the Web service.
  **/
  String targetNamespace() default "";

  /**
   *  The location of the WSDL document for the service (a URL).
  **/
  String wsdlLocation() default "";
}
```

### 2. 注解@WebServiceClient是如何被使用的

使用注解@WebServiceClient：
```
// 1.在类的上方定义注解
@WebServiceClient(name = "databaseService", targetNamespace = "http://server.webservice.com/", wsdlLocation = "http://ip/project_name/databaseService?wsdl")
@Component
// 2.继承Service
public class DatabaseService_Service extends Service {

}
```

构造器：
```
	public DatabaseService_Service() {
		super(DATABASESERVICE_WSDL_LOCATION, new QName("http://server.webservice.com/", "databaseService"));
	}
```

在类【Service】的构造器中，有类似如下的一段代码,用于通过提供者模式来生成对应类【ServiceDelegate】的子类【WSServiceDelegate】
```
 private ServiceDelegate delegate;
 
  protected Service(java.net.URL wsdlDocumentLocation, QName serviceName, WebServiceFeature ... features) {
        delegate = Provider.provider().createServiceDelegate(wsdlDocumentLocation,
                serviceName,
                this.getClass(), features);
    }
```

在类【WSServiceDelegate】的构造器中，有如下的一小段代码,用于解析注解【@WebServiceClient注解】
```
public WSServiceDelegate(@Nullable Source wsdl, @Nullable WSDLService service, @NotNull QName serviceName, @NotNull final Class<? extends Service> serviceClass, WebServiceFeatureList features) {
...
// 会查看有没有传入参数【wsdl】，如果没有那么就会通过注解进行初始化
if (wsdl == null && serviceClass != Service.class) {
	WebServiceClient wsClient = (WebServiceClient)AccessController.doPrivileged(new PrivilegedAction<WebServiceClient>() {
		public WebServiceClient run() {
			return (WebServiceClient)serviceClass.getAnnotation(WebServiceClient.class);
		}
	});
	String wsdlLocation = wsClient.wsdlLocation();
	wsdlLocation = JAXWSUtils.absolutize(JAXWSUtils.getFileOrURLName(wsdlLocation));
	wsdl = new StreamSource(wsdlLocation);
}
...
}
```

### 3. 与反射的搭配
反射与注解在很多场合都可以搭配使用的，通常会使用反射来解析注解，特别的有用。
```
 Class<? extends Service> serviceClass;
 public WebServiceClient run() {
	return (WebServiceClient)serviceClass.getAnnotation(WebServiceClient.class);
}
```

# 参考资料
> [Java注释@interface的用法](https://www.cnblogs.com/liaojie970/p/7879917.html)