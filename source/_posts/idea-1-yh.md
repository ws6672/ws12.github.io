---
title: idea使用指南（一）优化篇
date: 2020-03-23 23:22:20
tags: [tool]
---

# 一、设置优化

### 设置启动时不自动打开上次项目

*设置方法*
```
Setting->
	Appearance & Behavior - System Settin
	->	勾选 Reopen last project on startup 
```

### 关闭非必要的代码检查,IDEA会对代码进行检查，但有些检查是没有必要的

*设置方法*
```
File–>
	settings–>
		Editor–>
			Inspections–>
				可以关闭除error级别的其他检查，减少开支。但如果你配置很好，这一项也可以忽略。
```


### 设置字体大小
初始字体太小，可以适当调大字体。

*设置方法*
```
Editor->
	Font->
		调节size（字体大小）、Line spacing（行距）
```

### 主题设置
有些人不太喜欢IDEA默认的黑色主题，可以更改成其他的主题。

*设置方法*
```
File–>
	settings–>
		Appearance & Behavior–>
			Appearance–>
				配置页面中的Theme项
```

### 配置自动导包

*设置方法*
```
File–>
	settings–>
		Editor–>
			general–>
				Auto Import–>
					1.设置 Insert imports on paste为All，即所有项目都自动导包
					2.勾选Optimize imports on the fly,设置自动导包
```

### 加大编译进程和Maven的堆值

*设置方法*
```
File–>
	settings–>
		Build,Exception,Deployment->
			Compiler->
				设置Build process heap size 为1024
			Build Tools->
				Maven->
					Importing->
						设置VM options for importer 为-Xmx1024m
```

### 添加作者信息

路径：file->settings->Editor->File and Code Templates
![作者注释](/image/idea/idea-1-yh/zs.png)

```
/**
*@author 作者
*@date ${DATE} ${TIME}
*@description
*/
```
### 添加方法注释

路径：file->settings->Editor->Live Templates

1.添加模板组
2.添加模板，设置名称、解释、以及内容

内容如下：
```
	/**
	* @ Description TODO
	* @param $param$
	* @return $return$
	* @date $date$ $time$
	* @auther $user$
	*/
```


### 代码规范插件（阿里版）
1.打开IDEA，File-> Setteings->Plugins->Browse Repositories,
插件市场（ctrl+shift+A）,添加插件`Alibaba Java Coding Guidelines plugin support.`
2. 使用方法是：打开IDEA，点击tools--->安装的阿里编码规约，可以选择中英文切换，项目右键选择编码规约扫描就可以进行查看自己编码哪些地方不够好，进行修改哦。


# 二、性能优化

### 配置启动文件

打开idea，help->Edit Custom VM options,idea会自动复制bin目录下的idea64.exe.vmoptions文件；
文件路径（C:\Users\用户名\.IntelliJIdeaXXX\config\idea64.exe.vmoptions）

优化配置
```
-server //控制内存garage方式
-Xms1g  //初始堆内存
-Xmx2048m //最大堆内存
-Xverify:none // 关闭class加载验证
-XX:+DisableExplicitGC //禁止代码中显示调用GC；如果使用了java nio中的direct memory，则存在内存泄漏危险
-XX:MetaspaceSize=512m // Metaspace（元生代）扩容时触发FullGC的初始化阈值
-XX:ReservedCodeCacheSize=240m //设置代码内存容量
-XX:+UseConcMarkSweepGC  //设置年老代为并发收集
-XX:SoftRefLRUPolicyMSPerMB=1000 //每一MB空闲内存空间可以允许SoftReference对象存活多少毫秒
-XX:CICompilerCount=2 //编译线程的数目
-Dsun.io.useCanonPrefixCache=false
-Djava.net.preferIPv4Stack=true
-Djdk.http.auth.tunneling.disabledSchemes=""
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Djdk.attach.allowAttachSelf
-Dkotlinx.coroutines.debug=off
```

```
-XX:TraceClassLoading -XX:TraceClassUnloading
//通过日志追踪类加载和类卸载的情况，tomcat 启动则位于catalina.out中
```


##### metaspace
 metaspace ，元数据空间，专门用来存元数据的，它是jdk8里特有的数据结构用来替代perm
+	无论-XX:MetaspaceSize配置什么值，Metaspace的初始容量一定是21807104（约20.8m）；
+	Metaspace由于使用不断扩容到-XX:MetaspaceSize参数指定的量，就会发生FGC；且之后每次Metaspace扩容都会发生FGC；
+	如果Old区配置CMS垃圾回收，那么第2点的FGC也会使用CMS算法进行回收；
+	Meta区容量范围为[20.8m, MaxMetaspaceSize)；
+	如果MaxMetaspaceSize设置太小，可能会导致频繁FGC，甚至OOM


> [JVM参数MetaspaceSize的误解](https://blog.csdn.net/u011381576/article/details/79635867)