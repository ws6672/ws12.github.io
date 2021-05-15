---
title: 面试记录（二）某贸易——外贸——两轮
date: 2020-10-12 20:50:38
tags: [ms]
---


### 一、选择题

1. volatile关键字是否能保证线程安全?
不能，volatile关键字保证的是变量的安全，保护变量避免被反复修改造成的读写错误。它不能保证线程对变量修改是有序的，即不保证线程安全。

2. 下面哪个流类属于面向字符的输入流？（C）
A、InputStreamReader
B、FileInputStream
C、BufferedWriter
D、ObjectInputStream

3. java可否不通过构造函数创建对象（A）
A. 可以 B. 不可以

4. 当检索一个压缩文件时，首先要建立压缩文件输入流对象。该对象( B)。
A．以选择的压缩文件为参数
B．以FileInputStream对象为参数
C．以InputStreamReader对象为参数
D．以BufferedReader对象为参数

[解析] 当输入一个Z中文件时要将Z中文件作为FileInputStream构造方法的参数，而 FileInputStream对象又作为ZipInputStream构造方法的参数出现。

5. 90的元素时，查找成功的比较次数为（　B　）。
A．1
B．2
C．3
D．9

6. Java程序的并发机制是（A）。
A．多线程
B．多接口
C．多平台
D．多态性

7. 下列选项中，不属于模块间耦合的是 ( C)
A．数据耦合
B．同构耦合(标记耦合)
C．异构耦合
D．公用耦合

[解析] 本题主要考查模块间耦合的类型。模块之间的耦合程度反映了模块的独立性，也反映了系统分解后的复杂程度。按照耦合程度从弱到强，可以将其分成5级，分别是：数据耦合、同构耦合、控制耦合、公用耦合和内容耦合。没有选项C中的异构耦合这种耦合方式。

8. 下列关于内部类的说法不正确的是 ( C)
A．内部类的类名只能在定义它的类或程序段中或在表达式内部匿名使用
B．内部类可以使用它所在类的静态成员变量和实例成员变量
C．内部类不可以用abstract修饰符定义为抽象类
D．内部类可作为其他类的成员，而且可访问它所在类的成员

[解析] 本题考查内部类的概念。内部类不仅可以用abstract修饰定义为抽象类，也可以用private或protected定义，所以选项C说法错误。


### 二、简述题

1. Java创建对象有几种方法？
+	通过New语句创建对象
+	通过反射
+	通过clone()方法创建对象
+	反序列化

2. 有一个长度为10万的整型数组[2,5,-2,6,-3,8,...],请统计数组中重复出现的元素及出现次数；
```
	Map<Integer,Integer> count(int[] nums) {
		Map<Integer,Integer> map = new HashMap<Integer,Integer>();
		for(int num:nums) {
			map.put(num, map.getOrDefault(num, 0)+1);
		}
		Iterator<Map.Entry<Integer, Integer>> it = map.entrySet().iterator();
		while(it.hasNext()) {
			Map.Entry<Integer, Integer> entry = it.next();
			if (entry.getValue()==1) {
			 it.remove();
			}
		}
		return map;
	}
```

3. 秒杀活动如何避免超卖?
通过redis写Lua脚本
+	存储商品数量信息
+	检测键是否过期
+	修改数量，数量为0则设置过期


