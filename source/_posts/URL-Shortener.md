---
title: 短链接
date: 2020-03-26 17:09:36
tags:
---

`url-shortener`是一种服务，用于根据非常长的URL创建短链接。通常，短链接的大小为原始URL的三分之一甚至四分之一，这使它们更易于键入，呈现或发布推文。单击短链接用户将自动重定向到原始URL。 

### 一、功能要求 
+	用户需要能够输入长网址。我们的服务应保存该URL并生成一个短链接。
+	用户应该可以选择输入到期日期。在该日期之后，短链接应该无效。
+	单击短链接应将用户重定向到原始长URL。
+	用户应创建一个帐户以使用服务。服务可以有每个用户的使用限制。
+	允许用户创建自己的短链接。
+	服务应具有指标，例如访问最多的链接。

 
### 二、62进制的转换方法
```
	// 配置数字与进制符
	private  static  Map<Integer,Character> map;
    static {
        map = new HashMap<>();
        String[] strs = {"0-9","A-Z","a-z"};
        int index = 0;
        for (String str: strs) {
            String[] params = str.split("-");
            for (int i= params[0].charAt(0); i<=params[1].charAt(0); i++) {
                map.put(index++, (char)i);
            }
        }
    }
	
   /**
    * @Description 10进制转换为62进制（位数填充）
    * @param number 数字
    * @param bit 位数
    * @return
    */
    public String base10Tobase62(long number,int bit) throws Exception {
        if (bit<0) {
            throw new Exception("位数不可以为负");
        }
        String result = base10Tobase62(number);
        if (result.length() < bit) {
            do {
                result = '0'+result;
            } while (result.length() < bit);
        }
        return result;
    }


    /**
    * @Description 10进制转换为62进制
    * @param number 10进制数
    * @return 62进制数
    */
    public String base10Tobase62(long number) {
        String temp;
        if (number % 62 == number) {
//            递归结束
            temp = getAscll2((char) number)+"";
            return temp;
        } else {
            temp = getAscll2((char)(number%62))+"";
            return  base10Tobase62(number / 62)+temp;
        }
    }
	// 获得 Ascll字符
	public Character getAscll2 (int number) {
        return Test.map.get(number);
    }
```



### 三、数据库配置（JPA+MYSQL）

***POM***
```
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```

***yml配置***

```
server:
  port: 8088
spring:
  application:
    name: urlshorter
  datasource:
    url: jdbc:mysql://localhost:3306/toolweb?characterEncoding=utf8&useSSL=false&serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    database: mysql
    show-sql: true #    打印sql
    hibernate:
      ddl-auto: update
	  
    #      jpa hibernate
		##validate  加载hibernate时，验证创建数据库表结构
		##create   每次加载hibernate，重新创建数据库表结构，这就是导致数据库表数据丢失的原因。
		##create-drop        加载hibernate时创建，退出是删除表结构
		##update                 加载hibernate自动更新数据库结构
		##validate 启动时验证表的结构，不会创建表
		##none  启动时不做任何操作
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect #设置数据库引擎为InnoDB
debug: true
```

***Model配置***
```
@Data //lombok 注解，生成对应方法
@NoArgsConstructor //lombok 注解，生成空构造器
@Entity
@Table(name = "url_shorter")
public class Url implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)  //id自增长
    @Column(name = "id")
    private Long id;
    @Column(name = "url") //设置列属性
    private String url;
    @Column(name = "createdDate")
    private Date createdDate;
    @Column(name = "expiresDate")
    private Date expiresDate;

    public Url(String url) {
        this.url = url;
        createdDate = new Date();
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(createdDate);
        calendar.add (Calendar.HOUR_OF_DAY, 12);
        expiresDate = calendar.getTime();
        calendar = null;

    }
}
```

***Repository配置***
```
public interface UrlRepository extends JpaRepository<Url,Long> {

}

******

```

这里，由于项目过小，只有`controller`、`dao`，没有设置`service`层,避免结构臃肿。

### 四、控制器配置
***Controller***
```
@RestController
@RequestMapping("/url")
public class UrlShorterController {
    @Autowired
    UrlRepository urlRepository;

    /**
    * @Description 长链转短链
    * @param url 长链
    * @return 短链
    */
    @RequestMapping(value = "addUrlShorter")
    public String addUrlShorter(String url) {
        Url url1 = new Url(url);
        Long result;
        String code = null;
        try {
            url1 = urlRepository.save(url1);
            if (url1 != null) {
                result = url1.getId();
                code = BaseUtil.base10Tobase62(result,7);
            }
        }  catch (Exception e) {
            e.printStackTrace();
        }
         return "http://127.0.0.1:8088/url/root/"+code;
    }
    /**
    * @Description 通过短链获取长链
    * @param code 长链接ID的62进制
    * @return 长链接 
    */
    @RequestMapping(value = "root/{code}")
    public String getUrlShorter(@PathVariable String code) {
        Long id = BaseUtil.base62Tobase10(code);
        Url url = urlRepository.getOne(id);
        return url.getUrl();
    }

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String hello() {
        return "hello";
    }
}
```



安装`Sringboot YAML` 提示插件
+	yaml/ansible support
+	spring assistant 

 

### 五、遇到的几个问题
***The server time zone value 'XXX' is unrecognized or represents more than one time zone***
```
需要配置 serverTimezone=UTC
spring:
  application:
    name: urlshorter
  datasource:
    url: jdbc:mysql://localhost:3306/toolweb?characterEncoding=utf8&useSSL=false&serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```


***Error creating bean with name 'entityManagerFactory' defined in class path resource***

在类中的`@Id`使用了错误的包引入。我的导入是`org.springframework.data.annotation.Id`,改为`javax.persistence.Id`,这应该有助于解决您遇到的异常。


### 六、测试
获取短链：
```
http://127.0.0.1:8088/url/addUrlShorter?url=https://blog.csdn.net/yunfeng482/article/details/72721101
http://127.0.0.1:8088/url/root/0000007
```

通过短链访问：
```
http://127.0.0.1:8088/url/root/0000007

https://blog.csdn.net/yunfeng482/article/details/72721101
```


### 七、参考文章
> [URL Shortener: A Detailed Explanation](https://dzone.com/articles/url-shortener-detailed-explanation)
