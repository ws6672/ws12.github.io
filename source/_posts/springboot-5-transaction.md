---
title: springboot（五）事务管理
date: 2020-08-19 15:03:18
tags: [javaweb]
---



### 一、基础知识

***相关的业务背景***

常用的业务背景是取钱，通过ATM取钱需要判断你账号所剩余额以及ATM中现金的数目。如果银行扣了钱但ATM吐钱失败，那么用户就损失了金钱；如果ATM取钱成功，但银行扣钱失败，那么银行就损失了金钱。所以，这两个步骤需要同时成功或者同时失败。诸如此类的场景，就是事务的应用。

***事务的特性（ACID）***

+	原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不执行。
+	一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
+	隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其它事务隔离开来，防止数据损坏。
+	持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。


### 二、核心接口

Spring MVC的事务配置如下：

```
 <beans:bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">  //事务管理器
    <beans:property name="dataSource" ref="dataSource" />   // JDBC 配置
    <beans:property name="entityManagerFactory" ref="entityManagerFactory" />  // 实体类管理器
</beans:bean>

<!-- 声明使用注解式事务 -->
<!-- 在Spring内部启用@Transactional来进行事务管理 -->
<tx:annotation-driven transaction-manager="transactionManager" />
```

 Spring Boot内部提供的事务管理器是根据`@AutoConfigure`来进行决定的，当我们使用spring-boot-starter-jdbc或spring-boot-starter-data-jpa依赖的时候，框架会自动分别注入DataSourceTransactionManager或JpaTransactionManager。这两个事务管理器都实现了Spring中提供的PlatformTransactionManager接口，它是Spring的事务核心接口。

***类结构***

![事务管理器](/image/springboot/springboot-transaction.png)

`org.springframework.transaction.PlatformTransactionManager`是一个事务管理器接口，JPA、Hibernate、Mybatis等框架通过实现这个接口来构建自身的事务管理器。

```
package org.springframework.transaction;
import org.springframework.lang.Nullable;
public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
    void commit(TransactionStatus var1) throws TransactionException;
    void rollback(TransactionStatus var1) throws TransactionException;
}
```

TransactionStatus接口表示事务的状态，比如事务是否是一个刚构造的事务，事务是否已经执行完成等状态。

### 三、事务使用

***事务的管理方式***

Spring支持编程式事务管理和声明式事务管理两种方式：
1. 编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager，Spring推荐使用TransactionTemplate。
2. 声明式事务管理在底层是建立在AOP之上的。其本质是在方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。

声明式事务管理要优于编程式事务管理，这正是Spring倡导的非侵入式的开发方式。声明式事务避免了代码污染，但是细粒度比编程式事务管理低，无法到代码块级别。但是，我们可以把需要进行事务管理的代码块封装成相应的方法，扬长避短。


声明式事务的两种使用方式：
1. 配置`<tx:advice>`
2. (推荐)注解`@Transactional`

***事务配置***

引入Maven依赖
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

Springboot 是 约定大于配置，当引入该依赖后会自动默认分别注入DataSourceTransactionManager或JpaTransactionManager，这两个是常用的事务管理器；之后，可以通过`@Transactional` 启用事务。

`@Transactional` 可以在类或者方法中使用，类中使用表示类中所有public方法都启用了注解。类中的注解会重载方法的注解。


*** @Transactional 源码***

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
// 事务管理器
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

// 传播行为
    Propagation propagation() default Propagation.REQUIRED;
// 隔离级别
    Isolation isolation() default Isolation.DEFAULT;
// 超时
    int timeout() default -1;
// 只读
    boolean readOnly() default false;
// 回滚规则
    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}
```


### 四、事务属性

事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。事务属性包含了5个方面：

+	隔离规则
+	传播行为
+	事务超时
+	回滚规则
+	是否只读

这些属性都可以通过 事务注解`@Transactional`进行配置。

***隔离级别***

事务具有四个隔离级别
+	ISOLATION_DEFAULT（默认）
	+	默认值，使用后端数据库默认的隔离级别
+	ISOLATION_READ_UNCOMMITTED（读未提交）
	+	事务B可以读取事务A未提交的数据——会导致`脏读`
+	ISOLATION_READ_COMMITTED（读已提交）
	+	事务A提交了数据，事务B才能读取，会导致`不可重复读`
	+	`不可重复读`是指：事务B内两次读取事务A的数据，结果不一样；主要针对 update、delete操作
	+	`Oracle默认隔离级别`
+	ISOLATION_REPEATABLE_READ（可重复读）
	+	事务A提交了数据，事务B也不能读取；但是，会导致`幻读`；INNODB不会导致幻读。
	+	`幻读`是指：前后两次读取数据记录数不一致；主要针对insert操作。
	+	`mysql的默认级别`
+	ISOLATION_SERIALIZABLE（串行化 serializable）
	+	当事务A运行时，事务B排队等待，速度慢，并发差

注解配置
```
@Transactional (isolation = Isolation.DEFAULT)
public class BookService {
}	
```	

***传播行为***

事务传播行为（propagation behavior）用来描述由某一个事务传播行为修饰的方法被嵌套进另一个方法的时候，事务如何传播。

传播行为定义了事务范围，何时触发事务，是否暂停现有事务，或者在调用方法是如果没有事务则失败等等。


Spring Boot的事务传播行为常量被封装在枚举类Propagation，枚举值取自接口TransactionDefinition。该常量用于衡量A方法调用B方法时，B方法的事务策略，取值如下：

+	PROPAGATION_REQUIRED
	+	如果当前存在事务，则加入该事务；否则，新建一个事务。这是默认值
+	PROPAGATION_REQUIRES_NEW
	+	新建事务，如果当前存在事务，则挂起当前事务；另起一个事务，将与它的父事务相互独立
+	PROPAGATION_SUPPORTS
	+	如果当前存在事务，则加入该事务；否则，以非事务的方式继续运行
+	PROPAGATION_NOT_SUPPORTED
	+	以非事务方式运行，如果当前存在事务，则挂起当前事务
+	PROPAGATION_NEVER
	+	以非事务方式运行，如果当前存在事务，则抛异常
+	PROPAGATION_MANDATORY(mandatory)
	+	如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常
+	PROPAGATION_NESTED
	+	如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于PROPAGATION_REQUIRED；如果另起一个事务，那么依赖于父事务，提交要等待父事务提交

注解配置
```
@Transactional(propagation = Propagation.REQUIRES_NEW)
public class BookService {
}
```

***事务超时***

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。
默认设置为底层事务系统的超时值，如果底层数据库事务系统没有设置超时值，那么就是none，没有超时限制。

注解配置
```
@Transactional(timeout = 60)
public class BookService {
}
```

***事务只读属性***
只读事务用于只读取但不修改数据的情形，只读事务用于特定情景下的优化，比如使用Hibernate的时候。默认为读写事务。 

注解配置
```
@Transactional(readOnly = false)
public class BookService {
}
```

***事务回滚规则***

Spring事务管理器回滚一个事务的推荐方法是在当前事务的上下文内抛出异常。Spring事务管理器会捕捉任何未处理的异常，然后依据规则决定是否回滚抛出异常的事务。

默认配置下，Spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。

注解中有四个属性是和回滚相关的：
+	rollbackFor
+	rollbackForClassName
+	noRollbackFor
+	noRollbackForClassName

注解配置如下
```
//@Transactional(rollbackFor = Exception.class) 
//@Transactional(noRollbackFor = Exception.class)
//@Transactional(rollbackForClassName = "java.lang.Exception")
@Transactional(noRollbackForClassName = "java.lang.Exception")
public class BookService {
}
```

***调用事务的步骤***

1. 激活事务 connection.setAutoCommit(false)
2. 执行业务方法 
3. 结束事务 commit or rollback (Spring默认取决于是否抛出runtime异常)；如果抛出runtime exception 并在你的业务方法中没有catch到的话，事务会回滚。

> 如果某个业务方法被一个try catch整个包裹起来，那么这个业务方法也就等于脱离了Spring事务的管理，因为没有任何异常会从业务方法中抛出,就不会导致异常。
 
### 五、参考资料
> [Spring Boot事务管理（上）](https://www.cnblogs.com/east7/p/10585641.html)
[Spring Boot事务管理（中）](https://www.cnblogs.com/east7/p/10585699.html)
[Spring Boot事务管理（下）](https://www.cnblogs.com/east7/p/10585724.html)

