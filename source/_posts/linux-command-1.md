---
title: linux常用命令（CentOS7）
date: 2021-01-10 16:17:16
tags: [linux]
---


>Linux的命令繁多，但常用的却总是那几个，熟能生巧。

# 一、文件与目录 

### 1. 文件操作

1.0 PWD 显示目录路径。
```
$ pwd
/home/cen
```

1.1 cd命令，用于切换当前工作目录。

```
-- 进入 '/ home' 目录' 
cd /home

-- 返回上一级目录
cd ..
-- 返回上两级目录 
cd ../..

-- 进入用户的主目录 
cd ~userNAME
-- 返回上次所在的目录
cd -  
```

1.2 ls(英文全拼:list files)命令，用于显示指定工作目录下的内容。

```
-- 查看目录中的文件
ls -F
-- 显示文件和目录的详细资料
ls -l
-- 显示隐藏文件
ls -a
-- 显示包含数字的文件名和目录名
ls *[0-9]* 
```

1.3 tree命令，显示文件和目录由根目录开始的树形结构。
```
sudo yum -y install tree
```

1.4 lstree命令，显示文件和目录由根目录开始的树形结构。

1.5 mkdir命令，创建目录(不存在)。

语法如下：
+	`mkdir [Options] dir`
+	Options
	+	-m【mode、文件权限，类似用chmod做初始化】
	+	-p【需要时创建上层目录】
+	dir是文件名；可以用空格分开同时创建多个文件夹
+	```
-- 创建一个叫做 'test' 的目录' 
mkdir test

-- 同时创建目录test test1
mkdir test test1

-- 创建一个目录树
sudo mkdir -p /usr/local/md

touch vsftpd.sh
# 删除文件  -f 强制删除 -r 是递归
rm -rf dir
```

1.6 mkfile命令
创建一个或多个适合用作挂载 NFS 的交换区域或本地交换区域的文件。

语法如下：`mkfile [ -nv ] size[b|k|m|g] filename ...`
+	-n     空文件，记录大小，写入数据前不分配磁盘块
+	-v 详细信息

1.7 cp(英文全拼:copy file)命令，主要用于复制文件或目录。
语法如下：`cp [ options ] ... Source  Dest`
+	options
	+	-b 覆盖前备份
	+	-l 快捷方式
	+	-i 覆盖提示
	+	-R 递归复制
	+	-v 细节输出
+	```
-- 复制一个文件
cp file1 file2
-- 复制一个目录下的所有文件到当前工作目录
cp dir/* .
-- 复制一个目录到当前工作目录
cp -a /tmp/dir1 .
-- 复制一个目录
cp -a test1 test2  
```

1.8 rm(英文全拼:remove)命令用于删除一个文件或者目录。

语法如下：`rm [options]... file...`
+	Options
	+	-f 忽略提示
	+	-r 递归删除
+	```
-- 删除一个叫做 'file1' 的文件' 
rm -f test1
-- 删除一个叫做 'test1' 的目录' 
rmdir test1
-- 删除一个叫做 'test1' 的目录并同时删除其内容 
rm -rf test1
-- 同时删除两个目录及它们的内容
rm -rf test1 test2  		
```

1.9 mv(英文全拼:move file)命令

用来为文件或目录改名、或将文件或目录移入其它位置。语法如下：
`mv [options] source... directory`	

+	options
	+	-b	备份
	+	-f	忽略
	+	-u	相同不更新
	+	-v	详细信息输出
+	```
-- 将源文件名 source_file 改为目标文件名 dest_file
mv source_file(文件) dest_file(文件)
-- 目录名 dest_directory 已存在，将 source_directory 移动到目录名 dest_directory 中；
-- 目录名 dest_directory 不存在则 source_directory 改名为目录名 dest_directory
mv source_directory(目录) dest_directory(目录)
```

1.10 ln(英文全拼:link files)命令是一个非常重要命令,它的功能是为某一个文件在另外一个位置建立一个同步的链接。

### 2. 文件权限修改

Linux文件权限一般使用如下几个命令：

```
chmod——改变文件或目录的权限

chown——改变文件或目录的属主（所有者）

chgrp——改变文件或目录所属的组

umask——设置文件的缺省生成掩码
```

[具体内容](/2019/09/18/linux-permission/)


### 3. 查看文件

+	ls 查看文件简略信息
+	ll 查看文件详细信息
+	pwd 显示文件路径
+	cd 移动到目录下
+	find 查询文件


### 4. 文件中的文本处理

Linux操作文本不像windows那么简单，它需要一定技巧，但要入门只需要记住以下的几个命令即可。

4.1 文本显示

+	cat 查看所有文本
+	less 查询大文本，可以通过上下浏览文本
+	tail 输出文件末的文本
	+	tail [ options ] ... [ file ]
	+	options
		+	-f 随着文件的增长输出内容
		+	-n【K】 输出行数，K表示行数
	+	eg
		+	`tail -fn5 hello.sh`


4.2 搜索与比较

+	grep 搜索文件中与给定模式匹配的行
	+	grep [ options ] PATTERN [ FILE ...]
	+	options
		+	-i 忽略大小写
		+	-e 使用正则表达式
		+	-m 最多匹配行
		+	-n 输出行号
		+	-q 匹配或有错误退出
	+	eg
		+	`grep -inm 2 he hello.sh `
		+	` grep -q he hello.sh `
+	diff 比较文件差异
	+	diff [ options ] FILES
	+	options
		+	-v 输出详细信息
		+	-a 视为文本
		+	-i 忽略大小写
	

# 二、压缩与解压

Linux下常见的压缩文件有tar、bzip2、zip、rar, 7z相对较少，许多开源的资源在linux中的链接大都是【tar】格式。而Linux中【7z】虽然使用较少，但压缩解压的性能尤为出色。

+ [.tar](#tar)  使用tar命令压缩或解压
+ [.bz2](#bzip2) 使用bzip2命令操作
+ [.gz](#gzip) 使用gzip命令操作
+ [.zip](#zip) 使用unzip命令解压
+ [.rar](#rar) 使用unrar命令解压


1. tar

+	语法
	+	tar [[-]function] [options] 文件名或者目录
	+	options
		+ -c: 建立压缩档案
		+ -x：解压
		+ -t：查看内容
		+ -r：向压缩归档文件末尾追加文件
		+ -u：更新原压缩包中的文件
		+ -z：解压tar.gz
		+ -j： 解压 tar.bz2
		+ -Z：解压tar.Z
		+ -v：显示所有过程
		+ -O：将文件解开到标准输出

2. bzip2

该命令用于解压以及压缩【bz2】格式的文件。

+	语法
	+	bzip2 [ -cdfkqstvzVL123456789 ] [ filenames ... ]
	+	bunzip2 [ -fkvsVL ] [ filenames ... ]
	+	bzcat [ -s ] [ filenames ... ]
	+	bzip2recover filename
	+	options
		+	-c 压缩
		+	-d 强制解压缩
		+	-t 测试
		+	-k 在压缩或解压缩期间保留（不删除）输入文件
		+	-s 减少内存使用，速度约为正常的一半
		

3. gzip

用于解压以及压缩【gz】格式的文件。

+	语法
	+	gzip options ...
	+	OPTIONS
		+	-c 压缩
		+	-d 解缩
		+	-t 测试压缩文件完整性
		+	-v 显示详细信息


4. zip 与 rar

zip 命令用于压缩【zip】格式的文件；rar 命令用于压缩【rar】格式的文件。在Centos系统中并没有这两个命令，可以用【yum】命令进行安装

```
yum -y install rar unrar
-- You need to be root to perform this command. 没有权限，需要升权
sudo -i
yum -y install rar unrar
```


# 三、其他

### 1. 系统命令
```
	-- 查看日期
	# date
	Tue Sep  3 21:53:17 CST 2019

	-- 查看控制字符
	# stty -a

	-- 修改密码
	# passwd

	-- 登陆和注销shell
	# login logout

	-- 显示路径
	# pwd
	/home/ali

	-- 显示或部分显示文件内容
	more, less, head tail:

	-- 挂起
	Ctrl-z 可以将前台进程挂起(suspend), 然后可以用 bg jobid 让其到后台运行。
	job & 可以直接让 job 直接在后台运行。
```

### 2. vim常用命令

```
	:q 退出
	:wq 保存并退出
	d 删除
	y 复制
	p 粘贴
	‘/内容’：

	:nu 查看当前行数
	:set nu 查看所有行数
```

### 3. 配置环境变量

```
-- 所有用户
vim /etc/profile 

-- vim ~/.bashrc 当前用户
export NEO4J_HOME=/usr/local/neo4j/neo4j-community-3.5.9
export PATH=$PATH:$NEO4J_HOME/bin

-- 刷新环境变量
source /etc/profile
``` 
 

### 4. cat的使用

cat可以在终端输出一个文件的内容；可以将文件的内容复制到另一个文件中；可以合并文件；可以创建新的文本文件。

例如：
```
cat filename
cat file1 file2 
cat file1 file2 > newcombinedfile
cat < file1 > file2 #copy file1 to file2
```



### 4. yum错误

4.1 报 Loaded plugins: fastestmirror

```
	# 1. 关闭 fastestmirror 插件
	vim  /etc/yum/pluginconf.d/fastestmirror.conf

		[main]
		enabled=0
		verbose=0
		always_print_best_host = true
		socket_timeout=3

	# 2. yum取消使用插件
	vim /etc/yum.conf
	 
		plugins=0

	# 3. yum清空缓存并重建
	yum clean all
	yum makecache
```

4.2 `/var/run/yum.pid` 已被锁定，PID 为 XXXX 的另一个程序正在运行。

```
sudo rm -f /var/run/yum.pid
```

# 参考文献
> [ss64 ———— 操作系统命令参考网站](https://ss64.com/bash/)
[Linux上，最常用的一批命令解析（10年精选）](https://mp.weixin.qq.com/s/9RbTGQ4k4s92mrSf2xJ5TQ)
[Linux压缩和解压汇总](https://www.cnblogs.com/rwxwsblog/p/4505934.html)
[使用yum 出现 Loaded plugins: fastestmirror](https://www.cnblogs.com/lzcys8868/p/8184681.html)
