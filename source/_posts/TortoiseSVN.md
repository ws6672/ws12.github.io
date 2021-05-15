---
title: TortoiseSVN 的使用
date: 2019-11-30 16:16:02
tags: [tool]
---

TortoiseSVN 协同开发，谁用谁知道。

### 一、简介

TortoiseSVN 是 Subversion 版本控制系统的一个免费开源客户端，可以超越时间的管理文件和目录。文件保存在中央版本库，除了能记住文件和目录的每次修改以外，版本库非常像普通的文件服务器。你可以将文件恢复到过去的版本，并且可以通过检查历史知道数据做了哪些修改，谁做的修改。这就是为什么许多人将 Subversion 和版本控制系统看作一种“时间机器”。

[下载地址](https://tortoisesvn.net/downloads.html), 下载建议使用梯子。

 
### 二、TortoiseSVN 的使用

+	checkout（检出）：相当于从服务器获取到本地。导出获得文件后，导出的文件仍处于SVN版本控制中，与版本库保持关联，即带有【.svn隐藏文件夹】；而【export】这只是单纯导出文件而已
+	export（导出）：也是将文件获取到本地。但获取的文件是不受版本控制的

check out跟check in对应，export跟import对应。

 
### 三、踩坑

不想踩坑的程序员不是一个好码农。TortoiseSVN中的坑并不多，至少我遇到的不多。

+	【win】用SVN时，【checkout】至非空目录中
	+	```
	进入文件删除隐藏文件夹【.svn】即可
	```
+	【linux服务端】svn: error while loading shared libraries: libaprutil-1.so.0: cannot open shared object file: No such file or directory
	+	```
	#	1.系统还缺少了apache的库apr-util的支持，需要安装一下
	yum install -y apr-util
	
	#	2.库中缺少依赖，将依赖添加到共享库
	find / -name libaprutil-1.so.0
		/ewomail/apache/lib/libaprutil-1.so.0
		
	#	3.查看共享库
	# more /etc/ld.so.conf
		include ld.so.conf.d/*.conf
		#	添加查询到的路径
		/ewomail/apache/lib/
		
	#	4.更新
	ldconfig -v

	```
+	【win】连不上linux的svn，使用telnet测试连接
	+	```
		#	控制面板--->程序(程序和功能)-->打开或关闭windows功能，打开telnet服务端、telnet客户端
		telnet ip port  -- ip和端口之间不需要冒号

	```
+	【linux服务端】svn Invalid authz configuration
	+	```
	#	authz文件配置错误，可以使用【svnauthz-validate】检查【authz】文件
	
	```
 
### 四、官方几个更新

##### CryptSync

CryptSync是一个小型实用程序，它在加密一个文件夹中的内容时会同步两个文件夹。这意味着其中一个文件夹的所有文件（使用的文件）均未加密，而另一个文件夹的所有文件均已加密。同步以两种方式起作用：一个文件夹中的更改被同步到另一文件夹。如果在未加密的文件夹中添加或修改了文件，则会对其进行加密。如果在加密文件夹中添加或修改了文件，则该文件将解密到另一个文件夹。

如果文件夹放在公用的文件服务器上，那么这个小程序就很实用了，可以在一定程度上确保代码的安全性。

[链接](https://tools.stefankueng.com/CryptSync.html)

##### 回滚与回收站

在【TortoiseSVN】中，会保存多个版本的文件，所以你在多次修改后还能够找回以前的数据，但如果在一次修改后回滚了，文件依旧可以找回，因为它默认会将这些文件放置到回收站中去。

### 五、TortoiseSVN 云存储

电脑的空间是有限的，所以云存储成为了新的选择，而TortoiseSVN搭配CryptSync一起食用是极佳的解决方案。

##### 1.安装
```
yum -y install subversion

#	查看安装位置
rpm -ql subversion
```

##### 2.创建版本库

```
#	会在【/var/svn/md】下创建版本库相关文件【conf  db  format  hooks  locks  README.txt】
#	在【conf】中存在【authz  passwd  svnserve.conf】，用于授权、密码、SVN服务配置文件
mkdir -p /var/svn/md
svnadmin create /var/svn/md
cd /var/svn/md
vim conf/passwd

	[users]
	ali=ali

vim conf/authz
	
	[groups] #配置用户组
	admin = ali
	# viewer = viewer
	[md:/] #为文档库【md】配置用户组权限
	@admin=rw
	@viewer=r

vim conf/svnserve.conf

	anon-access = read
	auth-access = write
	password-db = passwd
	authz-db = authz

```

##### 3.运行
```
#	启动
#	svnserve -d -r 目录 【--listen-port 端口号，不加默认是3690】
svnserve  -d -r /var/svn/

#	停止
killall svnserve

#	访问需要开放端口【3690】
systemctl status firewalld
systemctl start firewalld
firewall-cmd  --zone=public --add-port=3690/tcp --permanent
firewall-cmd --reload

```


### 六、使用错误

##### 【svn clean up】cleanup failed to process the following paths
使用数据库可视化软件打开【\.svn\wc.db】sqlite数据库;
删除wc_lock和work_queue两张表下的所有记录;

***


参考文档
> [svn服务的停止与启动](https://blog.csdn.net/nanshaowei/article/details/52506083)
