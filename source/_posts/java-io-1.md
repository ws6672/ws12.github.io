---
title: java IO（一）IO体系
date: 2020-03-31 23:12:37
tags: [java]
---

`java.io`包是一个基于流的操作包，它几乎包含了输入、输出所需的各种类。Java.io 包中的流支持很多种格式，比如：基本类型、对象、本地化字符集等等。输入流表示从目标读取数据，输出流表示将数据写入到目标。

***

### 一、类图

字节流的类图：

![字节流](/image/java/io/java-io-inout.png)


字符流的类图：
![字节流](/image/java/io/java-io-rw.png)

***

### 二、按操作对象划分
在`java.io`包中，支持以下多种操作对象：

+	节点流
	+	文件操作
		+	字节流：FileInputStream、FileOutputStream
		+	字符流：FileReader、FileWriter
	+	数组操作
		+	字节数组（byte[]）：ByteArrayInputStream、ByteArrayOutputStream
		+	字符数组（char[]）：CharArrayReader、CharArrayWriter
	+	管道操作
		+	字节流：PipedInputStream、PipedOutputStream
		+	字符流：PipedReader、PipedWriter
		
+	处理流
	+	基本数据类型操作
		+	DataInputStream、DataOutputStream
	+	对象（序列与反序列）操作
		+	ObjectInputStream、ObjectOutputStream
	+	缓冲操作
		+	字节流：BufferedInputStream、BufferedOutputStream
		+	字符流：BufferedReader、BufferedWriter
	+	打印控制
		+	PrintStream、PrintWriter
	+	转换控制
		+	InputStreamReader、OutputStreWriter
	+	~字符串操作~：在`jdk8`中被废弃。


***


### 三、按流的类型划分
从以下的体系图可以看出，可以按流的类型进行划分：
+	字符流（文本文件）
	+	Reader
	+	Writer
+	字节流（二进制文件）
	+	InputStream
	+	OutputStream

按流的类型划分：
![iostream体系图](/image/java/io/java-iostream.png)

`java.io`包相关类很多，结构冗杂，但常用的类只有那么几种：`文件操作`、`缓冲操作`以及`对象操作`。



> 字节流读取单个字节，字符流读取单个字符（一个字符根据编码的不同，对应的字节也不同，如UTF-8编码下，汉字字节数是不确定的，因为）


### 四、参考文章
> [JavaIO类图](https://blog.csdn.net/songyiwen_zhaogui/article/details/39611111)