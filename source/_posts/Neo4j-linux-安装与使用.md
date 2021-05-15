---
title: Neo4j在 linux系统中的安装与使用
date: 2019-09-10 22:58:37
tags: [neo4j]
---

# 前言
在学习Neo4j后，由于本人有一台云服务器，打算在云服务器上配置Neo4j. 由于Neo4j是基于JAVA SE 的SDK的数据库，所以需要运行在支持JVM的机器上，即按照了JDK的机器。

环境准备：
+ 本地安装PSCP
+ 远程服务器安装Jdk8，可以通过FTP或者pscp将安装包传输到Linux系统上。

### 一、PSCP


##### 什么是PSCP
【Pscp】 是用于在服务器和另一台计算机（可以是另一台服务器或家用计算机）之间交换文件（加密）的程序。在Linux系统中存在【SCP】命令用于在两个Linux系统间传递文件，而【PSCP】是Windows系统与Linux系统交互的程序。

##### 使用
它是【Putty】的一个程序，【Putty】是用来远程访问Linux系统的程序。
+ [Putty 下载地址](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
+ 语法如下

```
在命令行CD到根目录；

传输到本地
pscp [options] [user@]host:source target

传输到远程服务器
pscp [options] target  [user@]host:source
Pscp war包路径   用户名@服务器域名/IP :服务器路径

参数解释：
	user:远程主机的用户名
	host:远程主机的ip
	source:远程主机上的文件， 只能是单个。
	target:本地的存放路径可指定文件名
	option
	    -ls 列文件
	    -P【大写】 指定ssh端口
	    -l 修改用户在命令中的位置【pscp -l user target host:source】

```

##### 连接提示
第一次连接，有提示【...Store key in cache? (y/n)】，即服务器的主机密钥没有缓存在注册表中，会询问这是不是本人的电脑.如果输入【Y】,会保存密匙；输入【N】就是一次性连接；返回则表示放弃。


### 二、centos7安装JDK8

+ 解压【jdk-8u221-linux-x64.tar.gz  】
    + tar -zxvf jdk-8u221-linux-x64.tar.gz
    + rm jdk-8u221-linux-x64.tar.gz 
    + mkdir /usr/local/java/
    + cp -rf jdk1.8.0_221/ /usr/local/java/
    + cd /usr/local/java/jdk1.8.0_221/ 
+ 配置环境变量
    + vim /etc/profile
        ```
export JAVA_HOME=/usr/local/java/jdk1.8.0_221
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
        ```
    + source /etc/profile #重置环境变量
+ java -version #查看安装是否成功

### 三、远程服务器安装Neo4j

下载有两个方法，一个是下载到Window系统，然后通过PSCP等方式将文件传输到Linux；
或者是通过命令在服务器直接下载文件，但是需要提前知道文件的下载链接。

##### 1. 安装

```

你可以通过【https://neo4j.com/download-center/#community】获取【Neo4j】最新的版本号，替换掉下面链接的版本号。

curl -O http://dist.neo4j.org/neo4j-community-3.5.9-unix.tar.gz

# 解压到指定目录
mkdir neo4j
tar  -zxvf  neo4j-community-3.5.9-unix.tar.gz -C neo4j

cd neo4j/neo4j-community-3.5.9/
```


##### 2. 配置环境变量
```
vim /etc/profile 所有用户

# vim ~/.bashrc 当前用户

	

    export NEO4J_HOME=/usr/local/neo4j/neo4j-community-3.5.9
	
    export PATH=$PATH:$NEO4J_HOME/bin



#刷新环境变量

source /etc/profile

```

##### 3. 修改配置

** vim conf/neo4j.conf **


neo4j.conf 

```
  	# 取消从指定位置获取文件，
	# dbms.directories.import=import
	# 由于内存不足，不修改内存分配
	# 设置堆内存【小于物理内存】
	# dbms.memory.heap.initial_size=5g
	# dbms.memory.heap.max_size=10g
	# 设置缓存
	# dbms.memory.pagecache.size=10g

	# 允许远程访问
	dbms.connectors.default_listen_address=0.0.0.0
	# 默认 bolt端口是7687，http端口是7474，https关口是7473，需要哪个就把【#dbms.connector.XXX.listen_address】的注解打开
	dbms.connector.bolt.enabled=true
	dbms.connector.bolt.listen_address=172.17.185.184:7687


	dbms.connector.http.enabled=true
	dbms.connector.http.listen_address=172.17.185.184:7474

	dbms.connector.https.enabled=true
	dbms.connector.https.listen_address=172.17.185.184:7473

	# 允许远程加载csv
	dbms.security.allow_csv_import_from_file_urls=true
	 

	# 允许可读可写
	dbms.read_only=false

```
##### 4. 使用

4.1 启动

```
cd bin/
#Usage: neo4j { console | start | stop | restart | status | version }
./neo4j start
```

4.2 开放端口

```
#Linux开放端口
	firewall-cmd --zone=public --add-port=7687/tcp --permanent
	firewall-cmd --zone=public --add-port=7473/tcp --permanent	
	firewall-cmd --zone=public --add-port=7474/tcp --permanent

#重启防火墙
	systemctl restart firewalld.service

#云服务器控制台配置安全组入方向,开放端口 7474，7473以及7687
```

4.3 访问

默认通过【http://localhost:7474/】访问；
配置后通过【dbms.connector.http.listen_address】访问（局域网）；
外网访问，将内网IP换成外网IP即可。

