---
title: docker攻略（一）原理探析
date: 2019-09-25 16:07:39
tags: [linux, docker]
---

开卷语
> Docker是运维工程师的神兵利器，但它值得每一个程序猿学习。


Docker已经成为互联网运维必备工具之一，作为一个在外包挣扎的渣渣程序员平时没有机会使用，但还是需要学习一下。
***
# 一、什么是Docker

在一开始学习docker前，我一直认为它是一个可以快速部署的低配虚拟机，但按照[百科](https://baike.baidu.com/item/Docker/13344470?fr=aladdin) 的说法却并非如此。 Docker是基于 Linux 内置的 Namespace 和 CGroup 等系统内置隔离机制而抽象出来的一种轻虚拟化技术。



### 1. 原理

> Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

Docker是基于mamespace、cgroups、chroot等技术来构建容器，而容器是一个用户态的概念，在linux中并不存在。

而Docker与虚拟机的区别在于，虚拟机类似于古代的诸侯国，它是国中之国、系统中的系统。你要调用它就需要提供资源，以维持虚拟机的正常运行，它才会执行你的命令。而Docker容器则是一个郡或者一个县，归属于上层，没有自己直属的地盘，是一个次级管理者，对于资源只有使用权而没有所属权。 Docker则是通过*虚拟化*让容器进程直接在主机中运行。比起虚拟机，它更像是一种沙盒。每个容器内运行一个应用，不同的容间相互隔离，容器之间也可以建立通信机制。容器的创建和停止都十分快速，资源需求远远低于虚拟机。

而Docker要在系统中正常运行，就需要实现文件隔离与网络隔离。Docker使用了Linux的【Namespaces】技术来进行资源隔离，如PID Namespace隔离进程，Mount Namespace隔离文件系统，Network Namespace隔离网络等。

+	文件隔离是先通过Mount Namespace对不同的目录进行挂载，然后通过【chroot】命令重新设置根目录。
+	网络隔离是通过network namespaces为你的系统网络协议栈提供的视图实现的。

### 2. Docker的虚拟化技术
虚拟化是一种资源管理技术，是将计算机的各种实体资源，如服务器，网络，内存等抽象、转化后呈现出来，使用户以更好的方式来应用这些资源。虚拟化目标往往是为了在同一个主机上运行多个系统或者应用，从而提高资源的利用率，降低成本，方便管理及容错容灾。

+	虚拟机使用的是传统的虚拟化技术，在硬件层面进行虚拟化，模拟出一个操作系统。
+	Docker底层使用了LXC来实现，LXC将linux进程沙盒化，使得进程之间相互隔离，并且能够各自对各进程的资源分配。
+	LXC是Linux Container；提供了在单一可控主机节点上支持多个相互隔离的server container同时执行的机制。	
	
### 3. 核心概念
Docker 中有三个核心概念：Image、Container、Repository。
+	Image意为镜像，类似于虚拟机镜像。
+	Container意为容器，容器是镜像的运行环境，而镜像依托 Docker 的虚拟化技术，给容器这个空白的模板添加了独立的端口、进程、文件等“空间”，但实际上是在原系统上进行资源隔离而已。容器从镜像启动的时候，docker会在镜像的最上一层创建一个可写层，镜像本身是只读的，保持不变。
+	Repository意为仓库注册服务器，可以进行版本控制。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。仓库分为公开仓库（Public）和私有仓库（Private）两种形式。

### 4. docker的架构
![docker架构图](/image/linux-docker-1/jg1.png)

+	host是执行docker命令的数组机。daemon核心程序负责执行命令。registry是共享镜像库，daemen可以往registry推/拉镜像
+	在使用的时候，我们需要先拉一个映像，然后可以基于这个映像启动一个容器，容器启动后会生成一些相关的文件，然后在【stop】后可以通过【start】再次运行，可以通过【docker ps -a】查看所有的容器，如果不需要了，需要先停止容器，再通过【docker rm 容器ID】来删除容器。
+	如果需要删除镜像，那么在删除之前需要停止删除容器。
+	Docker系统有两个程序：docker服务端和docker客户端。其中docker服务端是一个服务进程，管理着所有的容器。docker客户端则扮演着docker服务端的远程控制器，可以用来控制docker的服务端进程。大部分情况下，docker服务端和客户端运行在一台机器上。可以通过【docker version】命令确认docker服务在运行并且可以通过服务端连接。



### 5. Docker容器的特点

+	灵活：即使最复杂的应用程序也可以容器化。
+	轻量级：容器利用并共享主机内核。
+	可互换：您可以即时部署更新和升级。
+	可移植：您可以在本地构建，部署到云并在任何地方运行。
+	可扩展：您可以增加并自动分发容器副本。
+	可堆叠：您可以垂直和动态地堆叠服务。

### 6. docker的使用场景
+	web应用的自动化打包和发布；
+	自动化测试和持续集成、发布；
+	在服务型环境中部署和调整数据库或其他的后台应用；
+	从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。

### 7. 导读——namespace
+	namespace 是 Linux 内核用来隔离内核资源的方式
+	![namespace](/image/linux-docker-1/namespace.png)

### 8. 导读——cgroups
Linux CGroup全称Linux Control Group，是Linux内核的一个功能，用来限制，控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等）。

特点　　　　　　
+	cgroup的api以一个伪文件系统的实现方式，用户的程序可以通过文件系统实现cgroup的组件管理

+	cgroup的组件管理操作单元可以细粒度到线程级别，另外用户可以创建和销毁cgroup，从而实现资源载分配和再利用

+	所有资源管理的功能都以子系统的方式实现，接口统一子任务创建之初与其父任务处于同一个cgroup的控制组

 
***
# 二、docker安装

### 1. 系统要求

+	Docker Assemble需要安装了Docker Engine的Linux，Windows或macOS Mojave。

### 2. 安装

##### 2.1 下载地址
+	[社区版](https://hub.docker.com/search/?type=edition&offering=community)
+	[专业版](https://hub.docker.com/search/?type=edition&offering=enterprise)

##### 2.2 Centos安装

【安装要求】Docker要求运行在Centos 7上，要求系统为64位，系统内核版本3.10以上


```
# 下载安装
sudo yum install docker

#  检查docker是否安装成功
$ docker version
docker run hello-world



# 安装之后启动 Docker 服务，并让它随系统启动自动加载。
$ sudo service docker start
$ sudo chkconfig docker on

```

##### 2.3 在windows使用Docker
Docker是基于Linux的隔离机制来构建的轻虚拟化技术，它的内核是Linux系统，不可能支持windows系统的内核的。但windows 系统也可以使用，那它又是怎么工作的呢？

+	实际上，在 2017 年 10 月发布的 Windows Server 1709 版本包含了 Windows 容器，Windows 容器是真正能够运行 Windows 应用程序的容器技术。于此同时，windows仿造Linux提出了 CGroup 和 Namespace 的概念，并提供出一个新的抽象层次 Compute Service，即宿主机运算服务（Host Compute Service，hcs），用于提供与 Docker 兼容的操作接口。
+	Docker 可以以两种形式运行在 Windows 上
	+	以 Hyper-V 虚拟机的形式运行 Linux 格式的容器（运行Linux程序）
	+	运行原生的 Windows 容器（运行 Windows 程序）
+	Docker的windows版本只支持Win10的专业版和企业版


***

# 三、docker使用



### 1.	dockerfile
Dockerfile是一个包含用于组合映像的命令的文本文档。可以使用在命令行中调用任何命令。 Docker使用【build】命令可以自上而下的读取Dockerfile中的指令自动生成映像。

Dockerfile的基本结构

+	基础镜像信息
+	维护者信息
+	镜像操作指令
+	容器启动时执行指令，’#’ 为 Dockerfile 中的注释。

常用指令
+	RUN	构建镜像时执行的命令
	+	```
	RUN <linux command> 执行时不存在就拉取
	RUN ["executable", "param1", "param2"]
	#	会生成缓存镜像，以便下一个版本的构建；可以用 【build --no-cache】避免生成缓存镜像
	```
+	WORKDIR	类似cd命令,当前工作路径
+	ADD 添加文件，压缩文件会自动解压
+	FROM	指定基础镜像，必须为第一个命令
	+	```
	格式：
　　FROM <image>
　　FROM <image>:<tag>
　　FROM <image>@<digest>
	实例：
	FORM neo4j:1.0
	```
+	EXPOSE	端口设置
+	ENV
+	MAINTAINER 构建人的信息
	+	```
	MAINTAINER <name>
	MAINTAINER gcyy
	```
+	VOLUME	目录挂载

### 2. docker的指令


+	docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
	+	```
		OPTIONS
			-d :分离模式: 在后台运行
			-i :即使没有附加也保持STDIN 打开【即默认可键盘输入】
			-t :分配一个伪终端```

+	docker search [name]	【搜索镜像】
+	docker pull 用户名/镜像名
	+	大部分的镜像是通过这样的格式来下载的，但一些经过官方验证的镜像可以直接用镜像名下载
+	docker run -dti 用户名/镜像名  【运行】
+	docker ps [OPTIONS]
	+	```OPTIONS
			-a 所有容器
			-l 显示最新创建的容器
			-q 只显示容器id
			-s 容器大小
		```
+	docker inspect [OPTIONS] NAME|ID [NAME|ID...]【查看容器运行状态】
	+	```
		OPTIONS
			-f 格式设置为字符串，不可单独使用
			-s 文件大小
		```
+	docker push [OPTIONS] NAME[:TAG] 【发布到docker官方服务器】
	+	```
		OPTIONS
			---disable-content-trust 跳过镜像验证
		```
### 2. 下载与运行镜像

在Docker前期，暂时用不到自己编写 dockerfile ，只需要知道有这个文件就可以了。在实际使用中，绝大部分常用的框架、数据库和软件都可以在Docker中下载到，入门阶段是不需要自己编写 Dockerfile。

##### 2.1 实例

```
# 查询
docker search redis
# 拉取镜像
docker pull redis

# 查看是否成功
docker images
docker image ls #下载的镜像

# 容器执行镜像
docker run -p 9999:6379 -d redis:latest redis-server
	# 【-p】 6379:6379	设置端口，容器端口6379被映射到主机端口9999
	# 【-d】 后台模式，会返回一串ID，类似【db……543】
	# 【redis:latest】 镜像:标签
	# 【redis-server】 别名


# 查看在运行的镜像
docker ps
docker container ls

docker logs db02d5924e34 #  docker logs 【ID前缀】

# 连接容器中的redis
docker exec -ti 006a redis-cli -h 127.0.0.1 -p 6379 
	# 【-t -i】 终端+键盘输入
	# 【006a】 容器ID的开头部分，匹配开头
	# 【redis-cli -h 127.0.0.1 -p 6379】  redis启动命令
```

***
 




# 参考文档
> [Docker和宿主机操作系统文件目录互相隔离的实现原理](https://www.jianshu.com/p/79d4e3da1291)
[Docker之四种网络模式 、容器的互通与隔离](https://blog.csdn.net/lilygg/article/details/88616218)
[linux中的namespace](https://cnblogs.com/cherishui/p/4237883.html	)
[从 0 开始了解 Docker](https://juejin.im/post/5ad3172c5188257ddb10109a)
[docker核心概念（镜像、容器、仓库）及基本操作](https://www.cnblogs.com/whych/p/9446032.html)
[在 Windows 上可以用 Docker 吗？](https://www.cnblogs.com/xiandnc/p/9327386.html)
[是时候Docker: 1 Docker导学](https://juejin.im/post/5d8c169c6fb9a04e0855a141)
[终于有人把 Docker 讲清楚了，万字详解！](https://blog.csdn.net/youanyyou/article/details/102674776)