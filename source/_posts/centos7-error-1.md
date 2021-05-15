---
title: centos7 异常一
date: 2020-07-22 13:16:53
tags: [linux]
---


### `CentOS7 I/O error metadata corruption`异常
我的系统安装在虚拟机中，强制关闭后出现了这个错误，需要修复习惯。

下载[centos7 镜像](http://mirrors.aliyun.com/centos/7/isos/x86_64/)。
加载镜像到VMware 虚拟机中，`设置——>CD/IDE`


+	重启，按c，输入exit
+	Troubleshooting->Rescue a CentOS system->选择3
+	输入以下命令：```
		xfs_repair -v / dev / mapper / centos-root
	```



### 进入单用户界面
1、重启服务器，在选择内核界面使用上下箭头移动
2、选择内核并按“e”
3、修改脚本,删除掉rhgb quiet，如下图

![centos 脚本](/image/centos7/centos7-1.png)
4、使用“ctrl + x” 来重启服务器就可以了，重启后就会进入到单用户
5、退出单用户命令: `exec /sbin/init`
