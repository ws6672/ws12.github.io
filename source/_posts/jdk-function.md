---
title: JDK8新特性 ———— Function接口
date: 2021-05-20 14:17:17
tags: [jdk]
---

Function接口是JDK8定义的一个函数式接口；函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。函数式接口可以被隐式转换为 lambda 表达式。


# 一、Function接口

java.util.function.Function是函数式接口，它的特点是有且只有一个抽象方法，这样的接口被@FunctionalInterface所注释，能够应用于JDK1.8开始的函数式编程。


相关源码如下：
```
package java.util.function;
import java.util.Objects;

// @FunctionalInterface 来限制函数式接口不能修改为普通的接口
@FunctionalInterface
public interface Function<T, R> {

    /**
     * 将此函数应用于给定参数。
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    /**
     * 先执行参数，然后执行调用者
     *
     * @param <V> the type of input to the {@code before} function, and to the
     *           composed function
     * @param before the function to apply before this function is applied
     * @return a composed function that first applies the {@code before}
     * function and then applies this function
     * @throws NullPointerException if before is null
     *
     * @see #andThen(Function)
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     *  先执行调用者，然后再执行参数
     *
     * @param <V> the type of output of the {@code after} function, and of the
     *           composed function
     * @param after the function to apply after this function is applied
     * @return a composed function that first applies this function and then
     * applies the {@code after} function
     * @throws NullPointerException if after is null
     *
     * @see #compose(Function)
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * 返回一个始终返回其输入参数的函数。
     *
     * @param <T> the type of the input and output objects to the function
     * @return a function that always returns its input argument
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

我们可以进行如下定义：
```
	Function<Integer,Integer> f1 = n->n+2;
```

在f1中，`n->n+2` 是Function接口中的抽象方法R apply(T t)的实现。

相关示例如下：
```
public class Test {
    public static void main(String[] args) {
        Function<Integer,Integer> f1 = n->n+2;
        Function<Integer,Integer> f2 = n->n*3;

//      identity  使用identity方法来产生一个Function对象，那么其apply方法的作用为：返回所传入apply方法中的参数
        System.out.println(Function.identity().apply(2));
//      andThen 先执行调用者，然后再执行参数 (10+2)*3
        System.out.println(f1.andThen(f2).apply(10));
//      compose 先执行参数，然后执行调用者 10*3+2
        System.out.println(f1.compose(f2).apply(10));


    }
}
```

# 二、java.util.function包

function包是JDK8新增的，该包中的接口大致分为了以下四类：

+	Function: 接收参数，并返回结果，主要方法 R apply(T t)
+	Consumer: 接收参数，无返回结果, 主要方法为 void accept(T t)
+	Supplier: 不接收参数，但返回结构，主要方法为 T get()
+	Predicate: 接收参数，返回boolean值，主要方法为 boolean test(T t)

1. Function（函数）

接收一个参数，并返回结果：

|Interface|functional method|说明|
|:--|:--|:--|
|Function<T,R>|R apply(T t)|接收参数类型为T，返回参数类型为R|
|IntFunction<R>|R apply(int value)|以下三个接口，指定了接收参数类型，返回参数类型为泛型R|
|LongFunction<R>|R apply(long value)||
|Double<R>|R apply(double value)||
|ToIntFunction<T>|int applyAsInt(T value)|以下三个接口，指定了返回参数类型，接收参数类型为泛型T|
|ToLongFunction<T>|long applyAsLong(T value)||
|ToDoubleFunction<T>|double applyAsDouble(T value)||
|IntToLongFunction|long applyAsLong(int value)|以下六个接口，既指定了接收参数类型，也指定了返回参数类型|
|IntToDoubleFunction|double applyAsLong(int value)||
|LongToIntFunction|int applyAsLong(long value)||
|LongToDoubleFunction|double applyAsLong(long value)||
|DoubleToIntFunction|int applyAsLong(double value)||
|DoubleToLongFunction|long applyAsLong(double value)||
|UnaryOperator<T>|T apply(T t)|特殊的Function，接收参数类型和返回参数类型一样|
|IntUnaryOperator|int applyAsInt(int left, int right)|以下三个接口，指定了接收参数和返回参数类型，并且都一样|
|LongUnaryOperator|long applyAsInt(long left, long right)||
|DoubleUnaryOperator|double applyAsInt(double left, double right)||

接受两个参数，并返回结果：

|interface|functional method|说明|
|:--|:--|:--|
|BiFunction<T,U,R>|R apply(T t, U u)|接收两个参数的Function|
|ToIntBiFunction<T,U>|int applyAsInt(T t, U u)|以下三个接口，指定了返回参数类型，接收参数类型分别为泛型T, U|
|ToLongBiFunction<T,U>|long applyAsLong(T t, U u)||
|ToDoubleBiFunction<T,U>|double appleyAsDouble(T t, U u)||
|BinaryOperator<T>|T apply(T t, T u)|特殊的BiFunction, 接收参数和返回参数类型一样|
|IntBinaryOperator|int applyAsInt(int left, int right)||
|LongBinaryOperator|long applyAsInt(long left, long right)||
|DoubleBinaryOperator|double applyAsInt(double left, double right)||

2. Consumer（消费者）

表示接收一个参数但不产生返回值：

|interface|functional method|说明|
|:--|:--|:--|
|Consumer<T>|void accept(T t)|接收一个泛型参数，无返回值|
|IntConsumer|void accept(int value)|以下三个类，接收一个指定类型的参数|
|LongConsumer|void accept(long value)||
|DoubleConsumer|void accept(double value)||

表示接收两个个参数但不产生返回值：

|interface|functional method|说明|
|:--|:--|:--|
|BiConsumer<T,U>|void accept(T t, U u)|接收两个泛型参数|
|ObjIntConsumer<T>|void accept(T t, int value)|以下三个类，接收一个泛型参数，一个指定类型的参数|
|ObjLongConsumer<T>|void accept(T t, long value)||
|ObjDoubleConsumer<T>|void accept(T t, double value)||

3. Supplier （提供者、创建对象）

返回一个结果，并不要求每次调用都返回一个新的或者独一的结果：

|interface|functional method|说明|
|:--|:--|:--|
|Supplier<T>|T get()|返回类型为泛型T|
|BooleanSupplier|boolean getAsBoolean()|以下三个接口，返回指定类型|
|IntSupplier|int getAsInt()||
|LongSupplier|long getAsLong()||
|DoubleSupplier|double getAsDouble()||

4. Predicate（断言）

根据接收参数进行断言，返回boolean类型：

|interface|functional method|说明|
|:--|:--|:--|
|Predicate<T>|boolean test(T t)|接收一个泛型参数|
|IntPredicate|boolean test(int value)|以下三个接口，接收指定类型的参数|
|LongPredicate|boolean test(long value)||
|DoublePredicate|boolean test(double value)||
|BiPredicate<T,U>|boolean test(T t, U u)|接收两个泛型参数，分别为T，U|


相关示例如下：
```
public class Test {
    public static void main(String[] args) {
//        创建对象
        Supplier<Test> supplier = Test::new;
        System.out.println(supplier.get());
        System.out.println(supplier.get());

        BooleanSupplier booleanSupplier = ()->7>9;
        System.out.println(booleanSupplier.getAsBoolean());

        IntSupplier intSupplier = ()-> 6*76;
        System.out.println(intSupplier.getAsInt());

        LongSupplier longSupplier = ()-> 33*762;
        System.out.println(longSupplier.getAsLong());

//        断言
        Predicate<String> predicate = (s)-> s.matches("^[a-z][\\w]*");
        System.out.println(predicate.test("abc"));
        System.out.println(predicate.test("1abc"));
        
    }
}
```

# 三、参考资料

> [java 8 函数式接口](https://segmentfault.com/a/1190000016596774)