---
title: 设计模式（二）结构型模式
date: 2020-04-10 15:24:58
tags: [设计模式]
---

|范围\目的|结构型模式|
|:--|:--|
|类模式|(类）适配器|
|对象模式 |代理<br/>(对象）适配器<br/>桥接<br/>装饰<br/>外观<br/>享元<br/>组合|


结构型模式：描述了如何将类或对象按某种布局组成更大的结构。它分为类结构型模式和对象结构型模式，前者采用继承机制来组织接口和类，后者釆用组合或聚合来组合对象。


由于组合关系或聚合关系比继承关系耦合度低，满足“合成复用原则”，所以对象结构型模式比类结构型模式具有更大的灵活性。
+	组合：部分与整体是与生俱来的，部分的存在依赖于整体，部分无法独立存在。
+	聚合：部分与整体是松散的，部分不依赖于整体，可以独立存在。动态组合，称之为聚合。


结构型模式分为以下 7 种：
+	代理（Proxy）模式：代理访问，隐藏实现，代理类需要实现被代理类相同的接口。
+	适配器（Adapter）模式：转换接口，避免接口不兼容。
+	桥接（Bridge）模式：将抽象与实现分离，用组合关系代替继承关系。
+	装饰（Decorator）模式：动态地给对象增加一些职责，即增加其额外的功能。
+	外观（Facade）模式：为多个复杂的子系统提供一个一致的接口，使这些子系统更加容易被访问。
+	享元（Flyweight）模式：运用共享技术来有效地支持大量细粒度对象的复用。
+	组合（Composite）模式：将对象组合成树状层次结构，使用户对单个对象和组合对象具有一致的访问性。


***

# 适配器模式——适配不兼容接口

***定义***

适配器模式（Adapter）的定义如下：将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作。适配器模式分为类结构型模式和对象结构型模式两种，前者类之间的耦合度比后者高，且要求程序员了解现有组件库中的相关组件的内部结构，所以应用相对较少些。

该模式的主要优点如下。
+	客户端通过适配器可以`透明地调用目标接口`。
+	`复用了现存的类`，程序员不需要修改原有代码而重用现有的适配者类。
+	`将目标类和适配者类解耦`，解决了目标类和适配者类接口不一致的问题。

其缺点是：对类适配器来说，更换适配器的实现过程比较复杂。

***结构***


适配器模式（Adapter）包含以下主要角色。
+	目标（Target）接口：业务所需要的接口，它可以是抽象类或接口。
+	适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口。
+	适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。

![适配器模式](/image/degin-pattern/spqms.png)

***类适配器***

```
// 适配器
public class MyAdapter extends MyAdaptee implements Target{
    // 实际需要的方法
    @Override
    public void haha() {
        System.out.println("业务代码1");
        hello();
        System.out.println("业务代码2");
    }
}

// 需要的接口
interface Target {
    void haha();
}
// 适配者
class MyAdaptee {
    public void hello() {
        System.out.println("hello");
    }
}
```

***对象适配器***

```
// 适配器模式是为了在不重构旧接口的情况下，启用新的接口又能使用旧接口服务

// 对象适配器
// 不继承适配者，因为继承会破坏类的封装性；通过组合的方式来使用适配者
public class MyAdapter2 implements Target2{
    MyAdaptee2 myAdaptee2;
    public MyAdapter2(MyAdaptee2 myAdaptee) {
        this.myAdaptee2 = myAdaptee;
    }
    @Override
    public void haha() {
        System.out.println("业务代码1");
        myAdaptee2.hello();
        System.out.println("业务代码2");
    }
}

// 需要的接口
interface Target2 {
    void haha();
}

// 旧接口
interface OldTarget2 {
    public void hello();
}

// 适配者
class MyAdaptee2 implements OldTarget2{
    @Override
    public void hello() {
        System.out.println("hello");
    }
}


```

***

# 代理模式——隐藏实现细节
***定义***

代理模式的定义：由于某些原因需要给某对象提供一个代理以控制对该对象的访问。这时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。

代理模式的主要优点有：
代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用；
代理对象可以扩展目标对象的功能；
代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度；

其主要缺点是：
在客户端和目标对象之间增加一个代理对象，会造成请求处理速度变慢；
增加了系统的复杂度；

 
> 注：这符合迪米特法则，即最少知识原则；减少类与类的通讯，而是通过代理进行访问。但是，代理模式也使得代码阅读性较差。

***结构***

+	抽象主题（Subject）类：通过接口或抽象类声明真实主题和代理对象实现的业务方法。
+	真实主题（Real Subject）类：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。
+	代理（Proxy）类：提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以访问、控制或扩展真实主题的功能。

![代理模式](/image/degin-pattern/dlms.png)
 
***代理模式实例***

```
/*代理*/
public class MyProxy implements Subject{
    RealSUbject realSUbject;

    @Override
    public synchronized void hello() {
        if (realSUbject == null) {
            realSUbject = new RealSUbject();
        }
        preRequest();
        realSUbject.hello();
        postRequest();
    }
    public void preRequest()
    {
        System.out.println("访问真实主题之前的预处理。");
    }
    public void postRequest()
    {
        System.out.println("访问真实主题之后的后续处理。");
    }
}

/*抽象主题*/
interface Subject {
    void hello();
}

/*真实主题*/
class RealSUbject implements Subject {
    @Override
    public void hello() {
        System.out.println("这是真实主题");
    }
}
```

***代理模式与适配器模式的区别***

适配器模式：当旧的接口无法满足新的需求，我们无法修改或者替换旧的接口，可以将新接口转换成旧接口（即新接口需要有旧接口的所有方法）；早期的枚举接口是Enumeration而后定义的枚举接口是Iterator;有很多旧的类实现了enumeration接口暴露出了一些服务，但是这些服务我们现在想通过传入Iterator接口而不是Enumeration接口来调用，这时就需要一个适配器。

代理模式：实现的接口与旧接口完全一致，不存在扩展；只是为了隐藏方法的具体实现，而是通过代理进行处理。

***

# 桥接模式——抽象与实现分离

***定义***

桥接（Bridge）模式的定义如下：
+	将抽象与实现分离，使它们可以独立变化。它是用组合关系代替继承关系来实现，从而降低了抽象和实现这两个可变维度的耦合度。

桥接（Bridge）模式的优点是：
+	由于抽象与实现分离，所以扩展能力强；
+	其实现细节对客户透明。

缺点是：
+	由于聚合关系建立在抽象层，要求开发者针对抽象化进行设计与编程，这增加了系统的理解与设计难度。


***结构***

桥接（Bridge）模式包含以下主要角色。
+	抽象化（Abstraction）角色：定义抽象类，并包含一个对实现化对象的引用。
+	扩展抽象化（Refined    Abstraction）角色：是抽象化角色的子类，实现父类中的业务方法，并通过组合关系调用实现化角色中的业务方法。
+	实现化（Implementor）角色：定义实现化角色的接口，供扩展抽象化角色调用。
+	具体实现化（Concrete Implementor）角色：给出实现化角色接口的具体实现。

如下图所示，例如自行车是一种交通工具，但依据使用地段的不同可以分为公路自行车、场地自行车，这是一种维度；而对于公路自行车来说，也有许多款式，这是第二种维度。
![桥接模式](/image/degin-pattern/qjms.png)

***桥接模式实例***

```

public class MyBridge {
    public static void main(String[] args) {
        MyImplementor myImplementor = new ConcreteImplementorA();
        Abstraction abstraction = new ExtendAbstraction(myImplementor);
        abstraction.hello();
    }
}

// 实现化角色
interface MyImplementor {
    void helloImpl();
}

// 具体实现化角色A
class ConcreteImplementorA implements MyImplementor {

    @Override
    public void helloImpl() {
        System.out.println("具体实现化角色A:hello");
    }
}

// 抽象角色
// 抽象类中的方法需要显式声明为抽象的
abstract class Abstraction {
    protected MyImplementor myImplementor;
    public Abstraction(MyImplementor myImplementor) {
        this.myImplementor = myImplementor;
    }
    abstract void hello();
}

// 抽象角色实现
class ExtendAbstraction extends Abstraction {
    public ExtendAbstraction(MyImplementor myImplementor) {
        super(myImplementor);
    }
    @Override
    void hello() {
        myImplementor.helloImpl();
    }
}
```

***

# 装饰模式——动态增加职责

***定义***

装饰（Decorator）模式的定义：指在不改变现有对象结构的情况下，动态地给该对象增加一些职责（即增加其额外功能）的模式，它属于对象结构型模式。

装饰（Decorator）模式的主要优点有：
+	采用装饰模式扩展对象的功能比采用继承方式更加灵活。
+	可以设计出多个不同的具体装饰类，创造出多个不同行为的组合。

其主要缺点是：装饰模式增加了许多子类，如果过度使用会使程序变得很复杂。

***结构***

装饰模式主要包含以下角色。
+	抽象构件（Component）角色：定义一个抽象接口以规范准备接收附加责任的对象。
+	具体构件（Concrete    Component）角色：实现抽象构件，通过装饰角色为其添加一些职责。
+	抽象装饰（Decorator）角色：继承抽象构件，并包含具体构件的实例，可以通过其子类扩展具体构件的功能。
+	具体装饰（ConcreteDecorator）角色：实现抽象装饰的相关方法，并给具体构件对象添加附加的责任。

![装饰模式](/image/degin-pattern/zsms.png)

***实例***


```
public class MyDecorator {
    public static void main(String[] args) {
        Component component = new ConcreteComponent();
        Component d = new ConcreteDecorator(component);
        d.operation();
    }
}

//抽象构件角色
interface  Component {
    void operation();
}

// 具体构件角色
class ConcreteComponent implements Component {
    @Override
    public void operation() {
        System.out.println("基本构件");
    }
}

// 抽象装饰角色
abstract class Decorator implements Component {
    Component component;

    public Decorator(Component component) {
        this.component = component;
    }
    @Override
    public void operation() {
        component.operation();
    }
}

// 具体装饰角色
class ConcreteDecorator extends Decorator {

    public ConcreteDecorator(Component component) {
        super(component);
    }

    @Override
    public void operation() {
        super.operation();
        addDecor();
    }
    void addDecor() {
        System.out.println("额外装饰");
    }
}
```

***扩展：JDK中装饰模式的使用***

在Java的IO流中，用到了装饰模式。`FilterInputStream`就是一个经典的抽象装饰类，拥有多个具体装饰类。而`FileInputStream`是一个具体构件，InputStream是抽象构件。
源码如下：
```
// 抽象构件
public abstract class InputStream implements Closeable {
	...
	public abstract int read() throws IOException;
	...
}

// 具体构件
public class FileInputStream extends InputStream {
	...
	public int read() throws IOException {
        return read0();
    }
	...
}

// 抽象装饰类
public class FilterInputStream extends InputStream {
	...
	protected volatile InputStream in;
	
    protected FilterInputStream(InputStream in) {
        this.in = in;
    }
	public int read() throws IOException {
        return in.read();
    }
	...
}

// 具体装饰类
public class BufferedInputStream extends FilterInputStream {
	...
	public synchronized int read() throws IOException {
		// 缓冲区读取结束，重新填充
        if (pos >= count) {
            fill();
            if (pos >= count)
                return -1;
        }
        return getBufIfOpen()[pos++] & 0xff;
    }
	
	private void fill() throws IOException {
        byte[] buffer = getBufIfOpen();
        if (markpos < 0)
            pos = 0;            /* no mark: throw away the buffer */
        else if (pos >= buffer.length)  /* no room left in buffer */
            if (markpos > 0) {  /* can throw away early part of the buffer */
                int sz = pos - markpos;
                System.arraycopy(buffer, markpos, buffer, 0, sz);
                pos = sz;
                markpos = 0;
            } else if (buffer.length >= marklimit) {
                markpos = -1;   /* buffer got too big, invalidate mark */
                pos = 0;        /* drop buffer contents */
            } else if (buffer.length >= MAX_BUFFER_SIZE) {
                throw new OutOfMemoryError("Required array size too large");
            } else {            /* grow buffer */
                int nsz = (pos <= MAX_BUFFER_SIZE - pos) ?
                        pos * 2 : MAX_BUFFER_SIZE;
                if (nsz > marklimit)
                    nsz = marklimit;
                byte nbuf[] = new byte[nsz];
                System.arraycopy(buffer, 0, nbuf, 0, pos);
                if (!bufUpdater.compareAndSet(this, buffer, nbuf)) {
                    // Can't replace buf if there was an async close.
                    // Note: This would need to be changed if fill()
                    // is ever made accessible to multiple threads.
                    // But for now, the only way CAS can fail is via close.
                    // assert buf == null;
                    throw new IOException("Stream closed");
                }
                buffer = nbuf;
            }
        count = pos;
		// 获取基础构件，调用read方法
        int n = getInIfOpen().read(buffer, pos, buffer.length - pos);
        if (n > 0)
            count = n + pos;
    }
	...
}
```

可以从以上的源码看出，在读取文件的时候，`BufferedInputStream`借助装饰模式对FileInputStream的读取方法进行了拓展，添加了缓冲功能。


***

# 外观模式——为子系统中接口提供一致的界面

***定义***

外观（Facade）模式的定义：是一种通过为多个复杂的子系统提供一个一致的接口，而使这些子系统更加容易被访问的模式。该模式对外有一个统一接口，外部应用程序不用关心内部子系统的具体的细节，这样会大大降低应用程序的复杂度，提高了程序的可维护性。

外观（Facade）模式是“迪米特法则”的典型应用，它有以下主要优点：
+	降低了子系统与客户端之间的耦合度，使得子系统的变化不会影响调用它的客户类。
+	对客户屏蔽了子系统组件，减少了客户处理的对象数目，并使得子系统使用起来更加容易。
+	降低了大型软件系统中的编译依赖性，简化了系统在不同平台之间的移植过程，因为编译一个子系统不会影响其他的子系统，也不会影响外观对象。

外观（Facade）模式的主要缺点如下
+	不能很好地限制客户使用子系统类。
+	增加新的子系统可能需要修改外观类或客户端的源代码，违背了“开闭原则”。


***结构***

外观（Facade）模式包含以下主要角色。
+	外观（Facade）角色：为多个子系统对外提供一个共同的接口。
+	子系统（Sub System）角色：实现系统的部分功能，客户可以通过外观角色访问它。
+	客户（Client）角色：通过一个外观角色访问各个子系统的功能。

![外观模式](/image/degin-pattern/wgms.png)

***实例***

```
public class FacadePattern {
    public static void main(String[] args) {
        Facade f=new Facade();
        f.method();
    }
}

class Facade {
    private SubSystem01 obj1=new SubSystem01();
    private SubSystem02 obj2=new SubSystem02();
    private SubSystem03 obj3=new SubSystem03();
    public void method() {
        obj1.method1();
        obj2.method2();
        obj3.method3();
    }
}

//子系统角色
class SubSystem01
{
    public  void method1() {
        System.out.println("子系统01的method1()被调用！");
    }
}
//子系统角色
class SubSystem02
{
    public  void method2() {
        System.out.println("子系统02的method2()被调用！");
    }
}
//子系统角色
class SubSystem03
{
    public  void method3() {
        System.out.println("子系统03的method3()被调用！");
    }
}
```


***

# 享元模式——共享内部数据

***定义***

享元（Flyweight）模式的定义：运用共享技术来有効地支持大量细粒度对象的复用。它通过共享已经存在的又橡来大幅度减少需要创建的对象数量、避免大量相似类的开销，从而提高系统资源的利用率。

享元模式的主要优点是：
+	相同对象只要保存一份，这降低了系统中对象的数量，从而降低了系统中细粒度对象给内存带来的压力。

其主要缺点是：
+	为了使对象可以共享，需要将一些不能共享的状态外部化，这将增加程序的复杂性。
+	读取享元模式的外部状态会使得运行时间稍微变长。


***结构***

享元模式中存在以下两种状态：
+	内部状态，即不会随着环境的改变而改变的可共享部分；
+	外部状态，指随环境改变而改变的不可以共享的部分。享元模式的实现要领就是区分应用中的这两种状态，并将外部状态外部化


享元模式的主要角色有如下。
+	抽象享元角色（Flyweight）:是所有的具体享元类的基类，为具体享元规范需要实现的公共接口，非享元的外部状态以参数的形式通过方法传入。
+	具体享元（Concrete Flyweight）角色：实现抽象享元角色中所规定的接口。
+	非享元（Unsharable Flyweight)角色：是不可以共享的外部状态，它以参数的形式注入具体享元的相关方法中。
+	享元工厂（Flyweight Factory）角色：负责创建和管理享元角色。当客户对象请求一个享元对象时，享元工厂检査系统中是否存在符合要求的享元对象，如果存在则提供给客户；如果不存在的话，则创建一个新的享元对象。

![享元模式](/image/degin-pattern/xyms.png)

***实例***

```
public class FlyweightPattern {
    public static void main(String[] args) {
        FlyweightFactory flyweightFactory = new FlyweightFactory();
        Flyweight f01 = flyweightFactory.getFlyweight("a");
        f01.opra(new UnsharedFlyweight("第一次调用a"));
    }

}

// 非享元角色
class UnsharedFlyweight {
    private String message;

    public UnsharedFlyweight(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

// 抽象享元角色
interface Flyweight {
    void opra(UnsharedFlyweight unsharedFlyweight);
}


//具体享元角色
class ConcreteFlyweight implements Flyweight {
    private String name;

    public ConcreteFlyweight(String name) {
        this.name = name;
    }

    @Override
    public void opra(UnsharedFlyweight unsharedFlyweight) {
        System.out.println("享元信息："+name);
        System.out.println("非享元信息："+unsharedFlyweight.getMessage());
    }
}
//享元工厂
class FlyweightFactory {
    private HashMap<String, Flyweight> flyweightHashMap = new HashMap<>();
    public synchronized Flyweight getFlyweight (String name) {
        Flyweight flyweight = flyweightHashMap.get(name);
        if (flyweight == null) {
            flyweight = new ConcreteFlyweight(name);
            flyweightHashMap.put(name, flyweight);
        } else {
            System.out.println("享元 "+name+" 已存在");
        }
        return flyweight;
    }
}

```

***

# 组合模式——动态访问

***定义***

组合（Composite）模式的定义：
+	有时又叫作部分-整体模式，它是一种`将对象组合成树状的层次结构`的模式，用来表示“部分-整体”的关系，使用户对单个对象和组合对象具有一致的访问性。

组合模式的主要优点有：
+	组合模式使得客户端代码可以一致地处理单个对象和组合对象，无须关心自己处理的是单个对象，还是组合对象，这简化了客户端代码；
+	更容易在组合体内加入新的对象，客户端不会因为加入了新的对象而更改源代码，满足“开闭原则”；

其主要缺点是：
+	设计较复杂，客户端需要花更多时间理清类之间的层次关系；
+	不容易限制容器中的构件；
+	不容易用继承的方法来增加构件的新功能；

***结构***

组合模式包含以下主要角色。
+	抽象构件（Component）角色：它的主要作用是为树叶构件和树枝构件声明公共接口，并实现它们的默认行为。在透明式的组合模式中抽象构件还声明访问和管理子类的接口；在安全式的组合模式中不声明访问和管理子类的接口，管理工作由树枝构件完成。
+	树叶构件（Leaf）角色：是组合中的叶节点对象，它没有子节点，用于实现抽象构件角色中 声明的公共接口。
+	树枝构件（Composite）角色：是组合中的分支节点对象，它有子节点。它实现了抽象构件角色中声明的接口，它的主要作用是存储和管理子部件，通常包含 Add()、Remove()、GetChild() 等方法。

组合模式分为：
+	透明式的组合模式
	+	由于抽象构件声明了两种构件中的全部方法，所以客户端无须区别树叶对象和树枝对象，对客户端来说是透明的

![组合模式](/image/degin-pattern/zhms.png)

+	安全式的组合模式
	+	将管理树叶构件的任务交付给树枝构件，抽象构件和树叶构件没有对子对象的管理方法，避免了安全问题。但是，客户端调用时需要了解构件的类型，这就失去了透明性。

![组合模式](/image/degin-pattern/zhms1.png)


***实例***

```
public class CompositePattern
{
    public static void main(String[] args)
    {
        Component c0=new Composite(); 
        Component c1=new Composite(); 
        Component leaf1=new Leaf("1"); 
        Component leaf2=new Leaf("2"); 
        Component leaf3=new Leaf("3");          
        c0.add(leaf1); 
        c0.add(c1);
        c1.add(leaf2); 
        c1.add(leaf3);          
        c0.operation(); 
    }
}
//抽象构件
interface Component
{
    public void add(Component c);
    public void remove(Component c);
    public Component getChild(int i);
    public void operation();
}
//树叶构件
class Leaf implements Component
{
    private String name;
    public Leaf(String name)
    {
        this.name=name;
    }
    public void add(Component c){ }           
    public void remove(Component c){ }   
    public Component getChild(int i)
    {
        return null;
    }   
    public void operation()
    {
        System.out.println("树叶"+name+"：被访问！"); 
    }
}
//树枝构件
class Composite implements Component
{
    private ArrayList<Component> children=new ArrayList<Component>();   
    public void add(Component c)
    {
        children.add(c);
    }   
    public void remove(Component c)
    {
        children.remove(c);
    }   
    public Component getChild(int i)
    {
        return children.get(i);
    }   
    public void operation()
    {
        for(Object obj:children)
        {
            ((Component)obj).operation();
        }
    }    
}

```


***组合模式的应用场景***

前面分析了组合模式的结构与特点，下面分析它适用的以下应用场景。
+	在需要表示一个对象整体与部分的层次结构的场合。
+	要求对用户隐藏组合对象与单个对象的不同，用户可以用统一的接口使用组合结构中的所有对象的场合。


***

# 参考文章
> [模式的秘密-适配器模式和代理模式的区别](https://www.cnblogs.com/alsf/p/8506912.html)