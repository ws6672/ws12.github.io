---
title: java 垃圾回收
date: 2020-03-16 23:30:47
tags: [java]
---

# 前言
Java是我在学习完C语言后接触到的第一个面向对象的语言，它的主要优点之一就是不需要担心内存回收的问题。在C语言中，内存回收需要开发者编写代码进行处理，如果稍微不注意，程序就被埋下了地雷。与之相反，Java从一开始就会自动释放未使用的内存。但这并不意味着Java开发人员不需要了解Java处理内存的基础知识。因为它仍然可能存在内存泄漏和瓶颈。

### 内存结构
Java将其内存分为两部分：
堆：实例，变量等数据
非堆内存/永久代:
> 注：在JDK8中，Metaspace取代了永久代（Perm）


mvn archetype:generate \
	  -DinteractiveMode=false \
	  -DarchetypeGroupId=org.openjdk.jmh \
	  -DarchetypeArtifactId=jmh-java-benchmark-archetype \
	  -DgroupId=org.sample \
	  -DartifactId=test \
	  -Dversion=1.0