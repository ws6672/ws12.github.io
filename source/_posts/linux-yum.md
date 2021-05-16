---
title: 【yum】疑难手册
date: 2019-09-23 21:26:16
tags: [linux]
---

###  报 Loaded plugins: fastestmirror

```
# 1. 关闭 fastestmirror 插件
vim  /etc/yum/pluginconf.d/fastestmirror.conf

	[main]
	enabled=0
	verbose=0
	always_print_best_host = true
	socket_timeout=3

# 2. yum取消使用插件
vim /etc/yum.conf
 
	plugins=0

# 3. yum清空缓存并重建
yum clean all
yum makecache
```


### 参考文档

[使用yum 出现 Loaded plugins: fastestmirror](https://www.cnblogs.com/lzcys8868/p/8184681.html)

