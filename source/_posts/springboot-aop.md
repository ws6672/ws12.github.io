---
title: SpringBoot 使用 AOP（面向切面编程）
date: 2021-04-04 21:44:37
tags: [springboot]
---


AOP （Aspect Orient Programming）,译为 面向切面编程，AOP 是一种编程思想，是面向对象编程（OOP）的一种补充。

# 一、基础

面向切面编程实现了横切关注点的模块化，即多应用对象相同的处理逻辑被抽象为模块。在多个模块中，如果存在重复的代码，我们可以通过将该代码段抽取成特定的方法使用。但是，如果有新的需求，又需要重复增加新的方法。这时，我们可以通过AOP在不修改源码的情况下，为系统的多个组件添加相同的功能。

1. 分类

AOP可以分为两类：
+	静态 AOP 实现：代理类和被代理的类实现了同样的接口，代理类同时持有被代理类的引用，通过该引用代理类可以调用被代理类的方法
+	动态 AOP 实现： AOP 框架在运行阶段动态生成代理对象
	+	JDK 动态代理：利用反射机制生成一个实现代理接口的类，在调用具体方法前调用InvokeHandler来处理；针对实现接口的类生成代理。
	+	CGlib 动态代理：利用ASM（开源的Java字节码编辑库，操作字节码）开源包，将代理对象类的class文件加载进来，通过修改其字节码生成子类来处理；针对类实现代理，不能代理final修饰的类。

2. 实现

AOP 实现的比较：
![AOP 实现](/image/ssm/aop.png)


3. 术语

+	连接点（Joinpoint）：程序执行的某个特定位置，如类开始初始化前、类初始化后、类某个方法调用前、调用后、方法抛出异常后。一个类或一段程序代码拥有一些具有边界性质的特定点，这些点中的特定点就称为“连接点”。Spring仅支持方法的连接点，即仅能在方法调用前、方法调用后、方法抛出异常时以及方法调用前后这些程序执行点织入增强
+	切点（Pointcut）：每个程序都可以拥有多个连接点，而AOP通过切点查询连接点。切点与连接点的关系是一对多的关系。在Spring中，切点通过org.springframework.aop.Pointcut接口进行描述，它使用类和方法作为连接点的查询条件，Spring AOP的规则解析引擎负责切点所设定的查询条件，找到对应的连接点。
+	增强（Advice）；增强是织入到目标类连接点上的一段程序代码，在Spring中，增强除用于描述一段程序代码外，还拥有另一个和连接点相关的信息，这便是执行点的方位。结合执行点方位信息和切点信息，我们就可以找到特定的连接点
+	目标对象（Target）：    增强逻辑的织入目标类。如果没有AOP，目标业务类需要自己实现所有逻辑，而在AOP的帮助下，目标业务类只实现那些非横切逻辑的程序逻辑，而性能监视和事务管理等这些横切逻辑则可以使用AOP动态织入到特定的连接点上。
+	引介（Introduction）：引介是一种特殊的增强，它为类添加一些属性和方法
+	织入（Weaving）：织入是将增强添加对目标类具体连接点上的过程；织入方式如下：
	+	编译期织入，这要求使用特殊的Java编译器。
	+	类装载期织入，这要求使用特殊的类装载器。
	+	动态代理织入，在运行期为目标类添加增强生成子类的方式。（Spring采用的方式）
+	切面（Aspect）：切面由切点和增强（引介）组成，它既包括了横切逻辑的定义，也包括了连接点的定义，Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入到切面所指定的连接点中。

通知（Advice）类型：
+	前置通知（Before advice）：在某连接点（JoinPoint）之前执行的通知，但这个通知不能阻止连接点前的执行
+	后置通知（After advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）
+	返回后通知（After return advice）：在某连接点正常完成后执行的通知，不包括抛出异常的情况
+	环绕通知（Around advice）：包围一个连接点的通知，类似Web中Servlet规范中的Filter的doFilter方法。可以在方法的调用前后完成自定义的行为，也可以选择不执行
+	抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知



# 二、JDK 动态代理

Java动态代理类位于java.lang.reflect包下，是使用InvocationHandler和Proxy实现的。


1. InvocationHandler

InvocationHandler接口是proxy代理实例的调用处理程序实现的一个接口，每一个proxy代理实例都有一个关联的调用处理程序；在代理实例调用方法时，方法调用被编码分派到调用处理程序的invoke方法。 相关源码如下：
```
// Object proxy 代理类
// Method method 被代理的方法
// Object[] args 方法的参数数组
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}

```

通过这个接口，我们可以实现一个处理器，提供给代理类使用。

2. Proxy

Proxy类就是用来创建一个代理对象的类，一般通过 newProxyInstance 方法来生成对应类的代理类。

相关源码如下：
```
// 返回代理类的一个实例，返回后的代理类可以当作被代理类使用(可使用被代理类的在接口中声明过的方法)
// loader：classloader对象，定义了由哪个类加载器对象对生成的代理类进行加载
// interfaces：接口数组，表示为代理对象提供的数组
// h：InvocationHandler对象，表示的是当动态代理对象调用方法的时候会关联到哪一个InvocationHandler对象上，并最终由其调用

@CallerSensitive
public static Object newProxyInstance(ClassLoader loader,
									  Class<?>[] interfaces,
									  InvocationHandler h)
```

3. 实例

对应步骤如下：

+	创建一个实现接口InvocationHandler的类，它必须实现invoke方法
+	创建被代理的类以及接口
+	通过Proxy的静态方法newProxyInstance(ClassLoaderloader, Class[] interfaces, InvocationHandler h)创建一个代理
+	通过代理调用方法

对应实例如下：

```
/**
 * @author zws
 * @date 2021/4/3 23:16
 * @Description JDK 代理模式
 */
public class JdkProxy {
    public static void main (String[] args) {
        Weapon realWeapon = new Gun();
        InvocationHandler handler = new MyInvocationHandler(realWeapon);
        ClassLoader classLoader = realWeapon.getClass().getClassLoader();
        Class[] clazz = realWeapon.getClass().getInterfaces();

        Weapon weapon = (Weapon) Proxy.newProxyInstance(classLoader, clazz, handler);
        System.out.println(realWeapon);
        System.out.println(weapon);

        weapon.description();
        weapon.use();
    }
}

// 调用处理器实现类
class MyInvocationHandler implements InvocationHandler {
    private Object weapon;

    public MyInvocationHandler(Object weapon) {
        this.weapon = weapon;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("在调用实际对象前，实现附加功能");
        Object res = method.invoke(weapon, args);

        System.out.println("在调用实际对象后，实现附加功能");
        return res;
    }
}

// 动态代理的接口
interface Weapon {
    void use();
    void description();
}

// 代理的实际对象
class Gun implements Weapon {

    @Override
    public void use() {
        System.out.println("use gun");
    }

    @Override
    public void description() {
        System.out.println("This is a gun");
    }
}

```


# 三、CGlib 动态代理

1. Enhancer


我们需要使用setSuperclass方法设置父类(可以是接口或者是自定义类)，相关源码如下：

```
public void setSuperclass(Class superclass) {
	// 当传入的参数为接口。
	// Enhancer允许为非接口类型创建一个Java代理。Enhancer动态创建了给定类型的子类但是拦截了所有的方法。和Proxy不一样的是，不管是接口还是类他都能正常工作。
	if (superclass != null && superclass.isInterface()) {
		this.setInterfaces(new Class[]{superclass});
		this.setContextClass(superclass);
	} else if (superclass != null && superclass.equals(Object.class)) {
		// 当传入的参数为Object类
		this.superclass = null;
	} else {
		// 当传入的是自定义类
		this.superclass = superclass;
		this.setContextClass(superclass);
	}
}


```

通过create方法可以创建对应的类，其实是调用另外一个私有的方法createHelper，相关源码如下：

```
//	用于创建类
	public Object create() {
		this.classOnly = false;
		this.argumentTypes = null;
		return this.createHelper();
	}


// 
	private Object createHelper() {
		// 验证是否配置回调类型，验证过滤器是否为空
        this.preValidate();

        Object key = KEY_FACTORY.newInstance(this.superclass != null ? this.superclass.getName() : null, ReflectUtils.getNames(this.interfaces), this.filter == ALL_ZERO ? null : new WeakCacheKey(this.filter), this.callbackTypes, this.useFactory, this.interceptDuringConstruction, this.serialVersionUID);
        this.currentKey = key;
        Object result = super.create(key);
        return result;
    }
	
// 静态代理工厂，用于创建类
private static final Enhancer.EnhancerKey KEY_FACTORY;

// 提供一个创建类的接口
public interface EnhancerKey {
	// String var1 被代理的类名
	// String[] var2 被代理的接口名
	// WeakCacheKey<CallbackFilter> var3 回调过滤器
	// Type[] var4 过滤器接口
	// boolean var5 使用工厂
	// boolean var6 使用构造器拦截器
	// Long var7 serialVersionUID 序列化ID
	Object newInstance(String var1, String[] var2, WeakCacheKey<CallbackFilter> var3, Type[] var4, boolean var5, boolean var6, Long var7);
}

```


2. 实例

对应实例如下：

```

/**
 * @author zws
 * @date 2021/4/4 14:48
 * @Description CGLIB 动态代理
 */
public class CGlibProxy {
    public static void main (String[] args) {
//  Enhancer是CGLIB的字节码增强器，可以很方便的对类进行拓展
        Enhancer enhancer = new Enhancer();
//	设置被代理的类
        enhancer.setSuperclass(Hello.class);
//        设置过滤器
        enhancer.setCallback(new HelloMethodInterceptor());
//        通过enhancer创建对象
        Hello hello = (Hello) enhancer.create();
        hello.ha();
    }
}

class Hello {
    void ha() {
        System.out.println("Hello");
    }
}
class HelloMethodInterceptor implements MethodInterceptor {

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("调用"+method.getName()+"方法前输出");
        Object o1 = methodProxy.invokeSuper(o, objects);
        System.out.println("调用"+method.getName()+"方法后输出");
        return o1;
    }
}
```

在CGLIB中，方法的调用并不是通过反射来完成的，而是直接对方法进行调用：FastClass对Class对象进行特别的处理，比如将会用数组保存method的引用，每次调用方法的时候都是通过一个index下标来保持对方法的引用，因此比JDK 动态代理速度快。但是，CGLIB无法代理被final修饰的方法。


3. FastClass机制

Cglib动态代理执行代理方法效率之所以比JDK的高是因为Cglib采用了FastClass机制，它的原理简单来说就是：为代理类和被代理类各生成一个Class，这个Class会为代理类或被代理类的方法分配一个index(int类型)。这个index当做一个入参，FastClass就可以直接定位要调用的方法直接进行调用，这样省去了反射调用，所以调用效率比JDK动态代理通过反射调用高。


# 四、SpringBoot 使用 AOP

1. AOP标准

AOP联盟标准：
![AOP 规范](/image/ssm/aop-gf.png)

AOP 实现方式有很多种，包括反射、元数据处理、程序处理、拦截器处理等。而Spring AOP的实现使用的是Java语言本身的特性，即Java Proxy代理类、拦截器技术实现。

Springboot使用 aop 可以通过maven引入相应包：

```
<!--  springboot web 模块 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
<!--  aop-->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-aop</artifactId>
	</dependency>

```

2. PCD

PCD(pointcut designators )就是SpringAOP的切点表达式。SpringAOP的PCD是完全兼容AspectJ的。

```
语法格式如下：
	execution(<修饰符模式>? <返回类型模式> <方法名模式>(<参数模式>) <异常模式>?)

除了返回类型模式、方法名模式和参数模式外，其它项都是可选的。

-- 所有公有方法的执行
execution(public * *(..))

-- 设置指定格式开头的方法
execution(* test*(..))

-- AccountService接口下的所有方法的执行
execution(* com.service.AccountService.*(..))

-- 匹配com.xyz.service包下的所有类的所有方法（不含子包）
within(com.xyz.service.*)

```


3. aop 实例：拦截控制器

```

/**
 * @author zws
 * @date 2021/4/4 16:52
 * @Description 控制器切面
 */

@Aspect
@Component
public class ControllerAop {
    /**
     * 指定切点
     * 匹配 com.wsz.tool.rabbitmq.controller包及其子包下的所有类的所有方法
     */
//    @Pointcut("execution(public * com.wsz.tool.rabbitmq.controller..*.*(..))")
    @Pointcut("execution(public * com.wsz.tool.rabbitmq.controller.*Controller.*(..))")
    public void log() {
        System.out.println("日志输出：");
    }

    @Before("log()")
    public void doBefore(JoinPoint joinPoint) {
        System.out.println("前置通知");
//        目标方法的参数信息
        Object[] objects = joinPoint.getArgs();
        Signature signature = joinPoint.getSignature();

//        代理方法名

        System.out.println("代理方法名："+signature.getName());
        System.out.println("代理类名："+signature.getDeclaringTypeName());
        signature.getDeclaringType();
        MethodSignature ms = (MethodSignature) signature;
        System.out.println("参数名："+ Arrays.toString(ms.getParameterNames()));
        System.out.println("参数值："+ Arrays.toString(joinPoint.getArgs()));


// 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest req = attributes.getRequest();
        // 记录下请求内容
        System.out.println("请求URL : " + req.getRequestURL().toString());
        System.out.println("HTTP_METHOD : " + req.getMethod());
        System.out.println("IP : " + req.getRemoteAddr());
        System.out.println("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
    }

//    后置异常通知
    @AfterThrowing("log()")
    public void throwss(JoinPoint jp) {
        System.out.println("异常");
    }
    @After("log()")
    public void after (JoinPoint jp) {
        System.out.println("后置通知");
    }


}

```

4. aop实例：拦截自定义注解

```

/**
 * @author zws
 * @date 2021/4/4 20:12
 * @Description 自定义Log注解
 */

/*
*注解的作用目标：
*　@Target(ElementType.TYPE)                      // 接口、类、枚举、注解
*　@Target(ElementType.FIELD)                     // 字段、枚举的常量
*　@Target(ElementType.METHOD)                 // 方法
*  @Target(ElementType.PARAMETER)            // 方法参数
*　@Target(ElementType.CONSTRUCTOR)       // 构造函数
*　@Target(ElementType.LOCAL_VARIABLE)   // 局部变量
*　@Target(ElementType.ANNOTATION_TYPE) // 注解
*　@Target(ElementType.PACKAGE)               // 包
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface MyLog {
    String value();
}


/**
 * @author zws
 * @date 2021/4/4 20:20
 * @Description 自定义注解使用
 */

@Service
public class TestService {

    @MyLog("TestService")
    public void test() {
        System.out.println("test");
    }
}


/**
 * @author zws
 * @date 2021/4/4 20:13
 * @Description AOP拦截自定义注解
 *
 */

@Aspect
@Component
public class MyLogAop {
    @Pointcut("execution(public * com.wsz.tool.rabbitmq.aop.MyLog.*(..))")
    public void log() {

    }
    /**
     * @Pointcut与@Around的区别：@Pointcut与@Around可以使用相同的PCD表达式，@Pointcut定义的可以重复使用，而@Around不可以
     */
	 
    @Around(value = "@annotation(com.wsz.tool.rabbitmq.aop.MyLog)")
    public void before(JoinPoint point) {
        System.out.println("MyLogAop前置通知");
    }
}
```

5. aop实例：日志
[aop切面日志新增json参数打印和高并发场景](https://github.com/xkcoding/spring-boot-demo/tree/master/demo-log-aop)

```
@Aspect
@Component
@Slf4j
public class AopLog {
    /**
     * 切入点
     */
    @Pointcut("execution(public * com.xkcoding.log.aop.controller.*Controller.*(..))")
    public void log() {

    }

    /**
     * 环绕操作
     *
     * @param point 切入点
     * @return 原方法返回值
     * @throws Throwable 异常信息
     */
    @Around("log()")
    public Object aroundLog(ProceedingJoinPoint point) throws Throwable {

        // 开始打印请求日志
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = Objects.requireNonNull(attributes).getRequest();

        // 打印请求相关参数
        long startTime = System.currentTimeMillis();
        Object result = point.proceed();
        String header = request.getHeader("User-Agent");
        UserAgent userAgent = UserAgent.parseUserAgentString(header);

        final Log l = Log.builder()
            .threadId(Long.toString(Thread.currentThread().getId()))
            .threadName(Thread.currentThread().getName())
            .ip(getIp(request))
            .url(request.getRequestURL().toString())
            .classMethod(String.format("%s.%s", point.getSignature().getDeclaringTypeName(),
                point.getSignature().getName()))
            .httpMethod(request.getMethod())
            .requestParams(getNameAndValue(point))
            .result(result)
            .timeCost(System.currentTimeMillis() - startTime)
            .userAgent(header)
            .browser(userAgent.getBrowser().toString())
            .os(userAgent.getOperatingSystem().toString()).build();

        log.info("Request Log Info : {}", JSONUtil.toJsonStr(l));

        return result;
    }

    /**
     *  获取方法参数名和参数值
     * @param joinPoint
     * @return
     */
    private Map<String, Object> getNameAndValue(ProceedingJoinPoint joinPoint) {

        final Signature signature = joinPoint.getSignature();
        MethodSignature methodSignature = (MethodSignature) signature;
        final String[] names = methodSignature.getParameterNames();
        final Object[] args = joinPoint.getArgs();

        if (ArrayUtil.isEmpty(names) || ArrayUtil.isEmpty(args)) {
            return Collections.emptyMap();
        }
        if (names.length != args.length) {
            log.warn("{}方法参数名和参数值数量不一致", methodSignature.getName());
            return Collections.emptyMap();
        }
        Map<String, Object> map = Maps.newHashMap();
        for (int i = 0; i < names.length; i++) {
            map.put(names[i], args[i]);
        }
        return map;
    }

    private static final String UNKNOWN = "unknown";

    /**
     * 获取ip地址
     */
    public static String getIp(HttpServletRequest request) {
        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        String comma = ",";
        String localhost = "127.0.0.1";
        if (ip.contains(comma)) {
            ip = ip.split(",")[0];
        }
        if (localhost.equals(ip)) {
            // 获取本机真正的ip地址
            try {
                ip = InetAddress.getLocalHost().getHostAddress();
            } catch (UnknownHostException e) {
                log.error(e.getMessage(), e);
            }
        }
        return ip;
    }

    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    static class Log {
        // 线程id
        private String threadId;
        // 线程名称
        private String threadName;
        // ip
        private String ip;
        // url
        private String url;
        // http方法 GET POST PUT DELETE PATCH
        private String httpMethod;
        // 类方法
        private String classMethod;
        // 请求参数
        private Object requestParams;
        // 返回参数
        private Object result;
        // 接口耗时
        private Long timeCost;
        // 操作系统
        private String os;
        // 浏览器
        private String browser;
        // user-agent
        private String userAgent;
    }
}
```

6. 如下代码，`@Transactional`有没有生效？

```
@Service
public class OrderService {
 
	private void insert() {
		insertOrder();
	}
	 
	@Transactional
	public void insertOrder() {
		//SQL操作
	}
}
```

没有，`@Transactional`底层基于AOP实现，自身调用是无法生效的。因为在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。insert()是由被代理对象调用的，而它内调的方法insertOrder() 就不会由代理对象调用，也就无法被添加事务。




# 五、参考文章
> [Spring AOP——Spring 中面向切面编程](https://www.cnblogs.com/joy99/p/10941543.html)
[Spring AOP实现原理简介](https://blog.csdn.net/wyl6019/article/details/80136000)