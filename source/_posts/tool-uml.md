---
title: UML中类的关系
date: 2021-05-16 13:42:35
tags: [soft-skills]
---


1. 依赖（Dependency）：A类中存在方法形参为B类

实体之间一个“使用”关系暗示一个实体的规范发生变化后，可能影响依赖于它的其他实例（图D）。更具体地说，它可转换为对不在实例作用域内的一个类或对象的任何类型的引用。其中包括一个局部变量，对通过方法调用而获得的一个对象的引用（如下例所示），或者对一个类的静态方法的引用（同时不存在那个类的一个实例）

2. 关联（Association）：A类有成员变量是B类

实体之间的一个结构化关系表明对象是相互连接的。UML箭头是可选的，它用于指定导航能力。如果没有箭头，暗示是一种双向的导航能力。在Java中，关联（图E）转换为一个实例作用域的变量，就像图E的“Java”区域所展示的代码那样。可为一个关联附加其他修饰符。

+	合成/组合（Composition）：A类的成员变量是B类，且必定存在
	+	合成（图G）是聚合的一种特殊形式，UML箭头暗示“局部”在“整体”内部的生存期职责。合成也是非共享的。所以，虽然局部不一定要随整体的销毁而被销毁，但整体要么负责保持局部的存活状态，要么负责将其销毁。局部不可与其他整体共享。但是，整体可将所有权转交给另一个对象，后者随即将承担生存期职责。
+	聚合（Aggregation）：A类通过存在数组之类的结构存放B类
	+	聚合（图F）是关联的一种形式，UML箭头代表两个类之间的整体/局部关系。聚合暗示着整体在概念上处于比局部更高的一个级别，而关联暗示两个类在概念上位于相同的级别。聚合也转换成Java中的一个实例作用域变量。
+	合成与聚合的区别：聚合即A中可能有B对象，B对象不是A的一部分；合成即A中一定有B对象，并且生成A对象的同时一定生成B对象。

3. 泛化（Generalization）：A类继承于B类

泛化（图H）表示一个更泛化的元素和一个更具体的元素之间的关系。泛化是用于对继承进行建模的UML元素。

4. 实现（Realization）：A类实现了B接口

面向对象中表示接口的实现。