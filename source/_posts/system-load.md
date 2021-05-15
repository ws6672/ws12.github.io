---
title: Win重装系统(一) 备份软件数据
date: 2019-08-29 21:12:27
tags: 系统
---

# mysql数据库备份
1.导出成sql文件
2.mysqldump工具备份
3.将mysql server下的data文件夹进行复制，重装系统后重新安装mysql，替换data文件夹。


## mysqldump工具备份
*备份整个数据库*

```
$> mysqldump -u root -h host -p dbname > backdb.sql
备份数据库中的某个表

$> mysqldump -u root -h host -p dbname tbname1, tbname2 > backdb.sql
备份多个数据库

$> mysqldump -u root -h host -p --databases dbname1, dbname2 > backdb.sql
备份系统中所有数据库

$> mysqldump -u root -h host -p --all-databases > backdb.sql
```

# maven的备份
1.将maven复制到非系统盘下
2.重装系统后重新配置环境变量，类似路径path /maven/bin