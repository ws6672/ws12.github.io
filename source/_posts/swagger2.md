---
title: swagger2的使用
date: 2021-06-15 16:43:10
tags: [springboot]
---

# 一、基础

API功能的发展是不可避免的，但维护API文档的头痛并非必须如此。Swagger工具可以帮助您完成生成和维护API文档的工作，确保您的文档在API发展过程中保持最新状态。在Swagger团队加入SmartBear并且该规范于2015年被捐赠给OpenAPI计划之后，Swagger规范被重命名为OpenAPI规范（OAS），已成为定义RESTful API的因素标准。

> OAS合同描述了API的作用，它的请求参数和响应对象，所有这些都没有任何代码实现的指示。使用OAS定义的Web服务可以相互通信，而不管它们内置的语言，因为OAS是语言>无关的和机器可读的。

而Swagger的优势是可以生成API文档、在线对接口进行测试。

Swagger组成：
+	swagger
	+	Swagger Editor：基于浏览器的编辑器，我们可以使用它编写我们 OpenAPI 规范。
	+	Swagger UI：它会将我们编写的 OpenAPI 规范呈现为交互式的 API 文档，后文我将使用浏览器来查看并且操作我们的 Rest API。
	+	Swagger Codegen：它可以通过为 OpenAPI（以前称为 Swagger）规范定义的任何 API 生成服务器存根和客户端 SDK 来简化构建过程。


使用步骤
+	引入maven包
+	编写接口文件
+	使用**knife4j**展示，是以前使用的`swagger-ui`升级版。


# 二、配置

1. 导包

```xml
<dependency>
	<groupId>com.github.xiaoymin</groupId>
	<artifactId>knife4j-spring-ui</artifactId>
	<version>3.0.2</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.4.0</version>
</dependency>
```

2. 配置

定义配置类，定义全局状态。`HttpStatus`是定义的全局状态类，存放了响应状态码。通过`globalResponseMessage`方法可以将它设置为全局的响应信息。

```java
/**
 * @Author zws
 * @Description swagger配置类
 * @Date 17:41 2021/5/28
 **/
@Configuration
@EnableSwagger2
public class SwaggerConf {
    static  List responseList = new ArrayList<>();
    static {
        Arrays.stream(
                HttpStatus.values()).forEach(
                        status -> responseList.add(
                                new ResponseMessageBuilder().code(status.getCode())
                                        .message(status.getMessage())
                                        .responseModel(
                                                new ModelRef(status.getMessage()))
                                        .build()));
    }

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .globalResponseMessage(RequestMethod.GET, responseList)
                .globalResponseMessage(RequestMethod.POST, responseList)
                .globalResponseMessage(RequestMethod.PUT, responseList)
                .globalResponseMessage(RequestMethod.DELETE, responseList)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.simtek.cosimcloud.controller"))
                .paths(PathSelectors.any())
                .build()
                .pathMapping("/");
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Cosim Cloud 接口文档")
                .description("RestFul风格, 创建人：zws")
                .termsOfServiceUrl("https://github.com/cicadasmile")
                .version("1.0")
                .build();
    }

}
```

在实际使用中，一般会有多个环境，例如开发环境、正式环境；在正式环境中就不应当展示文档界面。我们可以通过以下的方式来切换环境：
+	在正式环境注释掉 swagger配置类的注解
+	从配置类获取配置

两种方法中，第二种虽然有一定工作量，但是效果更好，可以快速切换，无需二次编译。如application.yml：

```yml
swagger:
  enable: true
```

3. 处理资源拦截问题

在IDEA中如此配置后，还是无法访问，搜索了资料，发现是资源被拦截，定义如下类：
```java
@Configuration
public class IntercpetorConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 设置swagger静态资源访问
        registry.addResourceHandler("swagger-ui.html").addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("doc.html").addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```

4. 测试

然后，通过`http://localhost:8080/doc.html`即可访问了（默认路径是：http://localhost:8080/swagger-ui.html）。如下图所示：

![swagger2示意图](/image/swagger/swagger-ui.png)


# 三、注解

1. 使用于API的注解

1.1 使用在类的注解

+	@Api：定义一个API
	+	value	url的路径值
	+	tags	如果设置这个值、value的值会被覆盖
	+	description	对api资源的描述
	+	basePath	基本路径
	+	position	如果配置多个Api 想改变显示的顺序位置
	+	produces	如, “application/json, application/xml”
	+	consumes	如, “application/json, application/xml”
	+	protocols	协议类型，如: http, https, ws, wss.
	+	authorizations	高级特性认证时配置
	+	hidden	配置为true ，将在文档中隐藏

1.2 使用在方法的注解

+	@ApiOperation："用在请求的方法上，说明方法的作用"
	+	value="说明方法的作用"
	+	notes="方法的备注说明"
+	@ApiImplicitParams：用在请求的方法上，包含一组参数说明
	+	@ApiImplicitParam：对单个参数的说明	    
		+	name：参数名
		+	value：参数的说明、描述
		+	required：参数是否必须必填
		+	paramType：参数放在哪个地方
			+	query --> 请求参数的获取：@RequestParam
			+	header --> 请求参数的获取：@RequestHeader	      
			+	path（用于restful接口）--> 请求参数的获取：@PathVariable
			+	body（请求体）-->  @RequestBody User user
			+	form（普通表单提交）	   
		+	dataType：参数类型，默认String，其它值dataType="Integer"	   
		+	defaultValue：参数的默认值
+	@ApiResponses：方法返回对象的说明
	+	@ApiResponse：每个参数的说明
		+	code：数字，例如400
		+	message：信息，例如"请求参数没填好"
		+	response：抛出异常的类

2. 使用在Model的注解

+	@ApiModel：用于JavaBean上面，表示对JavaBean 的功能描述
	+	@ApiModelProperty：用在JavaBean类的属性上面，说明属性的含义

3. 忽略

+	@ApiIgnore()用于类，方法，方法参数 
	+	表示这个方法或者类被忽略 


# 四、使用

控制器的编写：
```java
@RestController
@Api(tags = "用户接口")
public class UserController {
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }


    @PostMapping("/login")
    @ApiOperation("用户登录接口")
    @ApiImplicitParams({
                @ApiImplicitParam(name = "mobile", value = "手机号", defaultValue = "178XXXXXXXX", required = true),
                @ApiImplicitParam(name = "password", value = "密码", defaultValue = "dfdsfsf", required = true)
        }
    )
    public CosimResponse login(String mobile, String password){
        return userService.checkUser(mobile, password);
    }

    @ApiOperation("用户注册接口")
    @ApiImplicitParam(name = "user", value = "用户", required = true, paramType = "com.simtek.cosimcloud.dao.entity.User")
    @PutMapping("/register")
    public CosimResponse register(@RequestBody User user){
        return userService.addUser(user);
    }

    @ApiOperation("用户信息接口")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true)
    @GetMapping("/user/{id}")
    public CosimResponse getById(@PathVariable String id){
        return userService.getUserById(id);
    }

    @ApiOperation("用户注销接口")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true)
    @DeleteMapping("/user")
    public CosimResponse delete(@PathVariable String id){
        return userService.delete(id);
    }
}

```

User实体类：
```java
@ApiModel(value = "用户", description = "表示用户的实体类")
@Entity
@JsonIgnoreProperties(value = {"hibernateLazyInitializer","handler" })
@Table(name = "t_user")
@org.hibernate.annotations.Table(appliesTo = "t_user", comment = "用户信息表")
public class User implements java.io.Serializable {

    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "org.hibernate.id.UUIDGenerator")
    @ApiModelProperty(value = "用户id")
    @Column(name = "id", length = 36, nullable = false, unique = true)
    private String id;

    @Column(name = "r_id", columnDefinition = "tinyint(4) default '1' comment '用户角色：0 管理员，1 用户 '")
    @ApiModelProperty(value = "角色类型")
    private Long rid;

    @Column(name = "register_source", columnDefinition = "tinyint(4) unsigned default '0' comment '注册来源：0手机号 1邮箱 2用户名 3备用'")
    @ApiModelProperty(value = "注册途径")
    private Byte registerSource;


    @Column(name = "password", columnDefinition = "varchar(48) comment '密码'")
    @ApiModelProperty(value = "用户密码")
    private String password;

    @Column(name = "mobile", columnDefinition = " varchar(16) default '' comment '手机号'")
    @ApiModelProperty(value = "用户手机号")
    private String mobile;

    @Column(name = "mobile_bind_time",  columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP comment '手机绑定时间'")
    @ApiModelProperty(value = "手机绑定时间")
    @Generated(GenerationTime.INSERT)
    private java.sql.Timestamp mobileBindTime;

    @Column(name = "email",  columnDefinition = "varchar(128) default '' comment '邮箱'")
    @ApiModelProperty(value = "用户邮箱")
    private String email;

    @Column(name = "email_bind_time",  columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP comment '邮箱绑定时间'")
    @Generated(GenerationTime.INSERT)
    @ApiModelProperty(value = "用户邮箱绑定时间")
    private java.sql.Timestamp emailBindTime;

    @Column(name = "nick_name", columnDefinition = "varchar(48) comment '用户昵称'")
    @ApiModelProperty(value = "用户昵称")
    private String nickName;

    @Column(name = "gender", columnDefinition = "tinyint(1) default '1' comment '用户性别 0-female 1-male'")
    @ApiModelProperty(value = "用户性别")
    private Byte gender;

    @Column(name = "birthday", columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP comment '生日'")
    @Generated(GenerationTime.INSERT)
    @ApiModelProperty(value = "用户生辰")
    private java.sql.Timestamp birthday;

    @Column(name = "face")
    @ApiModelProperty(value = "用户图片")
    private String face;

    @Column(name = "face200")
    @ApiModelProperty(value = "用户图片200*200")
    private String face200;

    @Column(name = "is_delete")
    @ApiModelProperty(value = "是否删除")
    private Boolean isDelete;

    public User() {
    }

    public User(String id, Long rid, Byte registerSource, String password, String mobile, Timestamp mobileBindTime, String email, Timestamp emailBindTime, String nickName, Byte gender, Timestamp birthday, String face, String face200, Boolean isDelete, Timestamp createTime, Timestamp loginTime, Timestamp lastLoginTime) {
        this.id = id;
        this.rid = rid;
        this.registerSource = registerSource;
        this.password = password;
        this.mobile = mobile;
        this.mobileBindTime = mobileBindTime;
        this.email = email;
        this.emailBindTime = emailBindTime;
        this.nickName = nickName;
        this.gender = gender;
        this.birthday = birthday;
        this.face = face;
        this.face200 = face200;
        this.isDelete = isDelete;
        this.createTime = createTime;
        this.loginTime = loginTime;
        this.lastLoginTime = lastLoginTime;
    }

    @Column(name = "create_time",  columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP comment '绑定时间'")
    @Generated(GenerationTime.INSERT)
    @ApiModelProperty(value = "用户创建时间")
    private java.sql.Timestamp createTime;

    @Column(name = "login_time",  columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP comment '登录时间'")
    @Generated(GenerationTime.INSERT)
    @ApiModelProperty(value = "用户登陆时间")
    private java.sql.Timestamp loginTime;

    @Column(name = "last_login_time",  columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP comment '上次登录时间'")
    @Generated(GenerationTime.INSERT)
    @ApiModelProperty(value = "用户最后登陆时间")
    private java.sql.Timestamp lastLoginTime;
	`
	。。。。忽略的getter/setter方法
}
```

生成文档如下所示：

![swagger2-user](/image/swagger/swagger2-user.png)


# 四、导读
> [Spring Boot 2.x（十二）：Swagger2的正确玩法](https://juejin.im/post/5d4a22aaf265da03ca1154c4)
[springboot+swagger接口文档企业实践（上）](https://juejin.im/post/5dcc00c2e51d45105d56306e)