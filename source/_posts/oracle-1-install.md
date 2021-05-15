---
title: oracle探索笔记（一）
date: 2019-08-29 21:45:47
tags: oracle
---

# Oracle安装与卸载

## oracle的安装
1. 下载：http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html，解压并打开setup.exe
2.1 配置安全更新，取消即可，但取消的话安装可能不通过，可以cmd到安装根目录下，输入命令 
	```
	setup.exe -ignorePrereq -J"-Doracle.install.db.validate.supportedOSCheck=false"
	```
2.2 选择安装选项，默认即可
2.3 选择系统类别，及桌面版与服务器版
2.4 oralce主目录用户配置，默认即可.
2.5 安装版本有企业版与标准版，企业版是运维用的，开发就用标准版即可。
	还需要设置口令，有一定规范， Oracle 建议输入的口令应该至少长为 8 个字符, 至少包含 1 个大写字符, 1 个小写字符和 1 个数字 [0-9]。
2.6 安装等待


## oracle卸载
由于自行下载安装的oracle的版本不对，需要重新安装，于是在此体验一下oracle的卸载流程

1.关闭oracle的相关服务，
2.卸载工具的位置dbhome_1\deinstall.bat，管理员启动，一路回车；其中有一项要填是或者否的，不能填y。
3.清注册表
4.清环境变量，我在安装中没有清环境变量，又配置了oracle_sid,结果在安装的配置阶段跑不下去，只能选择跳过。 

```
+ HKEY_LOCAL_MACHINE|SOFTWARE|ORACLE 删除此键。 
+ HKEY_LOCAL_MACHINE|SYSTEM|CurrentControlSet|Services删除以oracle为首的键。 
+ HKEY_LOCAL_MACHINE|SYSTEM|CurrentControlSet|Services|Eventlog|Application 删除以oracle为首的键。 
+ HKEY_CLASSES_ROOT删除以Ora，Oracle，Orcl，EnumOra 为前缀的键。 
+ HKEY_CURRENT_USER|Software|Microsoft|Windows|CurrentVersion|Explorer|MenuOrder|Start Menu|Programs删除以oracle为首的键。 
```

## Database Configuration Assistant的安装
由于环境变量没有删除干净，oracle_sid 不是默认的orcl，在安装该配置无法进行，需要手动安装。

1.删除环境变量oracle_sid或者修改环境变量为orcl
2.打开程序 Oracle - OraDB12Home1\配置和移植工具\Database Configuration Assistant
3.编码格式设置为UTF-8,如果没有需要，建议不要勾选容器服务器，不然后期创建用户需要加“C##”的前缀

## Net Configuration Assistant的安装
这个也是环境变量的锅。
1.打开程序 Oracle - OraDB12Home1\配置和移植工具\Net Configuration Assistant
2.添加监听程序从而生成tnsnames.ora和listener.ora
3.添加本地网络服务名配置

本地网络服务名配置使用户可以通过tcp访问oracle数据库。监听程序会在相应路径(product\12.2.0\dbhome_1\admin)下生成tnsnames.ora和listener. 
```
配置环境变量
	NLS_LANG，变量值为SIMPLIFIED CHINESE_CHINA.ZHS16GBK
	TNS_ADMIN，变量值为E:\app\wsz\virtual\product\12.2.0\dbhome_1\admin
```