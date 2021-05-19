---
title: java Stream 流的使用
date: 2021-05-18 15:28:20
tags: [java]
---

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

# 一、相关源码

Stream流相关源码如下：
```
public interface Stream<T> extends BaseStream<T, Stream<T>> {

    <R> Stream<R> map(Function<? super T, ? extends R> mapper);
    IntStream mapToInt(ToIntFunction<? super T> mapper);
    LongStream mapToLong(ToLongFunction<? super T> mapper);
    DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);

    <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
    IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);
    LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);
    DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);

    Stream<T> distinct();
    Stream<T> filter(Predicate<? super T> predicate);
    Stream<T> sorted();
    Stream<T> sorted(Comparator<? super T> comparator);
    Stream<T> peek(Consumer<? super T> action);
    Stream<T> limit(long maxSize);
    Stream<T> skip(long n);

    void forEach(Consumer<? super T> action);
    void forEachOrdered(Consumer<? super T> action);

    Object[] toArray();
    <A> A[] toArray(IntFunction<A[]> generator);

    T reduce(T identity, BinaryOperator<T> accumulator);
    Optional<T> reduce(BinaryOperator<T> accumulator);
    <U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
    Optional<T> min(Comparator<? super T> comparator);
    Optional<T> max(Comparator<? super T> comparator);
    long count();
	
	
    <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);
    <R, A> R collect(Collector<? super T, A, R> collector);



    boolean anyMatch(Predicate<? super T> predicate);
    boolean allMatch(Predicate<? super T> predicate);
    boolean noneMatch(Predicate<? super T> predicate);

    Optional<T> findFirst();
    Optional<T> findAny();

    // 用于生成流
    public static<T> Builder<T> builder() {
        return new Streams.StreamBuilderImpl<>();
    }
	// 返回空的顺序流
    public static<T> Stream<T> empty() {
        return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);
    }
	//	返回包含单个元素的顺序流
    public static<T> Stream<T> of(T t) {
        return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
    }
	// 返回包含多个的顺序流
    @SafeVarargs
    @SuppressWarnings("varargs") // Creating a stream from an array is safe
    public static<T> Stream<T> of(T... values) {
        return Arrays.stream(values);
    }
	// 用于创建无限流
    public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
        Objects.requireNonNull(f);
        final Iterator<T> iterator = new Iterator<T>() {
            @SuppressWarnings("unchecked")
            T t = (T) Streams.NONE;

            @Override
            public boolean hasNext() {
                return true;
            }

            @Override
            public T next() {
                return t = (t == Streams.NONE) ? seed : f.apply(t);
            }
        };
        return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
                iterator,
                Spliterator.ORDERED | Spliterator.IMMUTABLE), false);
    }
	// 返回无限顺序无序流
    public static<T> Stream<T> generate(Supplier<T> s) {
        Objects.requireNonNull(s);
        return StreamSupport.stream(
                new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
    }
	// concat可以把两个流组合为一个流
    public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) {
        Objects.requireNonNull(a);
        Objects.requireNonNull(b);

        @SuppressWarnings("unchecked")
        Spliterator<T> split = new Streams.ConcatSpliterator.OfRef<>(
                (Spliterator<T>) a.spliterator(), (Spliterator<T>) b.spliterator());
        Stream<T> stream = StreamSupport.stream(split, a.isParallel() || b.isParallel());
        return stream.onClose(Streams.composedClose(a, b));
    }

    public interface Builder<T> extends Consumer<T> {

        @Override
        void accept(T t);

        default Builder<T> add(T t) {
            accept(t);
            return this;
        }

        Stream<T> build();

    }
}

```
# 二、方法说明与示例

1. map、mapToXXX、collect

Stream提供了 mapToInt、mapToLong、mapToDouble、map等几个方法，用于将数据包装为其它类型的数据；map需要通过实现Function接口来自定义数据转换。 map 用于改变流中元素本身类型，即从元素中派生出另一种类型的操作

```
public class Test {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("2");
        list.add("22");
        list.add("222");
        list.add("12");
        list.add("23");

        list.stream().mapToInt(str-> Integer.parseInt(str)).forEach(System.out::println);
        list.stream().mapToLong(str-> Long.parseLong(str)).forEach(System.out::println);;
        list.stream().mapToDouble(str-> Double.parseDouble(str)).forEach(System.out::println);
        list.stream().map(new Function<String, Object>() {
            @Override
            public Object apply(String s) {
                return "输出："+s;
            }
        }).forEach(System.out::println);;

    }
}

// 交集
List<T> intersect = list1.stream()
                         .filter(list2::contains)
                         .collect(Collectors.toList());
// 差集(list1 - list2)
List<String> reduce1 = list1.stream().filter(item -> !list2.contains(item)).collect(toList());

// 使用并行流求并集
List<String> listAll = list1.parallelStream().collect(toList());
List<String> listAll2 = list2.parallelStream().collect(toList());
listAll.addAll(listAll2);
```


2. flatMap、flatMapToXXX

这是stream的一种中间操作，它和stream的map一样，是一种收集类型的stream中间操作，但是与map不同的是，它可以对stream流中单个元素再进行拆分（切片），相当于双重for循环。该类型的方法是二维转一维。

```
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("2");
        list.add("22");
        list.add("222");
        list.add("12");
        list.add("23");
        List<List<String>> list2 = new ArrayList<>();
        list2.add(new ArrayList<>(Arrays.asList("2","22")));
        list2.add(new ArrayList<>(Arrays.asList("222","12")));
        list2.add(new ArrayList<>(Arrays.asList("23")));

        list.stream().flatMapToInt(s-> IntStream.of(Integer.parseInt(s))).forEach(System.out::println);
        list.stream().flatMapToDouble(s-> DoubleStream.of(Double.parseDouble(s))).forEach(System.out::println);
        list.stream().flatMapToLong(s-> LongStream.of(Long.parseLong(s))).forEach(System.out::println);
//        实现集合的并、交、差操作
        list2.stream().flatMap(s->s.stream().map(word -> Integer.parseInt(word))).sorted().forEach(System.out::println);

    }
```

3. distinct、filter、sorted、peek、limit、skip（数据处理）

+	distinct列表去重；
+	filter 用于数据过滤，支持自定义比较器
+	sorted 数据排序
+	peek debug时用于输出，peek 操作接收的是一个 Consumer<T> 函数。顾名思义 peek 操作会按照 Consumer<T> 函数提供的逻辑去消费流中的每一个元素，同时有可能改变元素内部的一些属性
+	limit、skip 用于实现分页

```
   public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("2");
        list.add("2");
        list.add("22");
        list.add("222");
        list.add("222");
        list.add("12");
        list.add("23");
        list.stream().peek(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println("数据起始状态："+s);
            }
        }).mapToInt(Integer::parseInt).distinct().sorted().forEach(System.out::println);
        System.out.println("分页示例：");
        list.stream().mapToInt(Integer::parseInt).distinct().sorted().skip(2).limit(2).forEach(System.out::println);
    }
```

4. forEach、forEachOrdered、toArray
forEachOrdered 并行流下保证有序，而forEach是无序的；toArray用于转换为数组
```
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("2");
        list.add("22");
        list.add("222");
        list.add("12");
        list.add("23");
        list.stream().parallel().forEach(System.out::println);
        System.out.println("forEachOrdered 并行流下是有序的：");
        list.stream().parallel().forEachOrdered(System.out::println);
//        转换为数组
        list.stream().toArray();
    }
```

5. 聚合运算（reduce、max、min、count）

+	reduce 用于对stream中元素进行聚合求值；
+	max 用于对stream中元素求最大值
+	min 用于对stream中元素求最小值
+	count 用于获取stream元素总个数

```
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("2");
        list.add("3");
        list.add("6");
        list.add("4");
        list.add("5");
        int res = list.stream().mapToInt(Integer::parseInt).reduce((a,b)->a*b).getAsInt();

        System.out.println("max:"+list.stream().mapToInt(Integer::parseInt).max().getAsInt());
        System.out.println("min:"+list.stream().mapToInt(Integer::parseInt).min().getAsInt());
        System.out.println("count:"+list.stream().mapToInt(Integer::parseInt).count());
        System.out.println("reduce:"+res);
    }
```

6. anyMatch、allMatch、noneMatch、findFirst、findAny

boolean anyMatch(Predicate<? super T> predicate)
+	只要有一个条件满足即返回true

boolean allMatch(Predicate<? super T> predicate)
+	必须全部都满足才会返回true

boolean noneMatch(Predicate<? super T> predicate)
+	全都不满足才会返回true

例如：
```
graph.nodes().stream().filter(n -> n instanceof FMUProxy)
				.noneMatch(n -> n instanceof FMU3Proxy) ? stepperV2() : stepperV3();
```

以上的lamda表达式做了以下几件事：
+	将List转换为stream流
+	过滤非FMUProxy接口的顶点
+	如果都不是FMU3Proxy接口（v3）,noneMatch返回true（运行stepperV2()）；否则返回false（运行stepperV3()）



findFirst用于返回第一个元素；findAny在使用过滤器后（无论存在任何剩余元素），都可以（随机）返回该元素中的任何元素，尤其是在并行流操作中 。

7. 其余方法
+	builder 用于生成流
+	empty 返回空的顺序流
+	of 返回包含元素的顺序流
+	iterate 用于创建无限流
+	generate 返回无限顺序无序流

```
    public static void main(String[] args) {
        Stream.iterate(1, n->n*2).limit(10).forEach(System.out::println);

        Stream.generate(new Random()::nextInt)
                .limit(5).forEach(System.out::println);
    }
```