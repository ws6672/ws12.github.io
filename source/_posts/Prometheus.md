---
title: Prometheus 监控CentOS7
date: 2020-03-27 10:29:52
tags: [linux]
---

### 什么是 Prometheus
Prometheus 意为普罗米修斯。 Prometheus是由SoundCloud开发的开源监控报警系统和时序列数据库(TSDB)。Prometheus使用Go语言开发，是Google BorgMon监控系统的开源版本。Prometheus和Heapster(Heapster是K8S的一个子项目，用于获取集群的性能数据。)相比功能更完善、更全面。Prometheus性能也足够支撑上万台规模的集群。

### 特点
普罗米修斯的主要特点是：

+	一个多维数据模型，其中包含通过度量标准名称和键/值对标识的时间序列数据
+	PromQL，一种灵活的查询语言 ，可利用此维度
+	不依赖分布式存储；单服务器节点是自治的
+	时间序列收集通过HTTP上的拉模型进行
+	通过中间网关支持推送时间序列
+	通过服务发现或静态配置发现目标
+	多种图形和仪表板支持模式

### 组件
+	***Server***：主要负责数据采集和存储，提供PromQL查询语言的支持。
+	client libraries：客户端库，用于检测应用程序代码
+	exporters：输出被监控组件信息的HTTP接口
+	***Alertmanager***：警报管理器，用来进行报警。
+	***PushGateway***：支持临时任务，客户端可以主动推送数据到网关，而 Prometheus定时从网关抓取数据。

### 监控原理
Prometheus的基本原理是通过HTTP协议周期性抓取被监控组件的状态，任意组件只要提供对应的HTTP接口就可以接入监控。

![prometheus](/image/linux/prometheus.png)

### 监控流程
1.暴露接口。抓取目标需要暴露一个http服务的接口给它定时抓取
2.定时抓取。Prometheus Daemon 负责定时去目标上抓取metrics(指标)数据
3.存储数据。Prometheus在本地存储抓取的所有数据，会通过一定规则进行清理和整理数据，并把得到的结果存储到新的时间序列中
4.可视化。Prometheus通过PromQL和其他API可视化地展示收集的数据。




### 配置 Prometheus 服务端
