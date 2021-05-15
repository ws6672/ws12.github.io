---
title: docker攻略（二）从镜像到项目
date: 2019-09-30 15:11:43
tags: [docker]
---

在第一篇文章中，讲述的是docker的基础使用。在这一篇，通过在docker部署应用来学习docker的实践。在这个过程中，会基于tomcat的镜像配合应用的【war】包来构建应用镜像。


### 一、环境需求
执行一个极简的web应用需要的环境如下：
+	tomcat
+	jdk（tomcat自带jdk，如果以tomcat为基础构建映像，就不需要添加jdk了）
+	mysql（数据库）

```
docker pull tomcat #tomcat中自带jdk，就不需要【pull jdk】了
docker pull mysql #（数据库）

# 以上下载的软件都没有带版本号标签，默认是最新版
```

### 二、springboot项目通过tomcat运行
+	通过idea的【springboot】插件快速创建项目
+	修改pom.xml文件.  打包方式改为war包  并取消自带的tomcat。【pom.xml】修改,打包方式设置为【war】，取消使用【springboot】提供的【tomcat】
```
	<packaging>war</packaging>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
	</dependencies>

```
+	重写启动类
```
@SpringBootApplication(scanBasePackages = "com.toolweb.backend",exclude = {SecurityAutoConfiguration.class})
@EnableCaching
public class ToolwebApplication extends SpringBootServletInitializer {
	public static void main(String[] args) {
		SpringApplication.run(ToolwebApplication.class, args);
	}
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
		return builder.sources(ToolwebApplication.class);
	}
}
```
+	使用IDEA 的maven插件进行打包
+	把包放置到【\apache-tomcat-XXX\webapps】下
+	启动【startup.bat】就可以通过浏览器访问了



##### 1. tomcat无法启动
这时需要逐步排查。
+	在【catalina.bat】末尾添加【pause】避免启动就闪退。
+	【The JRE_HOME environment variable is not defined correctly.This environment variable is needed to run this program】：jre环境变量配置错误，重设即可



##### 2. 解决tomcat启动时的中文乱码问题

【conf/logging.properties】下设置
```
java.util.logging.ConsoleHandler.encoding = GBK
# java.util.logging.ConsoleHandler.encoding = UTF-8
```

##### 3. 没有访问指定页面
tomcat启动应用时，url要加webapp对应项目的文件名，而项目名是ROOT，则url不用加

> 在实际使用中，如果仅仅只是用于测试，可以直接打成jar包，使用内置tomcat即可。



### 三、使用docker运行
+	使用pscp将war包从window传输到指定服务器
```
	# pscp toolweb.war user@ip:/home/user
	pscp E:\toolweb\apache-tomcat-9.0.26-windows-x64\apache-tomcat-9.0.26\webapps\toolweb.war ali@XXXIP:/home/ali/
```
+	编写Dockerfile文件
```
	from tomcat
	expose 9999
	maintainer ws
	copy  ./toolweb.war /usr/local/tomcat/webapps
```
+	构建镜像
```
docker image build 		  -t 	toolweb:2.0 	.
	#【		构建命令	】   【-t表示使用标签,格式是 库：版本】		【点表示当前目录,表示dockerfile的位置】

```
	
+	启动与停止
```
	docker run -p 9999:9999 -d toolweb:2.0
	docker stop dd5a
```


##### 1. 删除容器与镜像

```
	# 查看
	docker images
	# 删除镜像
	# docker rmi <image id>
	# 删除所有容器
	docker rm $(docker ps -a -q)
	# 删除所有镜像
	docker rmi $(docker images -q)
```

通过ID删除有时候会报错【 image is referenced in multiple repositories】，表示id与镜像不是一一对应，可以通过仓库与标签删除

 
##### 2. 在压缩文件中修改配置文件

```
# 不解压而修改文件需要unzip的支持
yum install -y unzip 
yum install -y zip
vim toolweb.jar 
# 通过【/applic】查找到【yml】文件名，回车打开文件进行修改保存
# 可以通过【ip address】查找docker对应的虚拟ip


```

##### 3. 容器内无法用【vim】编辑文件

```
apt-get update
apt-get install -y vim
```

##### 4. 连接数据库

一个实际使用的项目，大都需要用到数据库，我们可以通过docker启动一个mysql的镜像，但还需要修改项目中的配置。

```
# 启动mysql

docker run -di --name tbs -p 3309:3306  -e MYSQL_ROOT_PASSWORD=root mysql
	# -p 3306:3306 端口映射
	# --name tbs 设置名称
	# -e MYSQL_ROOT_PASSWORD=root 设置默认密码
	# -v


docker exec -it tbs /bin/bash
	root@d50ac92aa6dd:/# mysql -uroot -p
		mysql> create database toolweb;

```

##### 5. 连接mysql的 url出现错误

【com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link 】
	配置文件中springboot配置文件错误，检查即可。
```
# 查看容器ip
docker inspect -f='{{.Name}} {{.NetworkSettings.IPAddress}} {{.HostConfig.PortBindings}}' $(docker ps -aq)

# 编辑文件【application.yml】
  profiles: linux
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://查到的容器IP:3306/toolweb?serverTimezone=UTC&useSSL=false
    username: root
    password: XXX
```

##### 6. Docker中tomcat无法访问【http://ip:9999/toolweb/】

+	排查tomcat是否没有正常启动
	```
	docker logs abea #abea 是tomcat的id
	```
	
+	排查是否端口号错误
```
	# 我的项目是springboot的，里面有设置端口号，但其实通过tomcat启动那么就使用的是tomcat的默认端口。
	
	# 查看IP
	docker inspect -f='{{.Name}} {{.NetworkSettings.IPAddress}} {{.HostConfig.PortBindings}}' $(docker ps -aq)
		#/serene_blackwell 172.18.0.3 map[9999/tcp:[{ 9999}]] tomcat的ip
		#/tbs 172.18.0.2 map[3306/tcp:[{ 3309}]]
		
	# 检测通过容器url是否可以访问
	curl http://172.18.0.3:8080/toolweb # 默认端口是8080,如果命令没有错误，那么tomcat的端口应该是8080
	
```

+	排查是否防火墙无开放对应端口
```
	 firewall-cmd --permanent --zone=public --list-ports 
	 
	 # 如果firewall没有开放对应的端口
	 
	 # 开放端口
	 firewall-cmd --permanent --zone=public --add-ports=21/tcp --permanent
	 firewall-cmd --reload
```

+	排查云主机安全机制中是否开放端口


### 参考文献
> [Docker部署MySql应用](https://www.cnblogs.com/yui66/p/9728732.html)
[docker中mysql数据库的数据导入和导出](https://www.cnblogs.com/gyadmin/p/7814737.html)
[使用Docker安装mysql，挂载外部配置和数据](https://www.cnblogs.com/linjiqin/p/11465804.html)

