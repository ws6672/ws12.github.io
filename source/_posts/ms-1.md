---
title: 面试（一）Java SE
date: 2020-05-30 12:10:27
tags: ms
---

### 一、JVM相关

***JVM如何运行代码***
+	`编译` .java `---编译器--->` .class（字节码 
+	`装载` 
	+	双亲委派机制，类加载器获取到类加载请求，会先委派给父类加载器处理，父类加载器无法解决，再尝试自己处理
+	`执行` .class（字节码）`---JVM--->` 机器码 
	+	解释执行、编译执行并存，如果存在热点方法（多次调用），会先进行编译执行，避免多次解释
	+	解释执行 每执行一句字节码，就翻译成机器码
	+	编译执行 所有字节码都先翻译成机器码，再调用

***JVM的内存如何划分***
![JVM体系结构](/image/jvm/JVM.png)


***类加载器有哪一些？***
+	BootStrap ClassLoader（启动类加载器）
	+	最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等
+	extension ClassLoader（拓展类加载器）
	+	负责加载ext文件夹（jre/lib）内的类。。还可以加载-D java.ext.dirs选项指定的目录。
+	Application ClassLoader（应用类加载器）
	+	负责加载应用程序级别的类路径（当前应用）
	
	
***堆空间如何划分？***

`JVM Heap`分为两个不同的分代：新生代以及旧生代：

![JVM Heap](/image/jvm/jvm-heap.png)


+	堆内存分为 年轻代（Young Generation）、老年代（Old Generation）；方法区就是一个永久代（Permanent Generation），但JDK8-废弃永久代(PermGen)迎来元空间(Metaspace)。垃圾收集器还为年轻代和老年代提供了虚拟空间，垃圾收集器使用它们来调整其他区域的大小，主要是为了满足不同的GC目标。
	+	年轻代（`标记-清理`）又分为`Eden`和`Survivor`区。`Eden`区占大容量，`Survivor`两个区占小容量，默认比例是8:1:1。 
		+	`Eden`区：新创建对象存放位置。
		+	`Survivor`区：堆回收时，存活对象的存放位置。由 `FromSpace` 和`ToSpace`组成。
			+	步骤一：第一次Minor GC后，幸存的对象从`Eden`区复制到`Survivor0`(`From`区)；`Eden`区清空。
			+	步骤二：第二次Minor GC后，`Eden`和`Survivor0`中的存活对象又会被复制送入第二块幸存空间，即`To`区；`Eden`和`Survivor0`清空。
			+	然后下一轮S0与S1交换角色，如此循环往复（重复步骤二）。*如果对象的复制次数达到16次，该对象就会被送到老年代中*。如此操作，永远有一块`Survivor`区是空的，而另外一块`Survivor`区是无碎片的。
	
	+	年老代（`复制算法`）主要存放JVM认为生命周期比较长的对象（经过几次的Young Gen的垃圾回收后仍然存在），采用压缩的方式来避免内存碎片
	+	元空间(Metaspace)：元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。
	
![jvm-heap-handle](/image/jvm/jvm-heap-handle.png)	

> 标记-清理算法：标记可达对象，清理未标记对象
复制算法:标记可达对象，复制带标记的对象到其它区，清空本区。避免碎片化
Minor GC：新生代GC，所有的Minor GC都会触发全世界的暂停（stop-the-world），停止应用程序的线程，不过这个过程非常短暂。
Major GC/Full GC：老年代GC，指发生在老年代的GC。

### 二、JAVA SE 基础

***面向对象都有哪些特性***

+	封装：将操作与数据绑定起来，只能通过方法访问
+	继承：对父类的拓展
+	多态：允许不同的子类对同一消息作出不同的响应
	+	重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载
	+	重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。重载对返回类型没有特殊的要求。
+	抽象：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面。
 
 
***权限访问修饰符***
+	public：所有类都可以访问
+	protected：同包及其子类可以调用
+	不写（默认）：同包可以调用
+	private：只能在当前类中调用
 
 
***new 一个对象的过程和 clone 一个对象的过程区别***
new的流程：基于类分配对应的内存、调用构造函数、返回地址
clone：基于原对象分配对应内存、使用原对象中各个域填充对应域、返回地址
 
***深拷贝和浅拷贝***
 
对象中存在非基本类型，而浅拷贝只是复制了属性的引用，而不是属性的数据，是父类的默认实现；
深拷贝则需要重写clone方法


***Object的几个方法***

clone、equal、hashCode 对象操作
finalize 垃圾回收
wait(3)、notify、notifyAll 线程处理
registerNatives 本地方法调用
toString()

***equal与hashCode的关联***

`equals`方法在比较大量对象的时候，效率太低。所以，在Java源码中会先调用`hashCode`方法，相等则再调用`equals`方法。所以，如果自定义类需要重写Object类的`hashCode`方法。例如：

```
class User {
    private Integer age;
    private String name;

    @Override
    public int hashCode() {
        return age*name.hashCode();
    }
}
```


***两个对象值相同 (x.equals(y) == true) ，但却可有不同的 hashCode，这句话对不对***

不对，`equals`为`TRUE`，则`hashCode`必定相等。

equals 方法必须满足：
+	自反性（x.equals(x)必须返回 true）
+	对称性（x.equals(y)返回 true 时，y.equals(x)，也必须返回 true）
+	传递性（x.equals(y)和 y.equals(z)都返回 true 时，x.equals(z)也必须返回 true


***当一个对象被当作参数传递到一个方法后，此方法可改变这个对象的属性，并可返回变化后的结果，那么这里到底是值传递还是引用传递?***

值传递。Java 语言的方法调用只支持参数的值传递。当一个对象实例作为一个参数被传递到方法中时，参数的值就是对该对象的引用.

***重写与重载的区别***

重载是一个编译期的概念，遵循编译期绑定，所以不是多态的；重写是一个运行期间的概念，遵循运行期绑定，所以是多态的。
+	重载
	+	方法在同一个类中，方法名相同、参数列表不同、返回值无限制
	+	可以抛出不同的异常
	+	可以有不同修饰符
	
+	重写
	+	方法在子父类中，是子类对父类方法的重定义，方法名相同、参数列表相同、返回值相同
	+	访问修饰符的限制一定要大于被重写方法的访问修饰符（public>protected>default>private）
	+	构造方法不能被重写，声明为 final 的方法不能被重写，声明为 static 的方法不能被重写
	+	重写的方法不能抛出新的强制性异常，或者比被重写方法声明的更广泛的强制性异常（例如父类抛出IOException，子类只能抛出它的子类，不能抛出Exception），反之则可以

> 注：如果父类方法为权限为 private，那么子类的同名同参方法不构成重写，也不构成多态关系

***为什么函数不能根据返回类型来区分重载***

那么编译器就无法确定你调用的是哪个函数

***char 型变量中能不能存储一个中文汉字，为什么***

char 类型可以存储一个中文汉字，因为 Java 中使用的编码是 Unicode（不选择任何特定的编码，直接
使用字符在字符集中的编号，这是统一的唯一方法），一个 char 类型占 2 个字节（16 比特），所以放一个中文是没问题的。


***抽象类(abstract class)和接口(interface)有什么异同？***

抽象类：
1.抽象类中可以定义构造器
2.可以有抽象方法和具体方法
3.接口中的成员全都是 public 的
4.抽象类中可以定义成员变量
5.有抽象方法的类必须被声明为抽象类，而抽象类未必要有抽象方法
6.抽象类中可以包含静态方法
7.一个类只能继承一个抽象类，并且需要实现它的抽象方法

接口：
1.接口可以有静态方法，默认方法（JDK8）
2.接口中不能定义构造器
3.接口中定义的成员变量实际上都是常量
4.一个类可以实现多个接口

***抽象的(abstract)方法是否可同时是静态的(static), 是否可同时是本地方法(native)，是否可同时被 synchronized***

都不能。抽象方法需要子类重写，而静态的方法是无法被重写的，因此二者是矛盾的。本地方法是由
本地代码（如 C 代码）实现的方法，而抽象方法是没有实现的，也是矛盾的。synchronized 和方法的实现细节有关，
抽象方法不涉及实现细节，因此也是相互矛盾的



***静态变量和实例变量的区别***

静态变量属于类，在创建第一个对象时就被初始化，被多个对象共享
实例变量属性实际对象，在对象创建的时候初始化，

***==和 equals 的区别***

`==`比较引用；而`equals`用于比较对象的内容是否相同

***String s = "Hello";s = s + " world!";这两行代码执行后，原始的 String 对象中的内容到底变了没有***

String是`final`定义的类，是不可变类，所以原始对象的内容没有变化，而是指向了一个新的 	字符串常量。


***Java 中实现多态的机制是什么***

1. 方法定义的引用参数为父类或接口
2. 程序调用的方法在`运行期才动态绑定`, 引用参数指向的实例对象才确定
3. 引用变量所指向的具体实例对象的方法，也就是内存里正在运行的那个对象的方法，而不是引用变量的类型中定义的方法


> a. Java没有 goto 语句，只是保留字
b. & 左边执行失败会执行右边；&&则是左边执行失败不会执行右边。
c. String 类是 final 类，不可以被继承


### 三、异常处理

***Java 中异常分为哪些种类***
按照异常需要处理的时机分为编译时异常（也叫强制性异常）也叫 CheckedException 和运行时异常（也叫非强制性异常）也叫 RuntimeException。只有 java 语言提供了 Checked 异常，Java 认为 Checked异常都是可以被处理的异常，所以 Java 程序必须显式处理 Checked 异常。

对 Checked 异常处理方法：
1 当前方法知道如何处理该异常，则用 try...catch 块来处理该异常。
2 当前方法不知道如何处理，则在定义该方法是声明抛出该异常。

***常见的运行时异常***

NullPointerException、SQLException、IndexOutOfBoundsException、ClassNotFoundException、ArrayIndexOutOfBoundsException


1）java.lang.NullPointerException 空指针异常；出现原因：调用了未经初始化的对象或者是不存在的对象。 

2）java.lang.ClassNotFoundException 指定的类找不到；出现原因：类的名称和路径加载错误；通常都是程序
试图通过字符串来加载某个类时可能引发异常。

3）java.lang.NumberFormatException 字符串转换为数字异常；出现原因：字符型数据中包含非数字型字符。 

4）java.lang.IndexOutOfBoundsException 数组角标越界异常，常见于操作数组对象时发生。

5）java.lang.IllegalArgumentException 方法传递参数错误。 

6）java.lang.ClassCastException 数据类型转换异常

7）java.lang.NoClassDefFoundException 未找到类定义错误。

8）SQLException SQL 异常，常见于操作数据库时的 SQL 语句错误。

9）java.lang.InstantiationException 实例化异常。

10）java.lang.NoSuchMethodException 方法不存在异常。


***error 和 exception 的区别？***
Error 类和 Exception 类的父类都是 Throwable 类，区别如下：
+	Error 类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢出等
+	Exception 类表示程序可以处理的异常，可以捕获且可能恢复


***throw 和 throws 的区别？***
throw 用于方法体内抛出异常，是一个抛异常的动作
trows 用于方法声明之后，定义了抛出了何种异常类型

***final、finally、finalize 的区别***
final：用于声明属性，方法和类，分别表示属性不可变，方法不可覆盖，被其修饰的类不可继承
finally：是try-catch之后，即异常处理语句结构的一部分，表示总是执行
finalize：Object 类的一个方法，在垃圾回收器执行的时候会调用被回收对象的此方法

### 四、SE 常用 API

***Math.round(11.5)等于多少？Math.round(- 11.5) 又等于多少?***

Math.round(11.5)的返回值是 12，Math.round(-11.5)的返回值是-11。四舍五入的原理是在参数上加 0.5
然后进行取整。

***switch 是否能作用在 byte 上，是否能作用在 long 上，是否能作用在 String上?***

+	Java5 以前 switch(expr)中，expr 只能是 byte、short、char、int。
+	从 Java 5 开始，Java 中引入了枚举类型，expr 也可以是 enum 类型
+	从 `jdk7` 开始，expr 还可以是字符串（String），但是长整型（long）在目前所有的版本中都是不可以的。

***String 、StringBuilder 、StringBuffer 的区别***
String是不可变类
StringBuffer是线程安全的；StringBuilder是非线程安全的，适合用于单线程下拼接字符串

***什么情况下用“+”运算符进行字符串连接比调用 StringBuffer/StringBuilder 对象的 append 方法连接字符串性能更好？***

+	无论使用何种方式进行字符串连接，实际上都使用的是 StringBuilder。
+	当需要进行频繁修改字符串的操作时，先建立StringBuilder类对象进行操作
+	在使用 StringBuilder 时要注意，尽量不要"+"和 StringBuilder 混着用，否则会创建更多的StringBuilder 对象。


***String结果输出判断***

```
class StringEqualTest {
	public static void main(String[] args) {
		String s1 = "Programming";
		String s2 = new String("Programming");
		String s3 = "Program";
		String s4 = "ming";
		String s5 = "Program" + "ming";
		String s6 = s3 + s4;
		System.out.println(s1 == s2); //false
		System.out.println(s1 == s5); //true
		System.out.println(s1 == s6); //false
	 }
}
```
***如何取得年月日、小时分钟秒？***
```
Calendar cal = Calendar.getInstance();
System.out.println(cal.get(Calendar.YEAR));
System.out.println(cal.get(Calendar.MONTH)); // 0 - 11
System.out.println(cal.get(Calendar.DATE));
System.out.println(cal.get(Calendar.HOUR_OF_DAY));
System.out.println(cal.get(Calendar.MINUTE));
System.out.println(cal.get(Calendar.SECOND));
```

***如何格式化日期***

```
SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
Date date = new Date();
String datestr = sdf.format(date1);
```

***Java8 的日期特性***
+	不变性：新的日期/时间 API 中，所有的类都是不可变的，这对多线程环境有好处。
+	关注点分离：新的 API 将人可读的日期时间和机器时间（unix timestamp）明确分离，它为日期（Date）、时间（Time）、日期时间（DateTime）、时间戳（unix timestamp）以及时区定义了不同的类。
+	清晰：在所有的类中，方法都被明确定义用以完成相同的行为。举个例子，要拿到当前实例我们可以使用 now()方法，在所有的类中都定义了 format()和 parse()方法，而不是像以前那样专门有一个独立的类。为了更好的处理问题，所有的类都使用了工厂模式和策略模式，一旦你使用了其中某个类的方法，与其他类协同工作并不困难。
+	实用操作：所有新的日期/时间 API 类都实现了一系列方法用以完成通用的任务，如：加、减、格式化、解析、从日期/时间中提取单独部分，等等。
+	可扩展性：新的日期/时间 API 是工作在 ISO-8601 日历系统上的，但我们也可以将其应用在非 ISO 的日历上

***时间函数常用api***

`java.time.LocalDate`
LocalDate 是一个不可变的类，它表示默认格式(yyyy-MM-dd)的日期，我们可以使用 now()方法得到当前时间，也可以提供输入年份、月份和日期的输入参数来创建一个 LocalDate 实例。

`java.time.LocalTime`
LocalTime 是一个不可变的类，它的实例代表一个符合人类可读格式的时间，默认格式是 hh:mm:ss.zzz。像LocalDate 一样，该类也提供了时区支持，同时也可以传入小时、分钟和秒等输入参数创建实例

`java.time.LocalDateTime`
LocalDateTime 是一个不可变的日期-时间对象，它表示一组日期-时间，默认格式是 yyyy-MM-dd-HH-mm-ss.zzz。它提供了一个工厂方法，接收 LocalDate 和 LocalTime 输入参数，创建 LocalDateTime 实例。

`java.time.Instant`
Instant 类是用在机器可读的时间格式上的，它以 Unix 时间戳的形式存储日期时间

几个时间api输出的默认时间格式如下：

![时间api](/image/ms/java.time.png)

***总结***
Java.time ：流畅的 API、实例不可变、线程安全
java.util.Calendar 以及 Date：不流畅的 API、实例可变、非线程安全
 
 
### 五、java的数据类型

***Java 的基本数据类型都有哪些各占几个字节***

![数据类型](/image/ms/sjlx.png)

***String 是基本数据类型吗***

不是，引用类型，底层由`char`数组实现

***short s1 = 1; s1 = s1 + 1; 有错吗?short s1 = 1; s1 += 1 有错吗***

前者不正确，后者正确。
对于问题一：由于 1 是 int 类型，因此 s1+1 运算结果也是 int 型，需要强制转换类型才能赋值给 `short` 型。
对于问题二：可以正确编译，因为 s1+= 1;相当于 s1 = (short)(s1 + 1);其中有隐含的强制类型转换。

***int 和 和 Integer 有什么区别？？***

Java 是一个近乎纯洁的面向对象编程语言，但是为了编程的方便还是引入了基本数据类型，为了能够将这些基本数据类型当成对象操作，Java 为每一个基本数据类型都引入了对应的包装类型（wrapper class），int 的包装类就是Integer，从 Java 5 开始引入了自动装箱/拆箱机制，使得二者可以相互转换。


Java 为每个原始类型提供了包装类型：
- 原始类型: boolean，char，byte，short，int，long，float，double
- 包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double
![包装类型](/image/ms/wrapper.png)

***下面 Integer 类型的数值比较输出的结果？***

![包装类型](/image/ms/wrapper2.png)


`Integer`  是包装类型，对常用的数 `-128~127`都进行了初始化，如果需要这样的数，就不会进行初始化，而是从常量池中获取。所以，第一个比较是 true;而第二个比较是false。


```
    public static void main(String[] args) {
        Integer i = 10;
        Integer i1 = 10; //（-128~127）从常量池获取
        Integer i2 = new Integer(10);
        Integer i3 = new Integer(10);//重新new对象，不从常量池获取
        System.out.println((i == i1)); //true
        System.out.println(i2== i3); // false
    }
```



***String 类常用方法***

![String 类](/image/ms/string.png)

***String、StringBuffer、StringBuilder 的区别***

String是不可变类，而其它两个则不是；
String：对象定义后不可变，线程安全；StringBuffer加入了同步锁，线程安全；StringBuilder是非线程安全的。


 
***字符串如何转基本数据类型？***

调用基本数据类型对应的包装类中的方法 parseXXX(String)或 valueOf(String)即可返回相应基本类型。
 
***基本数据类型如何转字符串？***

一种方法是将基本数据类型与空字符串（“”）连接（+）即可获得其所对应的字符串；另一种方法是调用 String 类中的 valueOf()方法返回相应字符串。

***Java 中有几种类型的流***

按照流的方向：
+	输入流（inputStream）
+	输出流（outputStream）

按照实现功能分：
+	节点流（可以从或向一个特定的地方（节点）读写数据。如 FileReader）
+	处理流（是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写

按照处理数据的单位：
+	字节流
+	字符流

![IO](/image/ms/io.png)


***字节流如何转为字符流***

+	字节输入流转字符输入流通过 InputStreamReader 实现，该类的构造函数可以传入 InputStream 对象。
+	字节输出流转字符输出流通过 OutputStreamWriter 实现，该类的构造函数可以传入 OutputStream 对象

***如何将一个 java 对象序列化到文件里***

在 java 中能够被序列化的类必须先实现 Serializable 接口，该接口没有任何抽象方法只是起到一个标记作用。

```
//对象输出流
ObjectOutputStream objectOutputStream = 
new ObjectOutputStream(new FileOutputStream(new File("D://obj")));
objectOutputStream.writeObject(new User("zhangsan", 100));
objectOutputStream.close();

//对象输入流
ObjectInputStream objectInputStream = 
new ObjectInputStream(new FileInputStream(new File("D://obj")));
User user = (User)objectInputStream.readObject();
System.out.println(user);
objectInputStream.close();

```


***字节流和字符流的区别***

字节流：
+	处理所有类型的数据
+	每次读取一个字节
+	字节流主要是操作 byte 类型数据，以 byte 数组为准，主要操作类就是 OutputStream、InputStream

字符流：
+	处理文本数据
+	读取一个或多个字节后，查询编码表，有对应的则输出该字符。
+	java 提供了 Reader、Writer 两个专门操作字符流的类

***如何实现对象克隆***

+	实现 Cloneable 接口并重写 Object 类中的 clone()方法
+	实现 Serializable 接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆

> 基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对
象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常，这种是方案明显优于使用 Object 类的 clone 
方法克隆对象。让问题在编译的时候暴露出来总是好过把问题留到运行时。

***什么是 java 序列化，如何实现 java 序列化***
序列号是处理对象流的一种机制，而对象流就是将对象的内容流化，然后可以通过网络传输或者保存在文件中。

实现序列化的步骤如下：
1. 序列化的类需要实现Serializable接口，该接口用于标志该类为可序列号的。
2. 通过输出流构造`ObjectOutputStream(对象流)`对象
3. 使用 ObjectOutputStream 对象的 writeObject(Object obj)方法就可以将参数为 obj 的对象写出(即保存其状态)

反序列化：
1. 通过输入流构造`ObjectInputStream(对象流)`对象
2. 调用该对象的readObject方法读取对象，然后强转为指定对象。


### 六、Java集合

***集合的安全性问题***
ArrayList、HashSet、HashMap不是线程安全的，但是我们可以用  Stack、Vector 和 HashTable替代，因为它们是线程安全的。这两个类实现线程安全的方法不过是加上了`synchronized`关键字而已。

但是，这样也不是绝对的线程安全。同步容器也有线程安全问题，对于原子操作来说是线程安全的。但是，对于带更新的复合操作，就是非线程安全的。

如以下代码，在多线程的环境下会有安全问题。元素再被当前线程删除前，就被其它线程删除了。
```
public void deleteLast() {
    synchronized (v) {
        int index = v.size() - 1;
        v.remove(index);
    }
}
```
可以修改为：
```
for (int i = 0; i < v.size(); i++) {
    v.remove(i);
}
```



##### 1. ArrayList

***ArrayList 内部用什么实现的？***

ArrayList底层是一个初始大小为10的Object数组。当数组已经满了之后，会动态增长。


动态增长数组，每次增长二分之一：
```
private void grow(int minCapacity) {
	// overflow-conscious code
	int oldCapacity = elementData.length;
	int newCapacity = oldCapacity + (oldCapacity >> 1);
	if (newCapacity - minCapacity < 0)
		newCapacity = minCapacity;
	if (newCapacity - MAX_ARRAY_SIZE > 0)
		newCapacity = hugeCapacity(minCapacity);
	// minCapacity is usually close to size, so this is a win:
	elementData = Arrays.copyOf(elementData, newCapacity);
}
```


***ArrayList 的remove方法***

如下，删除元素后面的元素依序向前覆盖，最后一个元素重置为`null`.
```
public E remove(int index) {
	rangeCheck(index);

	modCount++;
	E oldValue = elementData(index);

	int numMoved = size - index - 1;
	if (numMoved > 0)
		System.arraycopy(elementData, index+1, elementData, index,
						 numMoved);
	elementData[--size] = null; // clear to let GC do its work

	return oldValue;
}```



##### 2. HashMap

***HashMap 排序题***

已知一个 HashMap<Integer，User>集合， User 有 name（String）和 age（int）属性。请写一个方法实现对HashMap 的排序功能，该方法接收 HashMap<Integer，User>为形参，返回类型为 HashMap<Integer，User>，要求对 HashMap 中的 User 的 age 倒序进行排序。排序时 key=value 键值对不得拆散。

```
/**
 * @date 2020/5/31 12:58
 * @Description HashMap排序练习
 */
class User {
    private Integer age;
    private String name;

    public User() {
    }

    @Override
    public String toString() {
        return "User{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

    public User(Integer age, String name) {
        this.age = age;
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
public class HashMapTest {
    public static HashMap<Integer,User> sort(HashMap<Integer,User> map) {
        Set<Map.Entry<Integer, User>> set = map.entrySet();
        List<Map.Entry<Integer, User>> list = new ArrayList<>(set);
        Collections.sort(list, new Comparator<Map.Entry<Integer, User>>() {
            @Override
            public int compare(Map.Entry<Integer, User> o1, Map.Entry<Integer, User> o2) {
                return o1.getValue().getAge()-o2.getValue().getAge();
            }
        });
        LinkedHashMap<Integer,User> linkedHashMap = new LinkedHashMap<>();
        for (Map.Entry<Integer, User> entry:list) {
            linkedHashMap.put(entry.getKey(), entry.getValue());
        }
        return linkedHashMap;
    }
    public static void main (String[] args) {
        HashMap<Integer,User> hashMap = new HashMap<>();
        hashMap.put(1,new User(6, "aa"));
        hashMap.put(2,new User(3, "bb"));
        hashMap.put(3,new User(4, "cc"));
        hashMap.put(4,new User(1, "dd"));
        for (User user: HashMapTest.sort(hashMap).values()) {
            System.out.println(user.toString());
        }

    }
}

```


***HashMap 底层***

HashMap 结构：数组+链表
+	主干是数组，数组中的元素是链表的头节点，key-value的形式。
+	链表存放的是hashCode()相同的节点
+	查询流程：通过hashCode()获取对应数组下标，从头节点开始遍历比较KEY，获取对应的键值对


几个重要的字段：
```
	//实际存储的key-value键值对的个数
	transient int size;
	//阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold
	
	//负载因子，代表了table的填充度有多少，默认是0.75
	final float loadFactor;
	//用于快速失败，由于HashMap非线程安全，在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），需要抛出异常ConcurrentModificationException
	transient int modCount;
```


HashMap基于hashing原理，我们通过put()和get()方法储存和获取对象。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象。


```
transient Node<K,V>[] table; //链表
transient Set<Map.Entry<K,V>> entrySet; //链表

```
HashMap可以通过下面的语句进行同步：
```
Map m = Collections.synchronizeMap(hashMap); 
```

当两个对象的hashcode相同会发生什么:
因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中。

当hashcode相同，你如何获取值对象：
调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。如果有两个值对象储存在同一个bucket，他给出答案:将会遍历链表直到找到值对象

如果HashMap的大小超过了负载因子(load factor)定义的容量：
默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

> HashMap可以接受null键值和值



***并发集合和普通集合如何区别***
并发集合常见的有 ConcurrentHashMap、ConcurrentLinkedQueue、ConcurrentLinkedDeque 等。并发集合位于 `java.util.concurrent`包下，是 `jdk1.5`之后才有的.

在 java 中有普通集合、同步（线程安全）的集合、并发集合。普通集合通常性能最高，但是不保证多线程的安全性和并发的可靠性。线程安全集合仅仅是给集合添加了 synchronized 同步锁，严重牺牲了性能，而且对并发的效率就更低了，并发集合则通过复杂的策略不仅保证了多线程的安全又提高的并发时的效率。

***Map 中的 key 和 value 可以为 null 么？***

HashMap 对象的 key、value 值均可为 null。
HahTable 对象的 key、value 值均不可为 null。


### 七、多线程

***传统多线程***

传统使用类 Thread 和接口 Runnable 实现：

+	继承Thread，重写run方法
+	实现Runnable接口

线程互斥与同步：

线程同步的主要任务是使并发执行的各线程之间能够有效的共享资源和相互合作，从而使程序的执行具有可再现性。
当线程并发执行时，由于资源共享和线程协作，使用线程之间会存在以下两种制约关系：间接相互制约（资源共享）、直接相互制约（前置线程）。

`间接相互制约可以称为互斥，直接相互制约可以称为同步。`

线程局部变量 ThreadLocal:

+	局部变量,用于实现线程内的数据共享;用`private static`加以修饰
+	每个线程调用全局 ThreadLocal 对象的 set 方法，在 set 方法中，首先根据当前线程独享的 ThreadLocalMap对象，该对象的Key是 ThreadLocal对象，值是用户设置的具体值。

```
// Date工具类
//    静态 SimpleDateFormat中【calendar】是共享的，多线程使用会有问题
public class DateUtil  {
//    线程对象池
    private static Map<String, ThreadLocal<SimpleDateFormat>> map = new HashMap<String, ThreadLocal<SimpleDateFormat>>();
//    锁对象
    private static Object lock = new Object();

    public static SimpleDateFormat getSimpleDateFormat (final String pattern) {
        ThreadLocal<SimpleDateFormat> local = map.get(pattern);
        if (local == null)  {
            synchronized (lock) {
                local = map.get(pattern);
                if (local == null) {
                    local = new ThreadLocal<SimpleDateFormat>(){
                        @Override
                        protected SimpleDateFormat initialValue() {
                            return new SimpleDateFormat(pattern);
                        }
                    };
                    map.put(pattern, local);
                }
            }
        }
        return local.get();
    }
}


//使用
format = DateUtil.getSimpleDateFormat(hg[hg.length-1]);

```

> 在线程结束时可以调用 ThreadLocal.remove()方法，这样会更快释放内存，不调用也可以，因为线程结束后也可以自动释放相关的 ThreadLocal 变量。



***线程并发库***

Java 5 添加了一个新的包到 Java 平台，java.util.concurrent 包。这个包包含有一系列能够让 Java 的并发编程变得更加简单轻松的类。

java.util.concurrent.atomic 包 (多线程的原子性操作提供的工具类)：可以对多线程的基本数据、数组中的基本数据和对象中的基本数据
进行多线程的操作
java.util.concurrent.lock 包 (多线程的锁机制)：为锁和等待条件提供一个框架的接口和类，它不同于内置同步和监视器。该框架允许更灵活地使用锁和条件

+	`Lock 接口`：支持那些语义不同（重入、公平等）的锁规则，可以在非阻塞式结构的上下文（包括 handover-hand 和锁重排算法）中使用这些规则。主要的实现是 ReentrantLock。 
+	`ReadWriteLock 接口`：以类似方式定义了一些读取者可以共享而写入者独占的锁。此包只提供了一个实现，即 ReentrantReadWriteLock，因为它适用于大部分的标准用法上下文。但程序员可以创建自己的、适用于非标准要求的实现。
+	`Condition 接口`：描述了可能会与锁有关联的条件变量。这些变量在用法上与使用 Object.wait 访问的隐式监视器类似，但提供了更强大的功能。需要特别指出的是，单个 Lock 可能与多个 Condition 对象关联。为了避免兼容性问题，Condition 方法的名称与对应的 Object 版本中的不同。




***什么是volatile***

+	volatile 修饰的变量，线程在每次使用变量的时候，都会读取变量修改后的最的值。
+	加锁可以保证原子性与可见性，而使用 volatile只能保证可见性，无法使操作原子化。
+	`volatile`不可以过度使用。例如，它不可以用于标注递增数（i++），因为它不能使自增操作原子化。



***什么是线程池***
线程池作用就是限制系统中执行线程的数量。

为什么要用线程池:
+	减少了创建和销毁线程的次数,线程可以被重复利用，可执行多个任务
+	可以根据系统的承受能力，调整线程池中工作线线程的数目,防止死机
+	按需获取

几个相关的类：
+	Executor 线程池
+	Executors 线程池工厂类
+	ExecutorService ExecutorService 接口表示一个异步执行机制，使我们能够在后台执行任务; 继承了`Executor`接口
+	ThreadPoolExecutor Executors的底层实现


***Executors创建线程池***

```
//创建固定大小的线程池
ExecutorService fPool = Executors.newFixedThreadPool(3);
//创建缓存大小的线程池
ExecutorService cPool = Executors.newCachedThreadPool();
//创建单一的线程池
ExecutorService sPool = Executors.newSingleThreadExecutor();

//延迟线程池
ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);
Thread t4 = new MyThread();
pool.schedule(t4, 10, TimeUnit.MILLISECONDS); //在 10 秒后执行
```

> 在java 的多线程中，一但线程关闭，就会成为死线程。关闭后死线程就没有办法在启动了。再次启动就会出现异常信息：Exception in thread "main" java.lang.IllegalThreadStateException。那么如何解决这个问题呢？我们这里就可以使用 Executors.newSingleThreadExecutor()来再次启动一个线程.


线程池实例：
```
public class ThreadTest {
    public static void main(String[] args) {
//        初始化线程池
        ExecutorService pool = Executors.newCachedThreadPool();
//        添加线程
        Thread tread = new MyThread();
        Thread tread2 = new MyThread();
        Thread tread3 = new MyThread();
        pool.execute(tread);
        pool.execute(tread2);
        pool.execute(tread3);
//        销毁线程池
        pool.shutdown();
    }
}

class MyThread extends Thread {
    //        执行任务
    @Override
    public void run() {
        super.run();
        System.out.println(Thread.currentThread().getName());
    }
}
```

通过`ExecutorService`异步执行任务：

通过`ExecutorService`的`execute`方法调用一个新线程，异步执行任务。一个线程将一个任务委派给一个 ExecutorService 去异步执行。
一旦该线程将任务委派给 ExecutorService，该线程将继续它自己的执行，独立于该任务的执行。
```
    public static void main(String[] args) {
//        初始化线程池
        ExecutorService pool = Executors.newCachedThreadPool();

        pool.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("test:"+Thread.currentThread().getName());
            }
        });
//        销毁线程池
        pool.shutdown();
    }
```




***不推荐使用 Executors***

```
// 底层是通过ThreadPoolExecutor来创建，只是进行了基本配置
// 其中，有new一个阻塞队列 LinkedBlockingQueue
public class Executors {
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
}
// 如下，通过构造器创建了Integer.MAX_VALUE大小的队列，会堆积大量的请求，从而造成OOM（内存溢出）
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
}

```

> 结论：推荐通过`ThreadPoolExecutor`手动创建线程池，参数设置则可以参考`Executors`


ExecutorService的实现
+	ThreadPoolExecutor
+	ScheduledThreadPoolExecutor

ExecutorService 创建：
+	Executors（不推荐）
+	ThreadPoolExecutor、ScheduledThreadPoolExecutor

ExecutorService 使用:
```
	execute(Runnable)  异步执行
	submit(Runnable)  要求一个 Runnable 实现类，但它返回一个 Future 对象。这个 Future 对象可以用来检查 Runnable 是否已经执行完毕
	submit(Callable)  类似于 submit(Runnable) 方法
	invokeAny(…)方法要求一系列的 Callable 或者其子接口的实例对象。调用这个方法并不会返回一个 Future，但它返回其中一个 Callable 对象的结果。无法保证返回的是哪个 Callable 的结果 – 只能表明其中一个已执行结束  
	invokeAll(…) 将调用你在集合中传给 ExecutorService 的所有 Callable 对象。invokeAll() 返回一系列的 Future 对象，通过它们你可以获取每个 Callable 的执行结果
```
Executors 关闭：

+	shutdown ：只是将空闲的线程 interrupt() 了，shutdown（）之前提交的任务可以继续执行直到结束。
+	shutdownNow ：interrupt 所有线程， 因此大部分线程将立刻被中断。之所以是大部分，而不是全部，是因为 interrupt()方法能力有限。

