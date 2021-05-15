---
title: springboot（四）jpa
date: 2020-08-18 12:52:25
tags: [javaweb]
---


### 1. 概述

> JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。 

***jpa和mybatis的区别***

+	JPA是一种标准，而MyBatis只是一种ORM框架
+	jpa是对象与对象之间的映射，而mybatis是对象和结果集的映射。
+	jpa移植性比较好，不用关心用什么数据库，因为mybatis自由写sql语句，所以当项目移植的时候还需要改sql


***生命周期***

[/image/springboot/jpa-life.png]

实体生命周期有四种状态:
New：瞬时对象，尚未有id，还未和Persistence Context建立关联的对象。
Managed：持久化受管对象，有id值，已经和Persistence Context建立了关联的对象。
Datached：游离态离线对象，有id值，但没有和Persistence Context建立关联的对象。
Removed：删除的对象，有id值，尚且和Persistence Context有关联，但是已经准备好从数据库中删除

Managed状态下的数据保存，更新以及删除数据下的Removed状态，数据都不会立即更新到数据库，只有当你事务提交或者em.flush()，才会立即更新到数据库。
Datached的状态，可以调用em.merge()方法，这个方法会根据实体类的id来更新数据库数据

+	Managed 作为Java对象存在于实体管理器中；也存在于数据库中
+	Removed 作为Java对象存在于实体管理器中；
+	New 只作为java对象存在；
+	Detached 不存在；

### 2. 配置JPA

***maven 依赖***
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```

***YML配置***

```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/toolweb?serverTimezone=UTC&useSSL=false
    username: root
    password: root
  jpa:
    database: mysql
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: true
    hibernate:
      ddl-auto: create
```

1. `spring.datasource.*`的配置是通用的JDBC配置。

2. `jpa`配置的含义如下：

`spring.jpa.database-platform`：配置数据库引擎
`spring.jpa.show-sql`：命令行输出sql语句(true|false)
`spring.jpa.hibernate.ddl-auto`：
	+	create [删除-创建-操作]
		+	每次加载 hibernate 时都会删除上一次的生成的表，然后根据你的 model 类再重新来生成新表
	+	create-drop [删除-创建-操作-再删除]
		+	每次加载 hibernate 时根据 model 类生成表，但是 sessionFactory 一关闭，表就自动删除。
	+	update [没表-创建-操作 | 有表-更新没有的属性列-操作]
		+	第一次加载 hibernate 时根据 model 类会自动建立起表的结构（前提是先建立好数据库），以后加载 hibernate 时根据 model 类自动更新表结构
		+	表结构不会再应用部署后构建，而是在被调用后才构建
	+	validate[验证，验证失败，启动失败]
		+	不会创建新表，但是会插入新值
		

### 3 注解


##### 3.1 主键相关


***@id***

标注用于声明一个实体类的属性映射为数据库的主键列。该属性通常置于属性声明语句之前，可与声明数据同行，也可卸载单独行上。

`@Id` 标注也可置于属性的getter方法之前。

例如
```
import javax.persistence.Id

	@Id
	private Integer id;
```



***@GeneratedValue***

`@GeneratedValue`用于定义生成器。

`@GeneratedValue`有两个主要的属性strategy和generator。

strategy用于定义主键生成策略。策略被定义在枚举类`GenerationType`中, 取值如下

1. GenerationType.TABLE：  	使用一个特定的数据库表格来保存主键,持久化引擎通过关系数据库的一张特定的表格来生成主键。 好处就是不依赖于外部环境和数据库的具体实现，完成表与数据库的分离。该策略一般与另外一个注解`@TableGenerator`一起使用。例如

```
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "USER_GENERATE")
    @TableGenerator(name = "ID_GENERATE_GENERATE",
    pkColumnValue = "user_seq",
    table = "user_generate")
    @Id
    private Long id;
```


2. GenerationType.SEQUENCE：类似Oracle的数据库不支持主键自增长，而是提供了"序列(sequence)"的机制生成主键

```
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "menuSeq")
@SequenceGenerator(name = "menuSeq", initialValue = 1, allocationSize = 1, sequenceName = "MENU_SEQUENCE")
private Integer id;
```

3. GenerationType.IDENTITY
主键自增长策略，例如
```
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Integer id;
```

4. GenerationType.AUTO

把主键生成策略交给持久化引擎(persistence engine),持久化引擎会根据数据库在以上三种主键生成策略中选择其中一种。此种主键生成策略比较常用,由于JPA默认的生成策略就是GenerationType.AUTO,所以使用此种策略时.可以显式的指定@GeneratedValue(strategy = GenerationType.AUTO)也可以直接@GeneratedValue

```
@GeneratedValue(strategy = GenerationType.AUTO)
private Integer id;

@GeneratedValue
private Integer id;
```

***主键生成策略***

JPA 提供了四种主键生成器，如下
+	AUTO：由JPA根据数据库自行决定算法（`@GeneratedValue(strategy = GenerationType.AUTO)`）
+	IDENTITY：数据库自增列提供算法（`@GeneratedValue(strategy = GenerationType.IDENTITY)`）
+	SEQUENCE：由SEQUENCE对象提供算法（配合注解`@SequenceGenerator` 使用）
+	TABLE：由JPA提供者通过创建数据库表来记录生成的主键值（配合注解 `@TableGenerator` 使用）



##### 3.2 @TableGenerator(表生成器)

表生成器是通用性最强的 JPA 主键生成器。这种方法生成主键的策略可以适用于任何数据库，不必担心不同数据库不兼容造成的问题。

常用属性如下
+	name 主键生成策略的名称，被引用在`@GeneratedValue`中设置的“generator”值中。
+	table 生成策略所持久化的表名
+	catalog 属性和 schema 具体指定表所在的目录名或是数据库名
+	pkColumnName 保存主键名的字段
+	valueColumnName 保存主键值的字段
+	pkColumnValue 表里名字字段对应的值
+	allocationSize 每次增长的大小
+	initialValue 初始化值

***实例 ***

实体类定义
```
import javax.persistence.*;

@Entity
public class Book {

    @GeneratedValue(strategy = GenerationType.TABLE, generator = "ID_GENERATE")
	// 这个定义在数据库中生成了id_generate的表，用于存放主键
	// 表 id_generate 中有两个列：generate_name、generate_value，它们的值分别是book_seq、1（即表主键标志以及主键增长后的值）
	// 每次插入一条数据，表 id_generate 中，generate_name=book_seq的数据的 generate_value列会自增长
    @TableGenerator(name = "ID_GENERATE",
            table = "id_generate",
            initialValue = 1,
            allocationSize = 1,
            pkColumnName = "generate_name",
            valueColumnName = "generate_value",
            pkColumnValue = "book_seq")
    @Id
    private Long id;

    @Column(name = "book_name")
    private String book_name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getBook_name() {
        return book_name;
    }

    public void setBook_name(String book_name) {
        this.book_name = book_name;
    }
}
```

JPA Repository定义
```
@Repository
public interface BookRepository extends JpaRepository<Book, Integer> {
}
```

应用启动器需要添加扫描包的注解
```
@SpringBootApplication(scanBasePackages = "com.springboot.example")
@EntityScan
public class ExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```

添加数据
```
 @Autowired
private BookRepository bookRepository;

public void addBook() {
	Book book = new Book();
	book.setBook_name("哈利波特");
	bookRepository.save(book);
}
```

*** No identifier specified for entity***

没有`@Id`, 查看了一下是导入包不正确，应当导入`javax.persistence.Id`


##### 3.3 类注解

*** @Entity***

标注用于实体类声明语句之前，指出该Java类为实体类，将映射到指定的数据库表。

```
@Entity
public class User {
}
```
 

*** @Table***

当实体类与其映射的数据库表名`不同名`时，需要使用`@Table`标注说明，该注解与`@Entity`标注并列使用，置于实体类声明语句之前，可写于单独语句行，也可与声明数据同行。

`@Table`标注的常用选项是name，用于指明数据库的表名。

`@Table`标注还有两个可选项catalog和schema用于设置表所属的数据库目录或模式，通常为数据库名。

*** @MappedSuperClass***

1. `@MappedSuperclass`注解使用在父类上面，是用来标识父类的作用

2. `@MappedSuperclass`标识的类表示其不能映射到数据库表，因为其不是一个完整的实体类，但是它所拥有的属性能够映射在     其子类对用的数据库表中

3. `@MappedSuperclass`标识得类不能再有`@Entity`或`@Table`注解  但是可以使用`@Id` 和`@Column`注解

```
/*
* 泛化:继承关系
* 告诉JPA这是所有类都继承的父类
* */
@MappedSuperclass
public class BaseDomain {
    @Id
    @GeneratedValue
    protected Long id;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
}

@Entity
@Table(name = "department")
public class Department extends BaseDomain {
	private String name;
    public String getName() {
		return name;
	}
 
	public void setName(String name) {
		this.name = name;
	}	
}
```



*** @NoRepositoryBean***

在做项目时创建对象的功能会交给Spring去管理在扫描Reposytory层包时会扫描到BaseReposytory接口 ;所有对象类接口都会继承此接口 为了告诉JPA不要创建对应接口的bean对象 就在类上加注解`@NoRepositoryBean`

这样spring容器中就不会有BaseReposytory接口的bean对象

```
@NoRepositoryBean //告诉JPA不要创建对应接口的bean对象
public interface BaseReposittory <T,ID extends Serializable> extends 	JpaRepository<T,ID>,JpaSpecificationExecutor<T> {
}
```


***@Repository***

和`@Controller`、`@Service`、`@Component`的作用差不多，都是把对象交给spring管理。`@Repository`用在持久层的接口上，这个注解是将接口的一个实现类交给spring管理。


	
##### 3.3 列相关注解



*** @Column***

当实体的属性与其映射的数据库表的列不同名时需要使用`@Column`标注说明，该属性通常置于实体的属性声明语句之前，还可与`@Id`标注一起使用。

`@Column`标注的常量属性是name，用于设置映射数据库表的列名。此外，该注解还包含其他多个属性，比如：unique，nullable，length等。

`@Column`标注的columnDefinition属性：表示该字段在数据中的实际类型

*** @Transient 和 @Basic***

表示该属性并非一个到数据库表的字段的映射，ORM框架将忽略该属性。

如果一个属性并非数据库的字段映射，就务必将其标示为`@Transient`，否则，ORM框架默认其注解为`@Basic`。






*** @JsonIgnore***

作用是在json序列化时将java bean中的一些属性忽略掉，序列化和反序列化都受影响。一般标记在属性或者方法上，返回的json数据即不包含该属性。例如：

```
public class HistoryOrderBean {

    //基本属性字段
    private String insurantName;
    private String insuranceName;
    private String insurancePrice;
    private String insurancePicture;
    private String insuranceLimit;

    //快照属性字段
    @JsonIgnore
    private String goodsInfo;      //快照基本信息
    @JsonIgnore  
    private String extendsInfo;    //快照扩展属性信息

}
```

注解失败：
使用的是fastJson，尝试使用对应的注解来忽略字段，注解为`@JSONField(serialize = false)`。



##### 3.4 关联注解



***单向关联与双向关联***

单向关联：两者之间的关系是从属关系，一部分依附另外一部分。
双向关联：两者之间的关系是平等的，都是独立个体。


1. 以一对一的关系举例
+	业务场景一（单向关联）：假设一个人只拥有一只宠物，我们可以通过人去访问宠物，但是很难通过走失的动物找到人。在这个业务中，实体间的关系是一对一，且是单向的。
+	业务场景二（双向关联）：我国法律实行一夫一妻制。所以，一个丈夫只能有一个妻子，一个妻子也只有一个丈夫。这里就存在双向的一对一关系


2. 以一对多的关系举例

+	业务场景一（单向关联）：假设一个人可以拥有多只宠物，我们可以通过人找到所有宠物，但是很难通过走失的动物找到人。在这个业务中，实体间的关系是一对多，且是单向的。
+	业务场景二（双向关联）：一个老师有多个学生。我们可以通过老师去找到学生，也可以通过学生找到老师。所以，这里的联系是双向的。

具体是单向还是双向的，要具体业务具体分析，从业务出发。


***相关的注解***

+	`@OneToOne`(一对一关系)：一人属于一个班级
+	`@OneToMany`(一对多关系)：每个出版社出版很多书，但是每本书名只能出自一个出版社。
+	`@ManyToOne`（多对一关系）：一根头发属于一个人，这个人有多根头发
+	`@ManyToMany`(多对多关系)：一个学生有多门课程，一个门课程有多个学生参加
+	`@JoinColumn` 表示外键

*** @OneToOne***

`@OneToOne`与`@JoinColumn`结合使用，表示会在源实体（Source Entity，即声明该注解的实体类中创建外键，进行级联）

`@JoinColumn（设置外键）`注释的是保存表与表之间关系的字段，它要标注在实体属性上(相当于外键)。与`@Column`标记一样，是用于注释表中的字段的。它的属性与`@Column`属性有很多相同之处
 

注解源码

```
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface OneToOne {
    Class targetEntity() default void.class;
    CascadeType[] cascade() default {};
    FetchType fetch() default FetchType.EAGER;
    boolean optional() default true;
    String mappedBy() default "";
    boolean orphanRemoval() default false;
}
```

相关属性说明

1. targetEntity(关联的实体)

2. cascade（级联操作权限）

+	CascadeType.PERSIST：级联持久化，A对象新增了，会跟着新增B对象。B对象存在则抛异常
+	CascadeType.MERGE：级联合并，指A类新增或者变化，会级联B对象
+	CascadeType.REMOVE：级联删除，A类删除了会跟着删除B类
+	CascadeType.DETACH：级联游离，支持单独删除带B对象外键的A对象
+	CascadeType.REFRESH：级联刷新，假设场景 有一个订单,订单里面关联了许多商品,这个订单可以被很多人操作,那么这个时候A对此订单和关联的商品进行了修改,与此同时,B也进行了相同的操作,但是B先一步比A保存了数据,那么当A保存数据的时候,就需要先刷新订单信息及关联的商品信息后,再将订单及商品保存
+	CascadeType.ALL：拥有以上所有级联操作权限。

3. fetch（设置关联对象的加载方式）

+	FetchType.EAGER：立即加载，比如在加载学生对象信息时，立刻加载学生的成绩信息
+	FetchType.LAZy：延迟加载，需要用到的时候再加载

4. optional（表示该属性是否允许为null,默认为true）

5. mappedBy（关系维护，避免生成中间表）：用来标注拥有这种关系的字段。 除非关系是单向的，否则是必需的。

+	mappedBy 只有在双向关联时,才会使用这个属性
+	```
   @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy = "parent")
    private List<ListEntity> list = new ArrayList<ListEntity>();
	// mappedBy= "parent" 表示在ListEntity类中的 parent 属性来维护关系，这个名称必须和ListEntity中的parent属性名称完全一致才行(OneToMany必须写mappedBy，不然会多生成一张没用的中间表)
```

6. orphanRemoval

+	jpa 中 orphanRemoval 属性，如果为 true 的话，想要删掉子集合数据，那么调用子集合list 的 clear 方法清空，并且断关系可以直接在数据库中删除子集合数据， 不能直接设置 为null，否则抛出异常
+	如果没有该属性，调用子集合list 的 clear 方法清空，并且断关系则在数据库中把 子表数据中保存的主表id 设置为空,断开关系
+	而cascade 是总开关，如果 这里没有设置 CascadeType.all  或者 delete ，那么就算 orphanRemoveal 设置为 true 也无法执行删除

7. cascade 和 orphanremoval 的关系说明

+	首先二者的作用范围不一样，cascade的作用范围是数据库，当cascade属性设置了delete时，当删除级联关系中的子集时，顺便也会将数据库中对应的数据删除。
+	orphanremoval属性的作用范围仅仅是java应用代码中，做级联删除的操作也只适用于java实体代码范畴，它可以清楚javabean的级联关系，但并不能影响数据库的数据，只要cascade不点头是无法删除掉数据库的数据的。


实例1——单向

```
@Entity
public class People {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;

    @OneToOne
    @JoinColumn(name = "petId")
	// 该表会生成一个 pet_id的外键，维持两表的关系
    private Pet pet;
	...
}

@Entity
public class Pet {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;
	...
}
```

实例2——双向

数据库中Wife这张表找不到关联到Husband的外键，因为我们定义关系的时候，规定了关系的维护端是Husband。所以，对于数据库来说，双方的联系的单向的。当配置mappedBy后使关系成为双向，这一切对于数据库来说是透明的，一切都是JPA在处理。

```
@Entity
public class Husband {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;

    @OneToOne
    @JoinColumn(name = "wife_id")
    private Wife wife;
}

@Entity
public class Wife {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;

//    Wife实体是被 Hhusband实体中的外键“wife”所维护。
    @OneToOne(mappedBy = "wife")
    private Husband husband;
}
```

*** @OneToMany、@ManyToOne***

`@OneToMany` 用于维持一对多的联系。

源码

```
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface OneToMany {
    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    FetchType fetch() default FetchType.LAZY;

    String mappedBy() default "";

    boolean orphanRemoval() default false;
}
```

实例1——单向

```
@Entity
public class School {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long Id;
    private String name;
    @OneToMany
    @JoinColumn(name = "schoolId")
    private Set<Student> students;
}

@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long Id;
    private String name;
}
```

实例2——双向
一对多的双向联系需要同时使用“@OneToMany、@ManyToOne”两个注解。

```
@Entity
public class Grp {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long Id;
    private String name;

	// 需要配置mappedBy，否则会被识别为多对多的关系
    @OneToMany(mappedBy = "group")
    Set<User> users;
}

@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long Id;
    private String name;

    @ManyToOne
    @JoinColumn(name="gid")
    private Grp group;
}
```
 

***@ManyToMany、@JoinTable***

源码
```
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ManyToMany {
    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    FetchType fetch() default FetchType.LAZY;

    String mappedBy() default "";
}
```

manyToMany需要和 JoinTable 表结合使用，ManyToMany总是使用中间关系连接表来存储关系。如果两个Vo都定义了ManyToMany的话，因为单向关系，会生成有2个中间表。所以需要改造成双向关系，使其只存在一个中间表。

> 注：一方不需要mappedBy属性，一方需要。

实例
```
@Entity
@Table
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long Id;
    private String name;

    @ManyToMany
    @JoinTable(name = "stu_course",
            joinColumns = @JoinColumn(name = "stu_id"),
            inverseJoinColumns = @JoinColumn(name="course_id"))
	// 创建了stu_course的中间表，有两个字段stu_id、course_id
    private List<Course> courses;
	
}

@Entity
public class Course {
    @javax.persistence.Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long Id;
    private String name;

	//避免创建多个中间表
    @ManyToMany(mappedBy = "courses")
    List<Student> students;
}
```




[JPA mappedBy属性](https://my.oschina.net/bigyuan/blog/336515)
[JPA概念解析：CascadeType](https://www.jianshu.com/p/e8caafce5445)
[https://blog.csdn.net/yingxiake/article/details/50968059](https://blog.csdn.net/yingxiake/article/details/50968059)