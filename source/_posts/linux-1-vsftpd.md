---
title: 在centos7中安装vsftpd
date: 2019-09-05 15:27:04
tags: [linux]
---

# 一、安装与使用


### 1.安装
```
yum install vsftpd
```

### 2.启动和查看vftpd服务

```
systemctl start vsftpd #启动
systemctl is-enabled vsftpd.service #设置自启动
systemctl status vsftpd #查看
```

### 3.修改自启动级别

##### chkconfig命令
```
–add 　增加所指定的系统服务，让chkconfig指令得以管理它，并同时在系统启动的叙述文件内增加相关数据。
–del 　删除所指定的系统服务，不再由chkconfig指令管理，并同时在系统启动的叙述文件内删除相关数据。
–level<等级代号> 　指定读系统服务要在哪一个执行等级中开启或关毕。
等级 0 表示：表示关机
等级 1 表示：单用户模式
等级 2 表示：无网络连接的多用户命令行模式
等级 3 表示：有网络连接的多用户命令行模式
等级 4 表示：不可用
等级 5 表示：带图形界面的多用户模式
等级 6 表示：重新启动

chkconfig --list|grep vsftpd [name]：显示所有运行级系统服务的运行状态信息（on或off）。如果指定了name，那么只显示指定的服务在不同运行级的状态。
```
 

### 4.打开防火墙端口

用类似如下的命令打开20/21端口

```
firewall-cmd --zone=public --list-ports # 查看所有端口
firewall-cmd --zone=public --add-port=21/tcp --permanent   # 开放 21 端口
# firewall-cmd --zone=public --remove-port=21/tcp --permanent  关闭 21 端口
firewall-cmd --reload   # 配置立即生效
```

 


### 5.配置文件服务器的客户端

前文配置的【vsftpd】是ftp的服务端，如果要在服务器测试是否能够连接，还需要再安装【ftp】。

```
yum -y install ftp
```

### 6.测试
由于我的linux系统是安在云服务器中的，所以有局域网和公网两个IP需要测试。如果如以上的步骤进行配置，那么应该可以在服务器上通过局域网访问ftp服务。

```
ftp 172.17.185.184
```


阿里云服务器 中ftp无法远程连接
1.配置网络和安全组->安全组配置->添加配置，入方向打开 20/21 1024/65535 等端口范围
2.firewalld 开放端口，需要打开，不打开该服务无法远程连接
2.1 firewall-cmd –zone=public –add-port=20-21/tcp –permanent
2.2 systemctl restart firewalld.service 

FileZilla:不安全服务器,不支持 FTP over TLS、

# 四、添加SSL 加密传输模块


### 1.检测是否支持ssl模块
```
$ ldd $(which vsftpd) | grep ssl
		libssl.so.10 => /lib64/libssl.so.10 (0x00007fbdfe524000)
```
	
### 2.添加vsftpd专用凭据

需要管理员权限；然后，按提示输入证书的签名信息即可

```
$ cd /etc/pki/tls/certs/

# make vsftpd.pem
	Country Name (2 letter code) [XX]:CN  #国家
	State or Province Name (full name) []:gd #省份
	Locality Name (eg, city) [Default City]:gz #城市
	Organization Name (eg, company) [Default Company Ltd]:  #机构名称
	Organizational Unit Name (eg, section) []: #机构部门名称
	Common Name (eg, your name or your server's hostname) []:ali #用户名
	Email Address []: #邮箱

# cp vsftpd.pem /etc/vsftpd/
```




### 3.备份、修改配置文件，

*备份*
```
# cp vsftpd.conf vsftpd.conf.bat 
```

*修改*
```
vsftpd.conf的配置

	#SSL
	ssl_enable=YES
	allow_anon_ssl=NO
	force_local_data_ssl=YES
	force_local_logins_ssl=YES
	ssl_tlsv1=YES
	ssl_sslv2=NO
	ssl_sslv3=NO
	rsa_cert_file=/etc/vsftpd/vsftpd.pem        #凭证存放位置
	pasv_min_port=59000        #凭证连接最小端口
	pasv_max_port=59010        #凭证连接最大端口，大于最小端口
```
*防火墙开放端口*	

```
firewall-cmd --permanent --zone=public --add-port=59000-59010/tcp
```


# 三、vsftpd 虚拟用户配置	

 

### 1.安装 libdb-utils,用于生成虚拟用户认证文件

```
yum -y install libdb-utils 
```

### 2.创建宿主用户（guest_username），虚拟用户映射宿主用户的目录

```
 useradd -d /var/vsftp -s /sbin/nologin vsftp
```

### 3.修改主配置文件 vsftpd.config


	cd /etc/vsftpd/
	vim vsftpd.conf
	touch chroot_list #chroot()会检查chroot_list文件

	```
	vsftpd.config
		anonymous_enable=NO #关闭匿名访问
		local_enable=YES #启用本地系统用户，包括虚拟用户
		write_enable=YES#允许执行FTP命令，如果禁用，将不能进行上传、下载、删除、重命名等操作
		#限制用户不能访问非主目录
		chroot_local_user=NO 
		chroot_list_enable=YES
		chroot_list_file=/etc/vsftpd/chroot_list #例外用户清单
		
		#虚拟用户配置文件目录
		user_config_dir=/etc/vsftpd/vuser_dir
		#宿主用户
		guest_enable=YES
		guest_username=vsftpd
	```

### 4.创建虚拟用户

```
vim vsftpd.conf
	ftpv #用户名
	111111 #密码
```

### 4.1虚拟认证文件
```
db_load -T -t hash -f /etc/vsftpd/vuser /etc/vsftpd/vuser.db
chmod 600 /etc/vsftpd/vuser.db
```

### 4.2虚拟用户认证配置
```
cp /etc/pam.d/vsftpd{,.bak}
vim /etc/pam.d/vsftpd
	auth required /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser
	account required /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser
```

### 4.3虚拟用户配置文件

	mkdir /etc/vsftpd/vuser_conf/
	vim /etc/vsftpd/vuser_conf/ftpv　　#文件名与对应FTP虚拟用户一致
	```
			local_root=/var/vsftp/ftpserver　　#虚拟用户主目录，用户和组必须指定为宿主用户vsftp
			#vsftpd主配置文件中已规定虚拟用户权限与匿名用户一致，因此以下针对匿名用户的权限配置即为虚拟用户的权限 
			anon_umask=077 
			anon_world_readable_only=NO
			anon_upload_enable=YES
			anon_mkdir_write_enable=YES
			anon_other_write_enable=YES
	```

### 4.4虚拟用户主目录和权限
```
mkdir -p /var/vsftp/ftpv/newdir
chown -R vsftp.vsftp /var/vsftp
chmod -R 500 /var/vsftp
chmod -R 700 /var/vsftp/newdir
```
