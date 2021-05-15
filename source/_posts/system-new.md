---
title: Win重装系统(二) 安装软件
date: 2019-08-29 21:22:52
tags: 系统
---

我安装的是官方的纯净版系统，c盘直接格式化，也没有对软件进行备份。这篇文章，将简单的介绍如何安装常用的编程工具。


# nodejs
1.nodejs安装后会自动配置环境变量，安装成功后在命令行输入 node -v来查询是否安装成功
2.修改默认存储位置
2.1 查看当前配置：npm config ls
2.2 修改缓存位置  npm config set cache "D:\Program Files\nodejs\node_cache"  
2.3 修改前缀 npm config set prefix "D:\Program Files\nodejs\" 


## npm -v 卡住
删除账户目录下的.npmrc文件。


## 解决npm install卡住
1.使用淘宝镜像
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install -g @angular/cli
```
2.使用代理
```
npm config set registry https://registry.npm.taobao.org
```


# mysql 压缩文件包如何安装
1.配置环境变量;path : \mysql\bin
2.命令行cd到bin文件夹下
3.mysqld --initialize-insecure ，会生成data文件夹
4.为windows安装MYSQL服务,	mysqld --install mysql --defaults-file=D:\software\mysql\my.ini
5.启动mysql服务, net start mysql
6.停止服务，net stop mysql
7.mysqld --remove mysql

*注1：目录需要无中文路径，路径为“\\”而不是"\"*

## 安装mysql提示丢失MSVCP140.DLL
没有安装VC++2015版运行库导致的,在https://www.microsoft.com/zh-cn/download/confirmation.aspx?id=53587安装

## win8.1安装mysql时无法启动服务 
需要配置my.ini,由于mysql 2019新版本解压过后没有my-default.ini和my.ini文件，需要自行创建。

*在根目录添加，my.ini文件:*

```
[mysql]
#设置mysql客户端默认字符集
default-character-set=utf8
[mysqld]
#设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=H:\\mysql-8.0.13-winx64
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```


## Mysql初始化配置

```
>mysql -uroot -p #登陆
初次设置密码： 
	1.update user set authentication_string='' where user='root'; #mysql 8.0添加authentication_string安全字段，需要先设置为空
	2.ALTER user 'root'@'localhost' IDENTIFIED BY 'root';

# 忘记密码的操作
	1.net stop mysql
	2.cd到bin目录下,输入mysqld --skip-grant-tables 回车，即登陆跳过权限验证
```



## Navicat连接MySQL
Navicat连接MySQL，出现2059 - authentication plugin 'caching_sha2_password'的解决方案:

```
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;   #修改加密规则 
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';   #更新一下用户的密码 
      FLUSH PRIVILEGES;   #刷新权限 
	再重置下密码：alter user 'root'@'localhost' identified by 'root';
```
 


# Maven

window 安装maven驱动安装
1.*Maven3.2*.版本需要*JDK1.6*的支持，*Maven3.3*以上需要*JDK1.7*以上的支持
2.到官网下载zip包,解压。配置环境变量：path D:\software\apache-maven-3.6.1\bin
3.测试：mvn -version



# JDK配置

```
Java环境变量
1.Java_Home C:\Program Files\Java\jdk1.8.0_131
2. path&CLASSPATH  %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

cmd输入JAVA报错Could not create the Java virtual machine
环境变量的路径错误，cmd输入path查看
```

# win8.1开启睡眠模式

1、Win+R输入gpedit.msc 命令,然后进入组策略界面，依次选择“计算机配置”、“管理模板”、“Windows组件”；
2、然后再次查找“文件资源管理器”，启动“在电源选项菜单中显示休眠”的行


# Git的安装
下载地址：https://git-scm.com/
配置环境变量：
path
	;C:\Program Files\Git\bin;C:\Program Files\Git\mingw64\libexec\git-core;