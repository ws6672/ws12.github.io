---
title: 装.NET Framework3.5 卡在正在下载所需文件
date: 2019-08-29 21:31:42
tags: [tool]
---

# windows8.1 装.NET Framework3.5 卡在正在下载所需文件
	由于惠普笔记本散热差，所以需要下载hp coolsense软件降热，但需要系统支持.NET Framework3.5 .

1.系统安装包，解压sources 到盘
2.用管理员运行指令dism.exe /online /enable-feature /featurename:NetFX3 /Source:F:\sources\sxs