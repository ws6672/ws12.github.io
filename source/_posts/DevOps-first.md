---
title: 初识 DevOps
date: 2020-03-16 23:02:40
tags: [devops,运维]
---

### DevOps  
DevOps（Development和Operations的组合词）是一组过程、方法与系统的统称，用于促进开发（应用程序/软件工程）、技术运营和质量保障（QA）部门之间的沟通、协作与整合。
它是一种重视“软件开发人员（Dev）”和“IT运维技术人员（Ops）”之间沟通合作的文化、运动或惯例。透过自动化“软件交付”和“架构变更”的流程，来使得构建、测试、发布软件能够更加地快捷、频繁和可靠。
它的出现是由于软件行业日益清晰地认识到：为了按时交付软件产品和服务，开发和运维工作必须紧密合作。

+	Development和Operations 开发人和运维
+	可以把DevOps看作开发（软件工程）、技术运营和质量保障（QA）三者的交集。
+	DevOps 是一个完整的面向IT运维的工作流，以 IT 
自动化以及持续集成（CI）、持续部署（CD）为基础，来优化程式开发、测试、系统运维等所有环节。



以下几方面因素可能促使一个组织引入DevOps：
1、使用敏捷或其他软件开发过程与方法
2、业务负责人要求加快产品交付的速率
3、虚拟化和云计算基础设施（可能来自内部或外部供应商）日益普遍
4、数据中心自动化技术和配置管理工具的普及
5、有一种观点认为，占主导地位的“传统”美国式管理风格（“斯隆模型 vs 丰田模型”）会导致“烟囱式自动化”，从而造成开发与运营之间的鸿沟，因此需要DevOps能力来克服由此引发的问题。

![devops_first_1](/image/DevOps/devops_first_1.png)


### 相关工具的使用
工具链

+ 代码管理（SCM）：GitHub、GitLab、BitBucket、SubVersion
+ 构建工具：Ant、Gradle、maven
+ 自动部署：Capistrano、CodeDeploy
+ `持续集成（CI）`：Bamboo、Hudson、Jenkins
+ 配置管理：Ansible、Chef、Puppet、SaltStack、ScriptRock GuardRail
+ `容器`：Docker、LXC、第三方厂商如AWS
+ 编排：Kubernetes、Core、Apache Mesos、DC/OS
+ 服务注册与发现：Zookeeper、etcd、Consul
+ 脚本语言：python、ruby、shell
+ 日志管理：ELK、Logentries
+ 系统监控：Datadog、Graphite、Icinga、Nagios
+ 性能监控：AppDynamics、New Relic、Splunk
+ 压力测试：JMeter、Blaze Meter、loader.io
+ 预警：PagerDuty、pingdom、厂商自带如AWS SNS
+ HTTP加速器：Varnish
+ 消息总线：ActiveMQ、SQS
+ 应用服务器：Tomcat、JBoss
+ Web服务器：Apache、Nginx、IIS
+ 数据库：MySQL、Oracle、PostgreSQL等关系型数据库；cassandra、mongoDB、redis等NoSQL数据库
+ 项目管理（PM）：Jira、Asana、Taiga、Trello、Basecamp、Pivotal Tracker

### 采用 DevOps 会遇到的几个陷阱

+	使用工具越多，维护工具间的联系成本越大
+	工具使用成本过大，导致难以兑现；即学习成本过高
+	停止对测试的探索。不应当仅仅将管道局限于'构建，测试'，还可以进行部署、监视
+	不应当在出问题后才进行安全检查。

### DevOps 施行指标

+	部署频率 
+	变更的准备时间
+	恢复服务的时间
+	变更失败率

# 参考文章
> [DevOps简介](https://blog.csdn.net/chuixue24/article/details/83623246)
[4 Common Pitfalls When Adopting DevOps](https://dzone.com/articles/4-common-pitfalls-when-adopting-devops)
[What is DevOps?（翻译）什么是DevOps？](https://www.jianshu.com/p/d3bb62e9f72c)