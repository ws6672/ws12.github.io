---
title: linux学习指南（四）网络篇
date: 2019-09-25 14:27:08
tags: linux
---

开卷语
>	当linux系统从虚拟机转为实际系统时，就需要我们自己来配置网络、使用网络。

linux操作系统，以太网卡用“eth”表示网卡；序号从零开始。其中，【eth0】表示第一个网卡，以此类推；而其他接口则有默认的其它前缀，例如docker的虚拟接口就是【docker0】.

### 一、linux的网络生态

##### 1. Linux内核提供的功能
+ TCP/IP（传输控制协议/因特网互联协议）
+ NetBIOS/NetBEUI（局域网协议、广播型协议、缺乏路由和网络层寻址功能）
+ IPX/SPX（局域网、庞大、拥有路由功能）
+ 网络底层支持的其他协议——Ethernet(以太网)、Token Ring、ATM、PPP（PPPoE）、FDDI、Frame Relay

##### 2. 网络协议常见层次

+	TCP	传输控制协议；保证数据通信的完整性和可靠性，防止丢包。
	+	IP 因特网互联协议，实现多局域网的互联互通；定义IP地址，实现路由功能。
		+	Ethernet 以太网（底层），实现内网的点对点通信；将数据打包并附上地址广播，拿到数据的节点对比地址，不是就丢弃。

##### 3. 常见网络接口

+	eth 以太网 (Ethernet),是最底层的网络协议
+	tr 令牌环(Token Ring)
+	fddi 光纤
+	ppp 点对点协议
+	lo 本地回环接口
+	docker docker的虚拟接口

##### 4. 数据包的接收


### 二、网络配置
如果使用的是虚拟机，那么还有桥接模式、NAT模式以及仅主机模式几个概念，具体的介绍在[常用术语](#常见术语)中。

配置网络有两种方式：
+	命令修改（临时）
+	修改配置文件（永久）


##### 1. 网络接口命令
```
ifconfig lo # ifconfig 网卡名称
ip address # 查看所有网卡的ip地址
ifup eth0	#	启动网卡接口
ifdown eth0	#	关闭网卡接口

#/sbin/ifup: configuration for docker0 not found.
#网卡查询不到

cd /etc/sysconfig/network-scripts/ #查看有没有对应的网卡文件

```

##### 2. 临时配置


+	ifconfig命令可以临时地设置网络接口的IP参数
+	route命令可以临时地设置内核路由表
+	hostname命令可以临时地修改主机名
+	sysctl命令可以临时地开启内核的包转发

##### 3. 永久配置
要使网络配置永久生效，就需要修改文件了。

```
/etc/sysconfig/network 基础网络文件
/etc/sysconfig/network-scripts/ifcfg-ethX/ 以太网接口
/etc/sysconfig/network-scripts/route-ethX/ 以太网接口的静态路由
/etc/hosts 域名静态解析文件【在访问DNS之前，会先启用该文件的配置】
/etc/resolv.conf 指定域名服务器位置
/etc/host.conf 配置域名服务客户端
```

##### 4. 网络检测命令

```
ifconfig	检测网络接口配置
route	检测路由配置
ping	检测网络连通性
netstat	查看网络状态
lsof	查看指定IP 和/或 端口的进程的当前运行情况
host/dig/nslookup	检测DNS解析
traceroute	检测到目的主机所经过的路由器
tcpdump	显示本机网络流量的状态
```

### 常见术语

>  路由：是指路由器从一个接口上收到数据包，根据数据包的目的地址进行定向并转发到另一个接口的过程。路由通常与桥接来对比，在粗心的人看来，它们似乎完成的是同样的事。

> 桥接：与路由进行类似的工作，它们的主要区别在于桥接发生在OSI参考模型的第二层（数据链路层），而路由发生在第三层（网络层）。这一区别使二者在传递信息的过程中使用不同的信息，从而以不同的方式来完成其任务。 

> 网络模式：桥接模式、NAT模式以及仅主机模式。桥接模式下，虚拟机在路由器中有配置，可以独立与外部通信;NAT模式下，虚拟机无法独立通信，需要通过主机；仅主机模式下，通信断绝

### 参考文档

> [网络管理](https://www.cnblogs.com/yanjieli/archive/2018/08/29/9557198.html)
[Linux网络管理之基础知识详解](https://www.linuxidc.com/Linux/2017-09/146915.htm)
[Linux网络管理](https://juejin.im/post/5b20da20f265da6e281c1838)