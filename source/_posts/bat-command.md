---
title: 批处理命令
date: 2019-09-05 10:30:22
tags: [系统]
---

window bat语法

## 基础

1.bat文件是以.bat或者.cmd为拓展名的文件，每一行都是一条DOS命令
2.流程控制。使用if、goto以及for循环进行流程控制
3.大小写不敏感
4.MS-DOS或者是纯DOS下的命令，都是单线程的。只有等一条命令执行完毕才能继续执行

## 几个常用的命令

#### 进程控制
```
	taskkill /f /im notepad.exe [终止进程]
	Pause 暂停命令
```

#### 跳转命令
```
%~dp0
	%~dp0[获取当前路径] d是[drive]的缩写，意为驱动盘；p是[path]的缩写，意为路径

	cd %~dp0 [进入批处理目录]
	cd %~dp0bin\ [进入批处理目录下的bin目录]
	%cd% [执行的路径]

	
GOTO 跳转命令
	goto label —— label前需要用":" 表示为标签，标签起名需有意义
```

#### 显示与打印
```
@ 命令
	表示运行时不显示该行命令，写在命令的前面
echo 打印命令
	ECHO [ON | OFF] 打开回显或关闭回显功能。
	ECHO [message] 显示信息。
	ECHO 当前回显（即上条命令返回的结果）
	
Rem 注释命令
	Rem message ——  不会被执行
```
 

#### 占位符

*命令参数*
```
	if "%1" == "" (
		pause
	) else (
		echo add post
		@hexo new %1
		@hexo s
		@start chrome local:8080
	)
	命令行输入> xx.bat tt
		tt在批处理文件中就会取代%1，因为它是第一个参数
```

*set命令*
```
set/p option=请输入你的选择:
	if "%option%" == ""

set vl = vl
```


#### 判定符

```	
EQU - 等于
	NEQ - 不等于
	LSS - 小于
	LEQ - 小于或等于
	GTR - 大于
	GEQ - 大于或等于
```

#### 调用外部程序

```
start chrome http://localhost:4000
```

## 实例

#### 1 用批处理运行 neo4j

```
echo off
echo neo4j-start
cd %~dp0bin\
neo4j console
echo neo4j-start finished
echo on
```

#### 2 用批处理调用hexo
```
@ echo off
echo 开始添加文章
cd %~dp0

Rem 循环标号
:circle

Rem 设置文章名称
@set/p option=文章名：
if "%option%" == "" (
	echo 文章名不可以为空
	pause
) else (
	echo 添加文章：%option%
	hexo new %option%
	pause
)
Rem 判定是否循环
@set/p et=退出（y/n）：
@ if %et% NEQ "y" (
	goto circle
)

@ echo on
```

#### 3 用批处理复制文件

```
@ echo off
Rem 用于复制指定目录下的文件到指定目录中
echo 开始复制文件
cd %~dp0

Rem 循环标号
 
:circle

Rem 设置值

:circle_dirs
@set/p dirs=请输入复制目录：
if "%dirs%" == "" (
	echo 目录不可以为空
	pause
	goto circle_dirs
) 

:circle_mb
@set/p mb=请输入目标目录：
if "%mb%" == "" (
	echo 目录不可以为空
	pause
	goto circle_mb
) 
Robocopy %dirs% %mb% /E

Rem 循环判定
@set/p et=是否退出（y/n）:
@ if %et% NEQ "y" (
	goto circle
)

@ echo on
```

## 巧用windows的cmd
学会和会用是两码事，在学会命令后，在实操的时候，往往会有一些教程中没有的涉及到的盲点。
总结于下，留档备忘。

####  在bat中调用第三方程序
例如，在bat中调用notepad++

```
cd C:\Program Files (x86)\Notepad++
>"notepad++.exe" C:\Users\wsz\Desktop\191009.md
```

优化使用
1. 配置环境变量【path：C:\Program Files (x86)\Notepad++】
2. cmd命令：notepad++ C:\Users\wsz\Desktop\191009.md



