---
title: docker攻略（四）Docker的网络问题
date: 2019-10-22 23:50:46
tags: [docker]
---

Docker借助强大的镜像技术，让应用的分发、部署与管理变得史无前例的便捷。但是，用户并非一劳永逸的，我们在实际使用中还要面临很多的问题。例如，镜像多次提交后导致镜像过大、镜像源速度如何提速以及要在这篇文章探讨的Docker的网络问题。

Docker自身的网络主要包含两部分：
+	Docker Daemon的网络配置
+	Docker Container的网络配置。

#  一、daemon.json
docker 在启动的时候会先启动一个守护线程daemon，它在启动时是根据【daemon.json】配置文件运行的。我们可以通过修改配置文件来调整守护线程。
> [daemon文档](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)

该--config-file选项允许您以JSON格式设置守护程序的任何配置选项。此文件同样使用label作为关键字.配置文件中设置的选项不得与通过【--label】设置的选项冲突。如果在文件和标志之间复制选项，则无论其值如何，docker守护程序均无法启动。我们这样做是为了避免默默地忽略配置重载中引入的更改。

### 在Linux上支持的完整配置
```
{
	"authorization-plugins": [],
	"data-root": "",
	"dns": [],
	"dns-opts": [],
	"dns-search": [],
	"exec-opts": [],
	"exec-root": "",
	"experimental": false,
	"features": {},
	"storage-driver": "",
	"storage-opts": [],
	"labels": [],
	"live-restore": true,
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "10m",
		"max-file":"5",
		"labels": "somelabel",
		"env": "os,customer"
	},
	"mtu": 0,
	"pidfile": "",
	"cluster-store": "",
	"cluster-store-opts": {},
	"cluster-advertise": "",
	"max-concurrent-downloads": 3,
	"max-concurrent-uploads": 5,
	"default-shm-size": "64M",
	"shutdown-timeout": 15,
	"debug": true,
	"hosts": [],
	"log-level": "",
	"tls": true,
	"tlsverify": true,
	"tlscacert": "",
	"tlscert": "",
	"tlskey": "",
	"swarm-default-advertise-addr": "",
	"api-cors-header": "",
	"selinux-enabled": false,
	"userns-remap": "",
	"group": "",
	"cgroup-parent": "",
	"default-ulimits": {
		"nofile": {
			"Name": "nofile",
			"Hard": 64000,
			"Soft": 64000
		}
	},
	"init": false,
	"init-path": "/usr/libexec/docker-init",
	"ipv6": false,
	"iptables": false,
	"ip-forward": false,
	"ip-masq": false,
	"userland-proxy": false,
	"userland-proxy-path": "/usr/libexec/docker-proxy",
	"ip": "0.0.0.0",
	"bridge": "",
	"bip": "",
	"fixed-cidr": "",
	"fixed-cidr-v6": "",
	"default-gateway": "",
	"default-gateway-v6": "",
	"icc": false,
	"raw-logs": false,
	"allow-nondistributable-artifacts": [],
	"registry-mirrors": [],
	"seccomp-profile": "",
	"insecure-registries": [],
	"no-new-privileges": false,
	"default-runtime": "runc",
	"oom-score-adjust": -500,
	"node-generic-resources": ["NVIDIA-GPU=UUID1", "NVIDIA-GPU=UUID2"],
	"runtimes": {
		"cc-runtime": {
			"path": "/usr/bin/cc-runtime"
		},
		"custom": {
			"path": "/usr/local/bin/my-runc-replacement",
			"runtimeArgs": [
				"--debug"
			]
		}
	},
	"default-address-pools":[
		{"base":"172.80.0.0/16","size":24},
		{"base":"172.90.0.0/16","size":24}
	]
}
```

### 网络配置接口




### 什么是 dockerd
dockerd是管理容器的持久性线程。Docker在守护程序和客户端间可以使用不同的二进制可执行文件。要运行守护程序，使用【dockerd】命令。

·
```
	dockerd COMMAND
		Options:
			-v, --version
			-D, --debug  # 调试模式
			-b, --bridge string	# 对容器使用其他的网桥
			-H, --host list
			-G, --group string  # 设置socket套接字的用户组，默认为docker
```

要使用调试输出运行守护程序，请使用dockerd -D或添加【"debug": true】到daemon.json文件中。在docker中，它通过unix, tcp, 和fd三种不同类型的socket套接字来监听 [Docker引擎API](https://docs.docker.com/develop/sdk/).

```
	# Docker中不加密的信息通过2375端口传递，加密信息通过2376端口传递
	-H tcp://0.0.0.0:2375
	-H tcp://192.168.59.103:2375
	
	# 支持多个端口监听
	sudo dockerd -H unix:///var/run/docker.sock -H tcp://192.168.59.106 -H tcp://10.10.10.2
	
	# 也可以设置环境变量而不需要每次启动都设置端口监听
	export DOCKER_HOST="tcp://0.0.0.0:2375"
	docker ps
```

> 如果使用的是HTTPS 加密协议，需要使用TLS1.0或者更高版本


从Docker 18.09开始，Docker客户端支持通过SSH连接到远程守护程序：

```
	$ docker -H ssh://me@example.com:22 ps
	$ docker -H ssh://me@example.com ps
	$ docker -H ssh://example.com ps
	
	# ssh密匙如果受密码保护，还需要设置ssh-agent
```


### Daemon storage-driver
在linux中，Docker daemon支持几种不同的镜像层存储引擎驱动：aufs, devicemapper, btrfs, zfs, overlay 以及 overlay2.

|驱动|特点|使用|
|:--|:--|:--|
|aufs|基于Linux主内核不支持的补丁集，aufs允许容器共享可执行文件和共享库的内存|/|
|devicemapper|使用自动精简配置和写时拷贝（全体）的快照|配置【/var/lib/docker/devicemapper】|
|btrfs|运行速度非常快，在设备之间不共享可执行内存【仅ext4可使用】|dockerd -s btrfs -g /mnt/btrfs_partition|
|zfs|克隆共享块，仅缓存一次|dockerd -s zfs|
|overlay|非常快速的联合文件系统，以合并到linux主内核中，支持页面缓存共享|dockerd -s overlay|
|overlay2|使用相同的快速联合文件系统，但减少inode消耗|dockerd -s overlay2|




##### Failed to start Docker Application Container Engine.

这个错误是由于镜像拉取的速度太慢，通过命令修改镜像源后导致的。
1. `sudo systemctl start docker`
2. 检查daemon.json文件
	`vim /etc/docker/daemon.json`
	```
		# 修改为
		{
		  "registry-mirrors": ["http://harbor.test.com"], 
		  "insecure-registries": ["harbor.test.com","registry.cn-shenzhen.aliyuncs.com"], 
		  "max-concurrent-downloads": 10
		}
	```