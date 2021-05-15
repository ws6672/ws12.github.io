---
title: crawlab（一）入门
date: 2020-02-24 16:44:58
tags: [python]
---

# Crawlab
基于Golang的分布式爬虫管理平台，支持多种编程语言以及多种爬虫框架.

Crawlab主要解决的是大量爬虫管理困难的问题，例如需要监控上百个网站的参杂scrapy和selenium的项目不容易做到同时管理，而且命令行管理的成本非常高，还容易出错。Crawlab支持任何语言和任何框架，配合任务调度、任务监控，很容易做到对成规模的爬虫项目进行有效监控管理。



# 安装
1. 安装docker
2. 配置镜像源
```
// /etc/docker/daemon.json
	{
		"registry-mirrors": ["https://registry.docker-cn.com"]
	}
```
3. 配置镜像
`docker pull tikazyq/crawlab:latest`

### 长任务爬虫
长任务爬虫（Long-Task Spiders）是一种特殊的 自定义爬虫，这种爬虫跑任务不会停止，一般会一直获取消息队列中的 URL 并抓取，只有当用户主动停止或遇到错误时才会停止运行。长任务爬虫通常是分布式运行的，为的是有效的利用网络带宽资源和其他计算资源，将分布式节点的效率利用到极致。典型的例子就是基于 Scrapy 的分布式爬虫 scrapy-redis。

要启用长任务爬虫，只需要在 创建爬虫 或 爬虫详情 中打开 “是否为长任务开关” 就可以了。

您可以在 爬虫列表 中选择 “长任务” 来过滤所有长任务爬虫。