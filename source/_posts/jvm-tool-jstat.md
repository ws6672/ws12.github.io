---
title: JVM工具 jstat
date: 2020-03-31 12:08:47
tags: [java]
---

###	一、什么是 jstat

Jstat是JDK自带的一个轻量级的`JVM统计监视工具`，全称“Java Virtual Machine statistics monitoring tool”。

Jstat位于java的bin目录下，主要利用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。


Jstat 可以用来监视VM内存内的各种堆和非堆的大小及其内存使用量。

###	二、使用

程序所在路径：`C:\Program Files\Java\jdkXXX\bin\jstat.exe`

可以通过以下帮助命令获取使用方法：
![jstat-help](/image/jvm/jvm-jstat-help.png)


***-option***
```
C:\Users\e>jstat -options
	-class				显示ClassLoad的相关信息
	-compiler			显示JIT编译的相关信息
	-gc					显示和gc相关的堆信息
	-gccapacity			显示各个代的容量以及使用情况
	-gccause			显示垃圾回收的相关信息
	-gcmetacapacity		显示metaspace的大小
	-gcnew				显示新生代
	-gcnewcapacity		显示新生代大小和使用情况
	-gcold				显示老年代
	-gcoldcapacity		显示老年代的大小
	-gcutil				显示垃圾收集信息
	-printcompilation	输出JIT编译的方法信息
 ```

***实例***

查找java进程pid，可以通过Java自带的程序查看
```
C:\Users\e>jvisualvm
```

其中，`7664`为pid。

查看类加载情况
```
C:\Users\e>jstat -class 7664
Loaded  Bytes  Unloaded  Bytes     Time
 10566 19232.3        1     0.9      43.28
 
 -Loaded 装载类数量
 -Bytes 转载类占用的字节数
 -Unloaded	卸载类数量
 -Bytes	卸载类占用的字节数
 -Time	装载和卸载类所花费的时间
```

查看新生代情况
```
C:\Users\e>jstat -gcnew 7664
 S0C    S1C    	  S0U   S1U    TT MTT  DSS      EC       EU       YGC     YGCT
2048.0 10752.0    0.0 10610.3  9  15 12288.0  59392.0  31943.5     11    0.123

 -S0C    第一个survivor区的容量
 -S1C    第二个survivor区的容量
 -S0U  	 第一个survivor区已使用的容量
 -S1U    第二个survivor区已使用的容量 
 -TT	 持有次数限制
 -MTT    持有次数限制
 -DSS	 期望的幸存区大小 
 -EC     eden容量  
 -EU     eden已使用容量
 -YGC    年轻代GC次数
 -YGCT   年轻代GC时间

```

查看老生代情况
```
C:\Users\e>jstat -gcold 7664
MC       MU      	CCSC     CCSU       OC          OU       YGC    FGC    FGCT			GCT
52352.0  49243.3   7552.0   6984.9    107520.0     12774.7     11     2    0.12	3    0.246

-MC     元空间容量
-MU		元空间已使用容量 
-CCSC   压缩类空间大小
-CCSU   压缩类空间已使用大小
-OC     年轻代容量   
-OU     年轻代已使用容量 
-YGC	年轻代中gc次数      
-YGCT  	年轻代中GC时间  
-FGCT   老年代GC次数  
-GCT	GC消耗总时间

```

查看GC信息

```
C:\Users\e>jstat -gc 7664
S0C    S1C   	 S0U    S1U    EC       EU        OC         OU       MC      MU
CCSC     CCSU    YGC      YGCT    FGC  FGCT      GCT
7168.0 26112.0 7163.4  0.0   148480.0 87806.8  2784256.0   66216.8   116480.0 11
2838.0 11520.0 10708.3     44    0.326  28      2.280    2.605

-S0C    第一个survivor区的容量
-S1C    第二个survivor区的容量
-S0U  	 第一个survivor区已使用的容量
-S1U    第二个survivor区已使用的容量 
-EC     eden容量  
-EU     eden已使用容量     
-OC     年轻代容量   
-OU     年轻代已使用容量 
-MC     元空间容量
-MU		元空间已使用容量 
-CCSC   压缩类空间大小
-CCSU   压缩类空间已使用大小
-YGC	年轻代中gc次数      
-YGCT  	年轻代中GC时间  
-FGC  	老年代GC次数
-FGCT   老年代GC次数  
-GCT	GC消耗总时间
```

### 三、遇到的问题
***Error: could not open `C:\Program Files\Java\jre1.8.0_241\lib\amd64\jvm.cfg'***
+	`java -version` 失败,同时安装多个jdk后，导致异常。
+	处理方式：将jdk相关的环境变量移动到最前。



### 四、参考文章
> [java监控工具（jps，jstat，jstack，jmap，jvisualvm等）](https://wzktravel.github.io/2015/08/06/java-monitor/)
[java高分局之jstat命令使用](https://blog.csdn.net/maosijunzi/article/details/46049117)