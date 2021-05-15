---
title: Java集合（一）入门
date: 2021-01-09 23:16:24
tags: [Collection]
---

Oracle将集合和Java集合框架（Java Collections Framework（JCF））定义为：“一个`代表一组对象`的对象（例如经典的 Vector类）。集合框架是用于表示和操作集合的统一体系结构，使集合可以独立于实现细节进行操作。”。

# 一、概述

集合固定增长或缩小，且支持泛型，而Map中的值没有特定顺序，但是可以通过`key`进行检索。

Collection 的含义可以参考：
+	它的日常含义是“汇编或一组事物”，
+	组成Java Collections Framework的接口和类的集合，
+	可以容纳一组对象（例如数组）的数据结构（例如盒子或容器），
+	util.Collection接口，这是两个主要JCF接口之一
+	util.Collections，一个实用程序类，可以帮助修改或操作Java集合。

而与数组不同，所有Collection的大小都可以动态。



###	泛型与初始化大小
在集合框架中，广泛的使用了泛型；在使用集合框架时，应当尽量对它设置近似的初始化值，知道集合的确切大小甚至是近似大小要比确定默认集合大小更好，因为如果默认集合大小如果过小，那么就会进行多次扩充。一个适合的集合大小，可以提高程序的性能。而这种细节的处理，通常是普通程序员与软件工程师之间的差距。

`Collection<String> myList = new ArrayList<String>(100);`

*泛型定义的例子*
```
public interface MyInterface<E, T> {
E read();
void process(T o1, T o2);
}
```

*在初次操作之前不要进行初始化*
```
void addToMap(String key, Object x) {
  getOrCreateList().put(key,x);
}
// ConcurrentHashMap 提供给多个线程使用，需要考虑线程安全
private Map getOrCreateMap() {
    if (map == null) {
        // Make sure we aren't outpaced by another thread
        synchronized (this) {
            if (map == null) map = new ConcurrentHashMap();
        }
    }
    return map;
}
```


### Collections 与 Arrays 工具类
 `java.util.Collections `与` java.util.Arrays`都提供了多个静态方法。
 +	`Collections`提供了便捷的方法 sort、shuffle、reverse、search、min、max，这大大增强了`Collections`接口的通用性。
 +	`Arrays`也支持对数组进行排序或搜索
 
### 深克隆集合

深克隆集合是Java开发人员不断努力的目标，我们一般通过重写`Object.clone()`方法实现深克隆。jdk8 引入的新功能`Streams`,可以对集合中的元素进行深克隆。因为stream创建对象时，修改生成的实例将不会影响其来源。这允许从一个来源创建多个实例。     

例如：
```
Dog dog1 = new Dog("Puppy", 4);
Dog dog2 = new Dog("Tom", 5);
Dog dog3 = new Dog("Hen", 3);
Dog dog4 = new Dog("Jen", 7);
List<Dog> dogs = new ArrayList<>();
dogs.add(dog1);
dogs.add(dog2);
dogs.add(dog3);
dogs.add(dog4);
//clone with java 8
List<Dog> clonedList = dogs.stream().map(Dog::new).collect(Collectors.toList());

// 对原始集合所做的任何修改都不会影响克隆的集合
```

### 集合与封装

面向对象编程的核心之一是封装：不应当允许调用者直接访问类的字段。JavaBeans 规范是作为Java强制执行封装的方法引入的，编写JavaBeans意味着您需要将类的字段设为私有，仅通过getter和setter方法公开它们。这样做是很繁琐的，但JavaBean规范在使用集合类型时还是很有用的。

一个较为合适的处理集合的方式如下，这样处理就避免了将对`非null`的判定分散在代码中。
```
public void setMyStrings(List<String> s) {
    if (s == null) {
        this.myStrings.clear();
    } else {
        this.myStrings = s;
    }
}
```

即便我们将`null`传递给`setter`，它也将保留有效的引用。

###  数据排序

1. 数据存储于集合

数据在`Collection`（ArrayList、HashSet、TreeSet）中，
```
private void sortNumbersInArrayList() {
        List<Integer> integers = new ArrayList<>();
        integers.add(5);
        integers.add(10);
        integers.add(0);
        integers.add(-1);
        System.out.println("Original list: " +integers);
        Collections.sort(integers);
        System.out.println("Sorted list: "+integers);
        Collections.sort(integers, Collections.reverseOrder());
        System.out.println("Reversed List: " +integers);
}

// 对于  Set，我们需要将HashSet转换为ArrayList，或者使用TreeSet对数据进行排序
private void sortNumbersInHashSet() {
        Set<Integer> integers = new HashSet<>();
        integers.add(5);
        integers.add(10);
        integers.add(0);
        integers.add(-1);
		
        
		/*
		* System.out.println("Original set: " +integers);
		* Collections.sort(integers); This throws error since sort method accepts list not collection
		* HashSet无法直接输出，因为是无序的
		*/
		
		// 转换为ArrayList输出
        List list = new ArrayList(integers);
        Collections.sort(list);
        System.out.println("Sorted set: "+list);
        Collections.sort(list, Collections.reverseOrder());
        System.out.println("Reversed set: " +list);
		// 使用TreeSet对数据进行排序
		Set<Integer> reversedIntegers = new TreeSet(Collections.reverseOrder());
        reversedIntegers.add(5);
        reversedIntegers.add(10);
        reversedIntegers.add(0);
        reversedIntegers.add(-1);
        System.out.println("Reversed set: " + reversedIntegers);
 }

```

如果自定义类的集合需要通过`Collections`排序，那么自定义类需要实现接口`Comparable`

```
 @Override
  public int compareTo(Object o) {
     return (((Student) o).getName()).compareTo(this.getName());
  }
```


2. 数据存储于数组

如果数据存储在数组中，需要通过`Arrays.sort(Object[] a )`进行排序；

3. 通过比较器实现排序

继承*Comparator*类

+	`外比较器`,定义在其它类中,不需要对类进行修改；方法定义：`int compare(T o1,T o2)`;
+	使用场景：当我们想对无法修改的类的实例进行排序时；要求根据用例按不同字段进行排序。
+	```
public class DomainComparator implements Comparator<Domain>
{
    public int compare(Domain domain1, Domain domain2)
    {
        if (domain1.getStr().compareTo(domain2.getStr()) > 0)
            return 1;
        else if (domain1.getStr().compareTo(domain2.getStr()) == 0)
            return 0;
        else 
            return -1;
    }
}
```

实现*Comparable*接口

+	`内比较器`，需要进行自定义排序时，需要实现该接口，并且编写自然比较方法`compareTo`
	+	int compareTo (object)
		+	返回值：大于对象，返回正数；等于对象，返回0；小于对象，返回负数。
+	当我们在定义类时知道排序的顺序时，应该使用；并且在其他情况下，我们将不要求Collection / array按其他字段进行排序



### 总结

工具类的使用
+	Collections
+	Arrays

遍历Collection的几种方法
+	steam API和foreach
+	增强for循环
+	c类型的for循环
+	c类型的iterator遍历

遍历数组的方法
+	自定义排序方法
+	通过`Arrays.sort`方法

使用比较器
+	继承*Comparator*类
+	实现*Comparable*接口


***

# 二、Collection

集合框架是一个关于数据存储结构的体系，用来存储对象信息。大部分集合类位于java.util包下，支持多线程的集合类位于java.util.concurrent包下。

### Collection 层次结构

![集合框架结构图](/image/java/util/java_collection_1png.png)

从图中可以总结以下几点：
根节点是`Iterable`，表示所有的集合类都可以通过迭代器进行遍历。
往下是Collection 接口，提供了集合通用的操作方法，包括添加元素、删除元素、检查是否包含元素、获取元素个数
Collection 接口又有三个子接口：`Set` `List` `Queue` 
+	`Set`：不包含重复元素
	+	`AbstractSet`
		+	`HashSet`: 底层是哈希结构；非有序集合
			+	`LinkedHashSet`: HashSet和List的组合；底层是双向链表，可以按插入顺序遍历
	+	`SortedSet`：顾名思义，是一个有序的Set集合
		+	`NavigableSet`：提供浏览有序的Set集合，提供检索大于或小于给定Set元素的下一个元素的方法。
			+	`TreeSet`：底层是树结构；通过`（比较器）comparator`接口使元素按一定规则进行排序
+	`List`：有序集合、可重复
	+	`ArrayList`:默认实现；底层数组，可调整列表大小；遍历、读取快；添加、删除慢
	+	`Vector`：底层数组，可调整列表大小，*仅在需要线程安全和同步时才使用它*。
	+	`Stack`：LIFO（后进先出）阵列.Vector的子类，也是线程安全的。
	+	`LinkedList`:元素的双链表。提供从中间位置快速添加和删除的功能。还实现了`Deque` 接口.
+	`Queue`：*先进先出*；队列
	+	`Deque`:双向队列接口，支持在两端插入和删除元素的线性集合
		+	`ArrayDeque`： 是 Deque的实现，使用数组进行存储
		+	`LinkedList`：也是 Deque的实现
		+	`PriorityQueue`:一个基于优先级的无界优先级队列。
			+	优先级队列的元素按照其自然顺序进行排序，或者根据构造队列时提供的 Comparator 进行排序，具体取决于所使用的构造方法。
			+	该队列不允许使用 null 元素也不允许插入不可比较的对象(没有实现Comparable接口的对象)。
		



### 集合的公共方法

 

1. Iterator 与 Iterable

+	Iterator 是java.util包下的接口，迭代器类；
+	Iterable 是java.lang下的接口，封装了Iterator接口，实现该接口就可以使用`增强for循环`。


2. forEach 方法与 forEachRemaining方法的区别
forEach()方法是Iterable接口在1.8的时候引入的一个默认方法；forEachRemaining()是Iterator接口在1.8的时候引入的一个默认方法

对于大多数实现了Iterable接口的集合，可以多次调用forEach遍历，并将通过元素进行多次传递。
相反，forEachRemaining()使用迭代器Iterator的所有元素，并且第二次调用它将不会做任何事情，因为不再有下一个元素。

使用场景：获得对应集合的迭代器Iterator，然后您可以开始迭代，next()直到达到某个条件，然后使用
forEachRemaining()操作该Iterator上的其余部分。

3. 通过增强for循环，高速迭代

```
    @Test
    public void test() {
        Iterable<String> stringIterable = Arrays.asList("AA","BB","CC","DD");
        for (String name: stringIterable) {
            System.out.println(name);
        }
    }
```

4. 在JDK8可以通过`foreach()`和 lambda表达式进行遍历 

```
@Test
public void test1() {
	Iterable stringIterable = Arrays.asList("AA","BB","CC","DD");
	stringIterable.forEach(System.out::println);

	Arrays.asList("AA","BB","CC","DD").forEach(str->{
		System.out.println(str);
	});
}
```

5. 通过 `Iterator`迭代
```
    @Test
    public void test() {
        Iterable stringIterable = Arrays.asList("AA","BB","CC","DD");
        Iterator iterator = stringIterable.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
```


6. 几个常用操作
```
    @Test
    public void test2() {
        List<String> list = Arrays.asList("AA","BB","CC","DD");
        Set<String> names = new LinkedHashSet<>();
//        添加元素
        names.add("Mary");
        names.add("Annie");
        names.add(null);
//        合并集合
        names.addAll(list);

//        删除单个元素
        if (names.remove("AA")) {
            System.out.println("remove success");
        }
//        删除多个元素
        if (names.removeAll(Arrays.asList("Mary","Annie"))) {
            System.out.println("remove all success");
        }
//        删除匹配项 需要空处理
        names.removeIf((name -> name !=null && name.equals("BB") ));
//        遍历集合
        names.forEach(str->{
            System.out.println(str);
        });
    }
```

7. 保留相同的对象

通过`retainAll()`方法获取两个集合的交集
```
    @Test
    public void test3() {
        Collection<String> names = new ArrayList<>();
        names.addAll(Arrays.asList("Mary", "Annie", "Anna", "Margaret", "Helen"));
        Collection<String> keepEm = Arrays.asList("Elsie", "Lucy", "Margaret", "Dorothy", "Mary", "Helen", "Margaret");
        names.retainAll(keepEm);
        names.forEach(System.out::println); // prints: Mary Margaret Helen
		// 可以通过方法clear（）清空集合的所有元素
    }
```




8. 错误的遍历删除方法
```
Collection<String> names = new ArrayList<>(); 
names.addAll(Arrays.asList("Mary", "Annie", "Anna", "Margaret", "Helen", "Elsie", "Lucy", "Margaret", "Dorothy")); 
for (String name : names) { 
  if ( name.contains("t") ) 
    names.remove(name); 
} // throws ConcurrentModificationException
```


> 注1：Arrays.asList（）返回的列表是不可变的列表。尝试将项目删除（或添加）到此类列表会导致  UnsupportedOperationException


***

#  三、Map 接口

`Map` 接口与 `Collection`接口看起来并没有什么关系，`Map` 接口对两个实体进行操作：key(唯一)与value，可以通过`key`从`Map`中检索数据。但是，`Map` 与 `Set` 层次结构之间有许多相似之处，是因为`Set`接口的内部实现借助了`Map`的实现。

### Map层次结构

![](/image/java/util/collection_0_map.png)
+	Map
	+	Hashtable 基于哈希表的数据结构，线程安全，无排序（由于有同步开销,性能差,使用频率较低）
	+	HashMap 基于哈希表的数据结构，线程不安全，无排序（默认实现）。
	+	LinkedHashMap 基于哈希表的数据结构，保留插入顺序
	+	SortedMap 接口；可排序的Map
		+	NavigableMap 接口：提供浏览有序的`Map`集合
			+ TreeMap 基于红黑树结构实现的，并通过密钥进行排序。

# 五、参考文章	
> [A Look at Java Collections](https://dzone.com/articles/java-collections?utm_source=dzone&utm_medium=article&utm_campaign=java-collections-cluster)
[Java Collections: The List and Collections Interfaces](https://dzone.com/articles/java-collections-part-1?utm_source=dzone&utm_medium=article&utm_campaign=java-collections-cluster)
[How To Sort Objects In Java](https://dzone.com/articles/how-to-sort-objects-in-java?utm_source=dzone&utm_medium=article&utm_campaign=java-collections-cluster)