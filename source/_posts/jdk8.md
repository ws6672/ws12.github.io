---
title: JDK8 语言新特性
date: 2021-03-29 12:41:58
tags: [java]
---

# 接口新增默认方法与静态方法

```
/**
 * @author zws
 * @date 2021/3/24 19:01
 * @Description 接口新增默认方法与静态方法
 */

public interface Interface1 {
    void method1();
    default void test() {
        System.out.println("接口的默认方法");
    }
    
    static void test2() {
        System.out.println("接口的静态方法");
    }
}

```

# 使用@FunctionalInterface限制函数接口的转换

+	函数式接口仅仅只有一个方法(非默认或静态方法)，用于显示转换成ladbma表达式。
+	java.lang.Runnable接口、java.util.concurrent.Callable接口是两个最典型的函数式接口。
+	如果一个函数式接口添加一个普通方法，就变成了非函数式接口（一般定义的接口）。
+	Jdk8 规范里添加了注解@FunctionalInterface来限制函数式接口不能修改为普通的接口.

> 注：函数形接口 、供给形接口、消费型接口、判断型接口

```
/**
 * @author zws
 * @date 2021/3/24 19:03
 * @Description 使用@FunctionalInterface限制接口只能有一个方法
 */

@FunctionalInterface
public interface Functional1 {
    void test();
}
```


# Lambda表达式 与 stream 流

Lambda表达式（基于函数的匿名表达式）,相关语法如下：
`( object str,....)[参数列表]   ->[箭头符号]     代码块或表达式`

Stream是元素的集合，可以支持顺序和并行的对原Stream进行汇聚的操作。创建stream：
+	通过Stream接口的静态工厂方法
+	通过Collection接口的默认方法 stream()，把一个Collection对象转换成Stream

stream相关的几个方法：
+	转换
	+	distinct：去重
	+	filter：对于Stream中包含的元素使用给定的过滤函数进行过滤操作
	+	map：对于Stream中包含的元素使用给定的转换函数进行转换操作，支持类型转换（mapToInt，mapToLong和mapToDouble）
	+	peek：生成一个包含原Stream的所有元素的新Stream，同时会提供一个消费函数（Consumer实例），新Stream每个元素被消费的时候都会执行给定的消费函数
	+	limit: 对一个Stream进行截断操作，获取其前N个元素，如果原Stream中包含的元素个数小于N，那就获取其所有的元素
	+	skip: 返回一个丢弃原Stream的前N个元素后剩下元素组成的新Stream，如果原Stream中包含的元素个数小于N，那么返回空Stream
	+	sorted：排序
	+	forEach：遍历
+	汇聚（Reduce）Stream
	+	count：统计个数
	+	concat：组合多个流
	+	collect：流中多个元素合并为一个元素
相关实例如下：
```
public class LambdaTest {
    public static void main (String[] args) {
//        1. Lambda 表达式实现Runnable
        new Thread(()->System.out.println("1. Lambda 表达式实现Runnable，相关的内部类都可以这么写")).start();
//        2. Lambda 表达式迭代list

        System.out.println("2. Lambda 表达式迭代list");
        List list = Arrays.asList("a","b","c","d","f");
        list.forEach(n->System.out.println(n));


//        3. Lambda 表达式map
        System.out.println("3. Lambda 表达式map");
        List<Integer> list2 = Arrays.asList(11,52,7,9,40);
        list2.stream().map((num)-> num*2).forEach(System.out::println);

//        4. Lambda 表达式过滤数据
        System.out.println("4. Lambda 表达式过滤数据 输出三个字节以下的数据");
        List<String> list3 = Arrays.asList("ad","bsdfd","cdf","ddfd","fs");
        list3.stream().filter(str->str.length()<3).forEach(System.out::println);

//        5. Lambda 表达式连接字符
        System.out.println("5. Lambda 表达式连接字符");
        List<String> list4 = Arrays.asList("a","b","c","d","e");
        String res = list4.stream().map((cr)->cr.toUpperCase()).collect(Collectors.joining("-"));
        System.out.println(res);
//        6. Lambda 表达式计算集合元素
        System.out.println("计算集合元素的最大值、最小值、总和以及平均值");
        List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
        IntSummaryStatistics stats = primes.stream().mapToInt((x) -> x).summaryStatistics();
        System.out.println("Highest prime number in List : " + stats.getMax());
        System.out.println("Lowest prime number in List : " + stats.getMin());
        System.out.println("Sum of all prime numbers : " + stats.getSum());
        System.out.println("Average of all prime numbers : " + stats.getAverage());

//        7. Lambda 表达式去重
        System.out.println("7. Lambda 表达式去重");
        List<Integer> list5 = Arrays.asList(2, 2, 5, null, 11, 11, 17, 19, 17, 2);
//        filter(x->x!=null 避免为null的情况
//        distinct() 去重
        list5.stream().filter(x->x!=null).distinct().forEach(System.out::println);

//        8. Lambda 表达式排序
        System.out.println("8. Lambda 表达式排序");
        List<Integer> list6 = Arrays.asList(29, 2, 5, null, 11, 11, 17, 19, 17, 2);
        list6.stream().filter(x->x!=null).sorted((a,b)->a-b).distinct().forEach(System.out::println);

    }
}
```

# 新增方法引用

方法引用是只需要使用方法的名字，而具体调用交给函数式接口，需要和Lambda表达式配合使用。例如：
```
list.forEach(str -> System.out.print(str));
list.forEach(System.out::print);//等价
```

+	分类
	+	构造器引用
		+	格式：`Class::new`，调用默认构造器
	+	静态方法引用
		+	格式：`Class::static_method`
	+	类(任意对象)的方法引用（在流中会调用每个类）
		+	格式：`Class::method`，方法不能带参数
	+	实例对象的方法引用
		+	格式：`instance::method`

相关实例如下：
```
/**
 * @author zws
 * @date 2021/3/24 20:26
 * @Description 方法引用
 */
public class MethodUseTest {
    public MethodUseTest() {
        System.out.println("默认构造器");
    }

    public static String test() {
        System.out.println("静态方法");
        return "test";
    }
    void instance() {
        System.out.println("实例方法");
    }
    public static void main (String[] args) {
        List<MethodUseTest> list = new ArrayList<>();
//        1.构造器引用 需要使用 Supplier(创建对象的工厂)
        Supplier<MethodUseTest> mut = MethodUseTest::new;
        Supplier<MethodUseTest> mut2 = MethodUseTest::new;
//        2.静态方法引用。如果函数式接口的实现恰好是通过调用一个静态方法来实现，那么就可以使用静态方法引用
        Supplier<String> test = MethodUseTest::test;

        list.add(mut.get());
        list.add(mut2.get());
//        3.普通方法引用，方法不能带参数。
        list.forEach(MethodUseTest::instance);
//        4.实例对象的方法引用 需要以流元素作为参数
        MethodUseTest mut3 = new MethodUseTest();
        list.forEach(mut3::instance);
    }

    private void instance(MethodUseTest methodUseTest) {
        methodUseTest.instance();
    }
}

```


# 其它

1. 重复注解
Java 5引入了注解机制，这一特性就变得非常流行并且广为使用，但是相同的注解在同一地方只能使用一次。Java 8引入了重复注解机制，使相同的注解可以在同一地方声明多次，重复注解机制本身必须用@Repeatable注解。

2. 扩展注解的支持
JDK8 扩展了注解的上下文。现在几乎可以为任何东西添加注解：局部变量、泛型类、父类与接口的实现，就连方法的异常也能添加注解。
