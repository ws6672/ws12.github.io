---
title: centos 7（）在 vm 虚拟机中的安装
date: 2020-01-21 11:55:38
tags: [linux]
---



### 资源
[CentOS-7-x86_64-DVD-1908 镜像](http://mirrors.aliyun.com/centos/7.7.1908/isos/x86_64/)

### 虚拟机的配置

使用的虚拟机软件是【VMware Workstation Pro】，需要进行如下配置
+	设置 ISO镜像
![设置 ISO镜像](/image/centos/vm-centos-config.png)

+	启动虚拟机
![设置 ISO镜像](/image/centos/vm-centos7-start.png)


### 安装过程
1. 选择语言
2. 软件选择 
+	FTP 服务器
+	Java 平台
+	KDE 图形界面
+	性能工具
+	Linux远程管理（OpenLMI 、SNMP）
+	开发工具
3. 安装位置，可以进行分区，但由于是安装在虚拟机中，所以不需要进行设置
4. 当所有图标没有提示的时候，就点击开始安装
5. 安装的时候，会显示设置Root密码、创建用户
+	Root密码是Root管理员账号的秘密；
+	创建用户建议建立一个普通用户，便于权限管理。
6. 安装结束，重启即可


### FTP配置

### Vmware提示以独占方式锁定此配置文件失败