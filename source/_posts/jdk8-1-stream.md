---
title: jdk8（一）streams
date: 2020-03-26 15:34:23
tags: [java]
---

### 介绍
Streams 与 Java I/O流没有任何关系，I/O流是用于文件的包装器，而Streams是用于数据源的包装器。

> `Streams`不存储数据，从这个意义上说，它不是数据结构。它还永远不会修改基础数据源。
`java.util.stream`-支持对元素流进行功能样式的操作，例如对集合进行map-reduce转换




Stream.of()方法
```
private static Employee[] arrayOfEmps = {
    new Employee(1, "Jeff Bezos", 100000.0), 
    new Employee(2, "Bill Gates", 200000.0), 
    new Employee(3, "Mark Zuckerberg", 300000.0)
};
Stream.of(arrayOfEmps);
Stream.of(arrayOfEmps[0], arrayOfEmps[1], arrayOfEmps[2]);
```


构造器
```
Stream.Builder<Employee> empStreamBuilder = Stream.builder();
empStreamBuilder.accept(arrayOfEmps[0]);
empStreamBuilder.accept(arrayOfEmps[1]);
empStreamBuilder.accept(arrayOfEmps[2]);
Stream<Employee> empStream = empStreamBuilder.build();
```



Java 8向Collection接口添加了新的stream（）方法
```
private static List<Employee> empList = Arrays.asList(arrayOfEmps);
empList.stream();
```


流操作分为中间方法和终端方法。流管道包括一个流源，后跟零个或多个中间操作，以及一个终端操作。
中间方法
+	filter()
+	peek()
有一些中间方法被称之为短路方法，
skip()
limit()
min()
max()
终端方法（例如forEach()）会将流标记为已使用，此后将无法再使用它。
+	forEach()
+	collect()
+	count()


惰性计算
仅在启动终端操作时才执行对源数据的计算，并且仅在需要时才使用源元素。所有中间操作都是惰性的，因此直到实际需要处理结果时才执行它们。

基于比较的流操作

短路操作不适用于排序操作
