---
title: 面试-linux
date: 2020-06-29 21:40:37
tags: [ms]
---



Linux常用操作

### 1. 抓包——tcpdump

TCPDump可以将网络中传送的数据包完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。

##### 1.1 实例

```
# 默认启动，监控第一个网络接口上流过的所有数据包
tcpdump

# 查看网络设备,可以看到带设备号的网络接口`lo`
ifconfig


# 设置监听本地接口，即localhost;需要提权
#当 ping 127.0.0.1，tcpdump会抓取包
tcpdump -i lo
```


##### 1.2 其它语法

```
监视指定主机的数据包

	-- 打印所有进入或离开sundown的数据包.
	tcpdump host sundown
		
	-- 也可以指定ip,例如截获所有210.27.48.1 的主机收到的和发出的所有的数据包
	tcpdump host 210.27.48.1 

	--打印helios 与 hot 或者与 ace 之间通信的数据包
	tcpdump host helios and \( hot or ace \)

	-- 截获主机210.27.48.1 和主机210.27.48.2 或210.27.48.3的通信
	tcpdump host 210.27.48.1 and \ (210.27.48.2 or 210.27.48.3 \) 
		
	-- 打印ace与任何其他主机之间通信的IP 数据包, 但不包括与helios之间的数据包.
	tcpdump ip host ace and not helios


	-- 如果想要获取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包，使用命令：
	tcpdump ip host 210.27.48.1 and ! 210.27.48.2

	-- 截获主机hostname发送的所有数据
	tcpdump -i eth0 src host hostname
		
	-- 监视所有送到主机hostname的数据包
	tcpdump -i eth0 dst host hostname
		
监视指定主机和端口的数据包

	-- 如果想要获取主机210.27.48.1接收或发出的telnet包，使用如下命令
	tcpdump tcp port 23 and host 210.27.48.1
	
	-- 对本机的udp 123 端口进行监视 123 为ntp的服务端口
	tcpdump udp port 123 
```


### 2. IO瓶颈处理

+	内核，作为Linux中极为复杂的部分，被广泛应用于各种计算机，包括嵌入式设备、手持设备、笔记本电脑、服务器、数据库服务器、视频服务器和体型巨大的超级计算机等场景，以上种种场景包含了许许多多不同的请求类型，这其中就包括对用户输入的响应（例如，流媒体音乐、视频或者其他交互性不能被中断的请求），并且要保证设备需要有良好的I/O性能，以确保数据能够被完整存储。因此内核需要通过调度器来满足这些请求，例如具有很高的I/O吞吐量的系统，就需要I/O调度器来对I/O进行管理。
+	调度器通过在内核中调度系统资源来实现低延迟的输入、更加流畅的交互请求、更快的I/O请求，或者以上这些需求的集合。而调度程序主要关注CPU资源，但也会兼顾其他系统资源（如，内存，输入设备，网络等）

##### 2.1 I/O体系

![I/O体系](/image/linux/iosystem.png)

I/O体系 分层如下：
+	VFS虚拟文件系统：内核要跟多种文件系统打交道，内核抽象了这VFS，专门用来适配各种文件系统，并对外提供统一操作接口。
+	磁盘缓存：磁盘缓存是一种将磁盘上的一些数据保留着RAM中的软件机制，这使得对这部分数据的访问可以得到更快的响应。磁盘缓存在Linux中有三种类型：
	+	Dentry cache
	+	Page cache
	+	Buffer cache
+	映射层：内核从块设备上读取数据，这样内核就必须确定数据在物理设备上的位置，这由映射层（Mapping Layer）来完成。
+	通用块层：由于绝大多数情况的I/O操作是跟块设备打交道，所以Linux在此提供了一个类似vfs层的块设备操作抽象层。下层对接各种不同属性的块设备，对上提供统一的Block IO请求标准。
+	I/O调度层：大多数的块设备都是磁盘设备，所以有必要根据这类设备的特点以及应用特点来设置一些不同的调度器。
+	块设备驱动：块设备驱动对外提供高级的设备操作接口。
+	物理硬盘：这层就是具体的物理设备。


##### 2.2 I/O调度器

传统的旋转式磁盘通过将盘片旋转到特定的扇区来对数据进行读写，这个过程被称作“寻道”，寻道时间很可能在“寻找数据”时耗费大量的时间。I/O调度器就是可以极大的优化寻道时间的方法，通过将I/O请求合并到磁盘上类似的位置来实现优化。比如，通过I/O调度器，可以将相邻扇区的I/O请求分至一组，从而大大缩短寻道时间，提高磁盘I/O的总体响应时间。

Linux I/O调度器介于通用块层和块设备驱动程序之间，所以它接收来自通用块层的请求，试图合并请求，并找到最合适的请求下发到块设备驱动程序中。之后块设备驱动程序会调用一个函数来响应这个请求。


### 3. strace命令

strace命令是一个集诊断、调试、统计与一体的工具，我们可以使用strace对应用的系统调用和信号传递的跟踪结果来对应用进行分析，以达到解决问题或者是了解应用工作过程的目的。当然strace与专业的调试工具比如说gdb之类的是没法相比的，因为它不是一个专业的调试器。
strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间。
 

##### 3.1 语法
```
strace  [  -dffhiqrtttTvxx  ] [ -acolumn ] [ -eexpr ] ...
    [ -ofile ] [-ppid ] ...  [ -sstrsize ] [ -uusername ]
    [ -Evar=val ] ...  [ -Evar  ]...
    [ command [ arg ...  ] ]

strace  -c  [ -eexpr ] ...  [ -Ooverhead ] [ -Ssortby ]
    [ command [ arg...  ] ]
```

##### 3.2 参数配置

```
-c 统计每一系统调用的所执行的时间,次数和出错的次数等.
-d 输出strace关于标准错误的调试信息.
-f 跟踪由fork调用所产生的子进程.
-ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的filename.pid中,pid是各进程的进程号.
-F 尝试跟踪vfork调用.在-f时,vfork不被跟踪.
-h 输出简要的帮助信息.
-i 输出系统调用的入口指针.
-q 禁止输出关于脱离的消息.
-r 打印出相对时间关于,,每一个系统调用.
-t 在输出中的每一行前加上时间信息.
-tt 在输出中的每一行前加上时间信息,微秒级.
-ttt 微秒级输出,以秒了表示时间.
-T 显示每一调用所耗的时间.
-v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出.
-V 输出strace的版本信息.
-x 以十六进制形式输出非标准字符串
-xx 所有字符串以十六进制形式输出.
-a column 设置返回值的输出位置.默认 为40.
-e expr 指定一个表达式,用来控制如何跟踪.格式：[qualifier=][!]value1[,value2]...
qualifier只能是 trace,abbrev,verbose,raw,signal,read,write其中之一.value是用来限定的符号或数字.默认的 qualifier是 trace.感叹号是否定符号.例如:-eopen等价于 -e trace=open,表示只跟踪open调用.而-etrace!=open 表示跟踪除了open以外的其他调用.有两个特殊的符号 all 和 none. 注意有些shell使用!来执行历史记录里的命令,所以要使用\\.
-e trace=set 只跟踪指定的系统 调用.例如:-e trace=open,close,rean,write表示只跟踪这四个系统调用.默认的为set=all.
-e trace=file 只跟踪有关文件操作的系统调用.
-e trace=process 只跟踪有关进程控制的系统调用.
-e trace=network 跟踪与网络有关的所有系统调用.
-e strace=signal 跟踪所有与系统信号有关的 系统调用
-e trace=ipc 跟踪所有与进程通讯有关的系统调用
-e abbrev=set 设定strace输出的系统调用的结果集.-v 等与 abbrev=none.默认为abbrev=all.
-e raw=set 将指定的系统调用的参数以十六进制显示.
-e signal=set 指定跟踪的系统信号.默认为all.如 signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号.
-e read=set 输出从指定文件中读出 的数据.例如: -e read=3,5
-e write=set 输出写入到指定文件中的数据.
-o filename 将strace的输出写入文件filename
-p pid 跟踪指定的进程pid.
-s strsize 指定输出的字符串的最大长度.默认为32.文件名一直全部输出.
-u username 以username的UID和GID执行被跟踪的命令
```

##### 3.3 实例 

***例1***
```
$ strace cat /dev/null

# 输出结果如下，每一行都是一条系统调用。左边是调用函数及参数，右边是调用的返回值。
execve("/usr/bin/cat", ["cat", "/dev/null"], [/* 53 vars */]) = 0
brk(NULL)                               = 0x1f5c000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f0e02137000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
......
```

***例2***
```
# 统计每一个系统调用花费的时间，并输出到文件test
$ strace -o test -c cat /dev/null
$ vim test 

% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0         2           read
  0.00    0.000000           0         4           open
  0.00    0.000000           0         6           close
  0.00    0.000000           0         5           fstat
  0.00    0.000000           0         8           mmap
  0.00    0.000000           0         4           mprotect
  0.00    0.000000           0         1           munmap
  0.00    0.000000           0         4           brk
  0.00    0.000000           0         1         1 access
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         1           arch_prctl
  0.00    0.000000           0         1           fadvise64
------ ----------- ----------- --------- --------- ----------------
```
### 4. 参考文章
 
> [Linux tcpdump命令详解](https://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html)
[Linux strace命令](https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316692.html)
[strace命令](https://man.linuxde.net/strace)