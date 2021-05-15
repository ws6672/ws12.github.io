---
title: JAVA 微基准测试套件(JMH)
date: 2020-03-30 09:59:19
tags: [java]
---

### 定义

***术语***
**JMH** ：JAVA 微基准测试套件，即一个基于`基准测试`的测试工具，用于测试代码的性能
**基准测试** ：基准测试是指通过设计科学的测试方法、测试工具和测试系统，实现对一类测试对象的某项性能指标进行定量的和可对比的测试。[详情](https://baike.baidu.com/item/%E5%9F%BA%E5%87%86%E6%B5%8B%E8%AF%95/5876292)

**白盒测试**：是指实际运行被测程序，通过程序的源代码进行测试而不使用用户界面。这种类型的测试需要从代码句法发现内部代码在算法、溢出、路径和条件等方面的缺点或者错误，进而加以修正。
**黑盒测试**：又称功能测试、数据驱动测试或基于规格说明的测试，是通过使用整个软件或某种软件功能来严格地测试,，而并没有通过检查程序的源代码，或者很清楚地了解该软件的源代码程序具体是怎样设计的。

***特点***
+	可重复
+	可观测——过程可视化
+	可展示——结果图形化
+	可执行——可定位、可分析
+	真实



### 使用
```
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.19</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.19</version>
</dependency>
```

