---
title: springboot实现热部署
date: 2021-06-11 11:52:03
tags: [springboot]
---

# 热部署

实现方式：
+	使用springloaded配置pom.xml文件，使用mvn spring-boot:run启动
+	使用springloaded本地加载启动，配置jvm参数 `-javaagent:<jar包地址> -noverify`
+	devtools工具包（修改代码后自动重启，期间无法操作；配置简单）

以上几种方式都需要经过配置，但是IDEA可以安装插件JRebel，可以用于提高代码编写效率。JRebel的作用如下：
+	立即加载 95% 的典型代码更改，包括复杂的 Java EE 更新
+	通过消除应用程序或服务器重启，编写代码的速度提高 17%
+	将发布的可预测性提高 +8%
+	避免分心并专注于编写高质量的代码

JRebel是收费的，需要激活：
+	本地激活：[下载自己机器系统相对应的工具](http://github.com/ilanyu/ReverseProxy/releases/tag/v1.4)
+	在线激活：[在线创建 GUID](https://www.guidgen.com/)
	+	```
		选择Connect to online licensing service，填写以下数据
			https://jrebel.qekang.com/{GUID}
			你的邮箱
	```
在线激活可能会失效，可以尝试本地激活。


run和debug一一对应jrebel run和jrebel debug。启动后项目就可以使用jrebel做热部署，除了涉及到修改mysql、redis地址以及重要的配置无法热部署需要重启外，大部分的业务逻辑修改都不需要重启服务器即可生效，联调接口时效率非常高。


# 相关资料

[JRebel 激活](http://cache.baiducontent.com/c?m=uCt7n9U1340jAx2Q9Aa2yMVA_CNmxY2mIHPSr7klbofR5b-5z6ah2ej6O2iwVwxgyTjvUg0x90jhijv80gX6P33dq-QkehpaFnl_qaecjeqGVX01tFboIyu6dwQLOsxv&p=82759a42d48a33e00cb9c7710f5e&newp=c339ca5399934ea85ab2c7710f0792695d0fc20e3bddda01298ffe0cc4241a1a1a3aecbf2c271305d5c27c6101aa4e5eecf735763d0034f1f689df08d2ecce7e70&s=f2f88e17abe1608b&user=baidu&fm=sc&query=JRebel+%2Dsite%3Acsdn%2Enet&qid=8954b1110003d122&p1=4)

