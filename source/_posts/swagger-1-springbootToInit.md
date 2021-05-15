---
title: swagger（一）springboot 使用swagger2
date: 2019-12-09 11:37:47
tags: [javaweb]
---

# 一、基础

API功能的发展是不可避免的，但维护API文档的头痛并非必须如此。Swagger工具可以帮助您完成生成和维护API文档的工作，确保您的文档在API发展过程中保持最新状态。
在Swagger团队加入SmartBear并且该规范于2015年被捐赠给OpenAPI计划之后，Swagger规范被重命名为OpenAPI规范（OAS），已成为定义RESTful API的因素标准。
OAS合同描述了API的作用，它的请求参数和响应对象，所有这些都没有任何代码实现的指示。使用OAS定义的Web服务可以相互通信，而不管它们内置的语言，因为OAS是语言无关的和机器可读的。
优势：
+	生成API文档
+	在线对接口进行测试

使用步骤
+	编写接口文件
+	使用swagger-ui展示

### 1. 组成
+	swagger
	+	Swagger Editor：基于浏览器的编辑器，我们可以使用它编写我们 OpenAPI 规范。
	+	Swagger UI：它会将我们编写的 OpenAPI 规范呈现为交互式的 API 文档，后文我将使用浏览器来查看并且操作我们的 Rest API。
	+	Swagger Codegen：它可以通过为 OpenAPI（以前称为 Swagger）规范定义的任何 API 生成服务器存根和客户端 SDK 来简化构建过程。

### 2. 导包


```
<!--        swagger-->
<!--        文档-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
<!--        非原生ui，优化UI-->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>swagger-bootstrap-ui</artifactId>
            <version>1.9.0</version>
        </dependency>
```

### 3. 常用注解

 
+	@Api()用于类； 
	+	表示标识这个类是swagger的资源 
+	@ApiOperation()用于方法； 
	+	表示一个http请求的操作 
+	@ApiParam()用于方法，参数，字段说明； 
	+	表示对参数的添加元数据（说明或是否必填等） 
+	@ApiModel()用于类 
	+	表示对类进行说明，用于参数用实体类接收 
+	@ApiModelProperty()用于方法，字段 
	+	表示对model属性的说明或者数据操作更改 
+	@ApiIgnore()用于类，方法，方法参数 
	+	表示这个方法或者类被忽略 
+	@ApiImplicitParam() 用于方法 
	+	表示单独的请求参数 
+	@ApiImplicitParams() 用于方法	包含多个 @ApiImplicitParam


### 4. 实例

springboot中定义配置类
```
@Configuration //加载配置
@EnableSwagger2 //启动swqgger
@EnableSwaggerBootstrapUI
public class SwaggerConfig {
    @Bean
    public Docket   createApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
//                设置controller目录
                .apis(RequestHandlerSelectors.basePackage("com.example.rest.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return  new ApiInfoBuilder()
//                文档标题
                .title("REST项目文档")
                .description("REST项目文档")
                .termsOfServiceUrl("")
                .version("0.1")
                .build();
    }

}
```

定义返回类型
```
@ApiModel(value = "返回值", description = "返回json格式的响应数据")
//识别对象需要设置泛型
public class ReturnMsg<T> {
    @ApiModelProperty(value = "是否成功")
    private boolean success = true;
    @ApiModelProperty(value = "返回对象")
    private Object data;
    @ApiModelProperty(value = "错误编号")
    private Integer errCode;
    @ApiModelProperty(value = "错误信息")
    private String message;
}
```

一个控制器的实例
```
@RestController
@RequestMapping("/test")
@Api(value = "测试类")
public class TestController {

    @ApiOperation(value = "hello 方法", notes = "打招呼")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "name",value = "姓名",dataType = "String",paramType = "query")
    })
    @ApiResponses({
            @ApiResponse(code=200,message="获取成功",response=ReturnMsg.class)
    })
	// method = {RequestMethod.GET}需要定义，不然会展示该方法所有的请求类型
    @RequestMapping(value = "/hello", method = {RequestMethod.GET})
    public ReturnMsg<String> hello( String name) {
        if (name == null)
            name = "everyone";
        ReturnMsg msg = new ReturnMsg();
        msg.setData("hello "+name);
        return msg;
    }
}
```

>  如果不指定【@RequestMapping】的httpMethod属性，springfox会把这个方法的所有动作GET/POST/PUT/PATCH/DELETE的方法都列出来。

访问路径：
`http://localhost:9084/doc.html`

实际界面：
![swagger-ui.png](/image/swagger/swagger-ui.png)

# 二、实际应用

### 1. 根据环境选择显示文档

在实际使用中，一般会有多个环境，例如开发环境、正式环境；在正式环境中就不应当展示文档界面。
我们可以通过以下的方式来切换环境：
+	在正式环境注释掉 swagger配置类的注解 [false]
+	从配置类获取配置 [true]

两种方法中，第二种虽然有一定工作量，但是效果更好，可以快速切换，无需二次编译。




相关jar包
```
<!--       解决@ConfigurationProperties引起的 【spring boot configuration annotation processor not found in classpath】错误-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
<!--        @Data 该注解标在实体类上，不需要再为属性生成get、set方法-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.10</version>
        </dependency>
		
```


相关类
```
// 配置文件 【swagger.properties】

swagger.basePackage =com.example.rest.controller
swagger.title = REST项目文档
swagger.description = REST项目文档
swagger.version = 0.1
swagger.enable = true
swagger.contactName = ws
swagger.contactEmail = ws@gmail.com
swagger.contactUrl = http://aixf.club/
swagger.license =
swagger.licenseUrl =

// 信息类【SwaggerInfo】
@Component
@ConfigurationProperties(prefix = "swagger")
@PropertySource("classpath:/swagger.properties")
@Data
public class SwaggerInfo {
    private String basePackage;
    private String antPath;
    private String title = "HTTP API";
    private String description = "Swagger 自动生成接口文档";
    private String version ;
    private Boolean enable;
    private String contactName;
    private String contactEmail;
    private String contactUrl;
    private String license;
    private String licenseUrl;
}


```

### 参考文献
> [Spring Boot 2.x（十二）：Swagger2的正确玩法](https://juejin.im/post/5d4a22aaf265da03ca1154c4)
[springboot+swagger接口文档企业实践（上）](https://juejin.im/post/5dcc00c2e51d45105d56306e)