---
title: jdk8(一) Lambda表达式
date: 2020-03-20 15:35:07
tags: [java]
---


### 什么是 Lambda表达式？
它是匿名函数的简洁表示，可以传递；不需要进行定义，也不需要与特定类关联。Lambda表达式是一种简洁的表示形式，因为不需要像编写匿名类那样编写样板代码。

将方法转换为`Lambda表达式`:
+	去除方法名
+	去除返回类型定义
+	去除修饰符
+	添加箭头符号（->）

### Lambda表达式 基础
```
// 类型声明
MathOperation addition = (int a, int b) -> a + b;

// 不用类型声明
MathOperation subtraction = (a, b) -> a - b;

// 大括号中的返回语句
MathOperation multiplication = (int a, int b) -> { return a * b; };

// 没有大括号及返回语句
MathOperation division = (int a, int b) -> a / b;



```
### Lambda表达式 实例
```
    @org.junit.Test
    public void testStreams() {
        String[] strs = {"AA","BB","CC",null};
        Stream<String> stringStream = Stream.of(strs);
//        forEach()是终端操作，这意味着在执行该操作之后，流流被视为已消耗，并且不再可用
//        如果在forEach后再循环流，会报错 java.lang.IllegalStateException: stream has already been operated upon or closed

        System.out.println("Streams 通过 forEach（）输出：");
        stringStream.forEach(e->System.out.println(e));

//        filter()方法，可以用来过滤元素
//        map(String::toLowerCase) 转换流类型，通过调用String类的toLowerCase方法
//        collect() 对Stream实例中保存的数据元素执行可变的集合 （将元素重新打包到某些数据结构并应用一些其他逻辑，将它们串联等）
        List stringList = Stream.of(strs).filter(e-> e != null).map(String::toLowerCase).collect(Collectors.toList());
        System.out.println("Streams 过滤null值，转换为List,通过 forEach()输出：");
        stringList.forEach(e->System.out.println(e));


//        peek() 方法是中间操作，用于调试代码；需要加终端操作方法，不然不会有输出
        System.out.println("Streams 转换为数组,通过增强for循环输出：");
        String[] arrays = Stream.of(strs).peek(e->System.out.println("节点状态："+e)).filter(e -> e != null).toArray(String[]::new);
        for (String str :arrays) {
            System.out.println(str);
        }
    }
```



> 注1：lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误；可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）






















