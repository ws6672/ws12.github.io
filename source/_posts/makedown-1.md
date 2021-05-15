---
title: makedown使用技巧
date: 2019-10-22 15:47:57
tags: [tool]
---

### 一、makedown绘制表格
在md中绘制表格非常的麻烦，而我们可以通过[exceltk](https://github.com/fanfeilong/exceltk)来将excel中的表格转为md格式的文件。

*命令*
	```
		exceltk.exe -t md -xls a.xls # 将xls格式
	```

*结果*
	```
		|命令|删除内容|速度|带where|回滚|
		|:--|:--|:--|:--|:--|
		| delete |用于删除表数据，|删除速度慢|可|可|
		| drop |删除表结构和表数据|删除速度快|不可|不可|
		| truncate |用于删除表数据，|删除速度快|不可|不可|
	```

*带格式的表格*

```
	exceltk -t md -a r -xls example.xlsx,  【-a】 选项 用于控制表格的对齐方式
		-a l: aligin left
		-a r: aligin right
		-a c: aligin center
```

如果需要的话，也可以用于转换为json格式的数据或者Tex格式的数据

```
	exceltk.exe -t json -xls example.xls
	exceltk.exe -t tex -xls example.xls
```

*通过bat快速转换*
为了方便日常使用，写了一个简单的bat支持输入文件名然后快速转换，但需要文件、bat、程序在同一个文件夹下，比较局限。打算以后有空，尝试将bat加入右键，实现文件右键就可以一键生成对应格式的md文件。
```
	@ echo off
	Rem 脚本需要放置在exceltk同一目录
	echo 开始转换【文件需要放在与脚本同一目录】
	cd %~dp0

	Rem 循环标号
	:circle

	@set/p option=文件名：
	if "%option%" == "" (
		echo 文件名不可以为空
		pause
	) else (
		echo 转换：%option%
		exceltk.exe -t md -xls %option%
		pause
	)

	@ echo on
```

### 二、图片与链接
链接
	```
		#	格式：[描述](链接地址)
			[exceltk](https://github.com/fanfeilong/exceltk)
	```
图片
	```
		#	格式：![描述](链接地址)
		![带标签的属性图模型](/image/19831/neo4j-1-1.png)
	```
	
	
TODO
> 待补充