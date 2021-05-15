---
title: Java集合源码分析（一）ArrayList 
date: 2020-02-15 15:12:38
tags: [java]
---



# ArrayList

###	1. 简述

ArrayList就是动态数组，用MSDN中的说法，就是Array的复杂版本，它提供了动态的增加和减少元素，实现了ICollection和IList接口，灵活的设置数组的大小等好处

###	2. 定义

*类定义*
`ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable`


*继承类*
+	`public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E>`


*接口*
+	List<E>	包含List通用操作方法
+	RandomAccess	标记接口，用于标明实现该接口的List支持快速随机访问	
+	Cloneable	标记接口，用于标明可调用clone深层克隆
	+	```
		// ArrayList 克隆方法
		public Object clone() {
			try {
			//  【反射-浅拷贝】调用父类的clone方法，实现浅克隆，返回值是Object
				ArrayList<?> v = (ArrayList<?>) super.clone();
			// 深拷贝
				v.elementData = Arrays.copyOf(elementData, size);
				v.modCount = 0;
				return v;
			} catch (CloneNotSupportedException e) {
				// this shouldn't happen, since we are Cloneable
				throw new InternalError(e);
			}
		}
	```
+	java.io.Serializable	可序列化


##### 2.1 向上转型
下图，为向上转型内存图。其中，对象A只能够访问对象B的a1、a2部分，即它本身拥有的属性。
![向上转型内存图](/image/java/util/list/向上.png)

 
> 注1：随机访问和顺序访问。
随机访问是指可以访问数据结构中任意位置的节点；譬如数组就可以随机访问，因为它底层是连续的存储块，可以直接访问。
顺序访问则需要从某个节点开始向后遍历才可以访问；譬如链表，在物理结构上是离散的，需要通过逻辑结构中的首节点进行遍历比较才可以访问其它位置的元素。 

> 注2：向上转型和向下转型.
向上转型：将子类转化为父类，转型后，（内存中）父类引用指向子类对象，这时候父类只可以调用父类拥有的属性和方法。在向上转型后，允许我们向下转型，将子类引用指向子类对象，先向上转型再向下转型就不会报错。
向下转型：将父类转化为子类。不能直接向下转型，因为子类引用不能直接指向父类对象。




> [JAVA DOC](https://tool.oschina.net/apidocs/apidoc?api=jdk_7u4)