---
title: docker攻略（三）镜像的精简
date: 2019-10-13 11:17:38
tags: [linux, docker]
---

从虚拟机到 Docker容器 ，对于开发与运维来说是质的突破。Docker容器不仅仅是一种新的运维技术，还是一种新的技术思维。但是，Docker也不是没有缺点的。由于 Docker容器拥有的镜像仓库的功能，导致镜像会越来越大。而且，即便Docker容器小于虚拟机，但在我基于【Tomcat】构建一个很小的应用时，镜像也达到了555M。在实际使用中，一个镜像可能就是1G起步了。



# 一、基础

### 镜像层

Docker的层用于保存镜像的上一版本和当前版本之间的差异。就像Git的提交一样，如果你与其他存储库或镜像共享它们，就会很方便。
实际上，当你向注册表请求镜像时，只是下载你尚未拥有的层。这是一种非常高效地共享镜像的方式。
但高效是通过空间换取的，镜像的版本越高，它的体积也就越大。所以，可以通过减少镜像层、合并镜像层来精简镜像。

### 联合命令

在一些 DockerFiles中，有许多都会使用联合命令，即使用【&&】连接命令。
例如
```

	FROM ubuntu
	RUN apt-get update && apt-get install vim

# 替代

	FROM ubuntu
	RUN apt-get update
	RUN apt-get install vim
	
```
之所以使用【&&】可以压缩镜像是因为从Docker 1.10开始，COPY、ADD和RUN语句会向镜像中添加新层。即一个RUN命令就是一层，而使用联合命令就可以减少添加的层数。

### 使用 .dockerignore
可以在Dockerfile对应目录下面，添加【.dockerignore】文件，将需要忽略的文件名添加进去，用于在构建过程中忽略不需要的文件。



# 二、Docker多阶段构建
当Git存储库变大时，你可以选择将历史提交记录压缩为单个提交。事实证明，在Docker中也可以使用多阶段构建达到类似的目的。

> Docker多阶段构建是17.05以后引入的新特性，旨在解决编译和构建复杂的问题。减小镜像大小。因此要使用多阶段构建特性必须使用高于或等于17.05的Docker。

Dockerfile中的多阶段构建，其实就是有多个from，即可以基于不同镜像在不同阶段中进行构建，然后通过 【COPY --from】来从上一个阶段获取文件，而上一个阶段中构建使用的中间镜像都被丢弃，不会被保存在最后的镜像中。

### Dockerfile
	```

		FROM node:8 as part1 # 阶段起别名为【part1】
		WORKDIR /app
		COPY package.json index.js ./
		RUN npm install
		FROM node:8
		COPY --from=part1 /app /  # 从第一阶段build复制目录/app/
		# COPY --from=0 /app / 这样写也可以，即从第一个from获取
		EXPOSE 3000
		CMD ["index.js"]
		
	```
	
### 停在一个阶段
在构建容器时可以不跑到容器的最后一个阶段，即可以运作到某个阶段就停止。
	`docker build --target part1 -t node-multi-stage .`

### 使用外部镜像
在构建镜像时，不仅仅可以从上一个阶段中获取，也可以从外部进行获取
	`COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf`



# 三、distroless

### 什么是 distroless
[distroless](https://github.com/GoogleContainerTools/distroless)容器映像是由google构建的，是docker镜像精简版。distroless只包含您的应用程序及其运行时依赖项。它们不包含包管理器、shell或任何您可能在标准Linux发行版中找到的其他程序。使用它确实可以减小镜像的大小，但更重要的是可以提高镜像的安全性。

我们可以通过 Google提供的工具[bazel](https://docs.bazel.build/versions/master/bazel-overview.html) 来使用它。


### 优点
通过使用 distroless，不仅仅可以减小镜像的大小，还可以提高镜像的安全性。
+	更少的攻击面：通过避免打包的docker映像上出现不必要的程序，减少了从内部或外部进行任何攻击的机会。
+	避免映像漏洞：使用特定于操作系统的常规docker映像，如果有任何漏洞或应用安全补丁，则需要对其进行更新。
+	明确不可变性：在运行docker容器时，即使您偶然在docker内部进行手动操作，也必须即操即离，即不能长时间的进入容器内部。
+	安全的密匙变量:我们基本上使用基于文件的密匙或环境变量。有人可以用某种方式进入到docker容器中，但是由于Distroless没有shell访问权限，这个密匙就会被清楚地暴露出来。

 
### Get XXX_ping: dial tcp XXX i/o timeout
Docker 拉取资源所用的官方镜像源太差，导致响应超时。网上搜了一下，可以通过使用国内的镜像源[daocloud](https://www.daocloud.io/mirror) 来加快进行的拉取速度。

```
	# 安装 会下载一个脚本，用于重新设置daemon.json的镜像源
	curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
	# 重启
	sudo systemctl restart docker 
```


参考文献
>	[三个技巧，将Docker镜像体积减小90%](http://www.docker.org.cn/docker/176.html)
[Docker多阶段构建最佳实践](http://dockone.io/article/8179)
[Distroless is for Security if not for Size](https://medium.com/@dwdraju/distroless-is-for-security-if-not-for-size-6eac789f695f)
[docker 配置国内镜像源 linux/mac/windows](https://www.cnblogs.com/wangjunjiehome/p/9276207.html) 
