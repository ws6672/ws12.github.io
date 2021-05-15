---
title: Kettle（二）处理各种excel数据
date: 2020-01-03 09:37:30
tags: [tool]
---

### 处理带多行合并单元格的excel数据

在处理excel文件时，有时候会处理到带多行合并的数据，例如：
![excel](/image/kettle/kettle-excel-1.png)

kettle抽取后的数据格式：
![抽取后的数据](/image/kettle/kettle-excel-2.png)

抽取后按行插入即可，但有些是空值【null】， 所以需要用类似下面的方法来填充单元格的空值。

*代码*
```
//Script here
var sbname,sbtype,dwtype;
// 当存在 第一行（等于undefined）或 多行合并首行（不为null）这两种情况时，对【sbname,sbtype,dwtype】进行赋值
if (sbname === undefined||a1 != null) {

	 sbname = a1;
}

if (sbtype === undefined||a2 != null) {

	 sbtype = a2;
}

if (dwtype === undefined||a3 != null) {
	if (a3 == "长度/km")
		a3 = "长度";
	if (a3 == "容量/MVA")
		a3 = "容量";


	 dwtype = a3;
}
```
> 注：	前三列 a1,a2,a3 值映射为sbname,sbtype,dwtype




### 多列转换为多行
```
业务需求：
	拥有分月数据表，需要将数据处理后将每列（每月）存储到数据中。由于kettle是按行处理数据的，所以需要将多列转换为多行。
实现方法：
	通过行转列、字段选择组件进行转换。
```
![多列转换为多行](/image/kettle/kettle-excel-3.png)


### 字段合并
```
业务需求：
	修改列值或进行合并
实现方法：
	可以通过 JavaScript组件合并字段。
```
![字段合并](/image/kettle/kettle-excel-4.png)
 
