---
title: 数据集成-水壶（一）入门须知
date: 2019-10-22 16:21:39
tags: [tool]
---

迫于工作需要，学习一下kettle。

# 一、使用

### 什么是kettle
Kettle是一款国外开源的ETL工具，纯java编写，可以在Windows、Linux、Unix上运行，数据抽取高效稳定。Kettle 中文名称叫水壶，该项目的主程序员MATT 希望把各种数据放到一个壶里，然后以一种指定的格式流出。


### 下载、安装与使用

[官网地址](https://community.hitachivantara.com/s/article/data-integration-kettle), 需要JDK环境；安装按默认的即可。安装完双击【spoon.bat】打开图形界面。

在kettle中，有两个比较大的概念：任务(kjb)与转换(ktr);任务是指kettle脚本需要完成的总目标，通常有一个起点，多个中间任务，一个终点。kettle通常需要一个数据源，编写完成后，可以以命令行的方式进行调用，通过命令参数获取对应的文件，即需要处理的数据主体。


### 主要的程序
在kettle中，有几个常用的程序，只需要了解它的用途即可。
+	kitchen.sh/kitchen.bat 用于在命令行执行脚本，其中任务的后缀为kjb，转换的后缀为ktr	
+	spoon.sh/Spoon.bat	图形界面，可以通过拖拽控件编写kettle的相关脚本
+	Pan.bat 

*常用的控件*
执行sql语句
![执行sql语句](/image/191022-kettle/zxyj.png)

JS脚本
![JS脚本](/image/191022-kettle/jsjb.png)

表输入
![表输入](/image/191022-kettle/bsr.png)

Excel输入
![Excel输入](/image/191022-kettle/esr.png)

增加序列
![增加序列](/image/191022-kettle/zjxl.png)

插入/更新
![插入/更新](/image/191022-kettle/crgx.png)

获取系统信息
![获取系统信息](/image/191022-kettle/hqxtxx.png)


### 运行程序
*通过spoon图形界面运行*

![spoon启动步骤](/image/191022-kettle/sqdbz.png)

*CMD运行*

【kitchen.bat的参数】
```
/rep        : Repository name
/user       : Repository username
/pass       : Repository password
/job        : The name of the job to launch
/dir        : The directory (dont forget theleading /)
/file       : The filename (Job XML) to launch
/level      : The logging level (Basic, Detailed,Debug, Rowlevel, Error, Nothing)
/logfile    : The logging file to write to
/listdir    : List the directories in the repository
/listjobs   : List the jobs in the specified directory
/listrep    : List the available repositories
/norep      : Do not log into the repository
/version    : show the version, revision and build date
/param      : Set a named parameter<NAME>=<VALUE>. For example -param:FOO=bar
/listparam : List information concerning the defined parameters in thespecified job.
/export     : Exports all linked resources of the specifiedjob. The argument is the name of a ZIP
file. 
```

实例
```
	# 使用脚本【TMDA_REPORT_ENERGY.kjb】转换文件【20191028.xls】
	kitchen /file E:/data-integration/kettle/TMDA_REPORT_ENERGY.kjb E:/data-integration/excel/20191028.xls
```

# 二、常见疑问【来自官网，留档备查】
*当我在Windows环境中启动Spoon.bat时，没有任何反应*
> 编辑Spoon.bat文件，然后：
在最后一行“ start javaw”中仅替换为“ java”。
在下一行添加“暂停”。
保存并重试。

*是否可以排序转换？*
>  这是不可能的，PDI转换的基本内容之一是所有步骤都是并行运行的。因此，您无法对其进行排序。这将需要对PDI进行体系结构更改，并且顺序处理也将导致处理速度非常慢。

*转型和工作之间有什么区别*
> 转换是关于将行从源移动到目标并进行转换。作业更多是关于高级流控制的：执行转换，失败时发送邮件，ftp'ing文件，...

# 三、导读
> [数据集成-水壶](https://community.hitachivantara.com/s/article/data-integration-kettle)
 [Kettle使用教程（一）](http://www.kettle.net.cn/1310.html)