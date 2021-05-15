---
title: tomcat实战（一）linux使用多个tomcat
date: 2019-11-08 14:00:27
tags: [javaweb]
---

### 1. 修改server.xml
Tomcat服务器通过Connector连接器组件与客户程序建立连接，Connector组件负责接收客户的请求，以及把Tomcat服务器的响应结果发送给客户。当多个tomcat一起工作的时候，我们需要修改以下几个端口：
+	8005是 tomcat接受关闭指令的端口，不修改的话会导致一个shutdown脚本同时关闭多个tomcat
+	8080是http 1.1 connector，也就是接收处理http请求的端口
+	8009是ajp connector，它一般用来设置tomcat集群。

```
<!-- 修改停止端口8005 -->
<Server port="9005" shutdown="SHUTDOWN">
  <GlobalNamingResources>
   
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
 
  <Service name="Catalina">
  <!-- 修改运行端口8080 -->
    <Connector port="8082" protocol="HTTP/1.1" 
               connectionTimeout="20000" 
               redirectPort="8443" />
			   
    <Connector port="9009" protocol="AJP/1.3" redirectPort="8443" />
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true"
            xmlValidation="false" xmlNamespaceAware="false">
      </Host>
    </Engine>
  </Service>
</Server>
```

### 2. 配置不同的环境变量
在linux中使用tomcat，需要配置环境变量。在增加tomcat后，每个tomcat中，除了jdk的环境变量可以共用，其它的需要使用不同的环境变量。

```
vi /etc/profile 
	
	# 在文件末尾添加
	export CATALINA_2_BASE=/root/ops/tomcat2
	export CATALINA_2_HOME=$CATALINA_2_BASE
	export TOMCAT_2_HOME=$CATALINA_2_BASE 
# 更新环境变量
source /etc/profile 
```

### 3. 修改tomcat的脚本
在tomcat下的bin目录中为脚本引入环境变量。


*startup.sh 、shutdown.sh文件*
```
# Stop/Start script for the CATALINA Server
	export JAVA_HOME=/root/jdk1.8.0_161
	export CLASSPATH=$JAVA_HOME/bin
	export CATALINA_HOME=$CATALINA_2_HOME
	export CATALINA_BASE=$CATALINA_2_BASE
```

*catalina.sh*
```
# Start/Stop Script for the CATALINA Server
	export CATALINA_BASE=$CATALINA_2_BASE
	export CATALINA_HOME=$CATALINA_2_HOME
	export TOMCAT_HOME=TOMCAT_2_HOME
```
### 4. 启动

```
cd bin/
./startup.sh
```