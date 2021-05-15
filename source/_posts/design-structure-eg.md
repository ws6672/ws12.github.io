---
title: 结构型模式——实例
date: 2021-03-29 14:12:18
tags: [设计模式]
---


# 适配器模式（兼容）

适配器模式（Adapter）的定义如下：将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作；可以分为类适配器模式以及对象适配器模式。

1. 类适配器模式

类适配器模式可采用多重继承方式实现，如 C++ 可定义一个适配器类来同时继承当前系统的业务接口和现有组件库中已经存在的组件接口；Java 不支持多继承，但可以定义一个适配器类来实现当前系统的业务接口，同时又继承现有组件库中已经存在的组件。


相关实例如下：

```

/**
 * @author zws
 * @date 2021/3/29 12:50
 * @Description 类适配器
 */

// 适配器类：通过包装一个需要适配的对象，把原接口转换成目标接口。
public class AdapterTest extends Adaptee implements AdapterTarget{
    public static void main (String[] args) {
        AdapterTarget target = new AdapterTest();
        target.request();
    }

    @Override
    public void request() {

    }

    @Override
    public void request2() {

    }

    @Override
    public void request3() {

    }
}

// 目标接口：目标抽象类定义客户所需接口，可以是一个抽象类或接口，也可以是具体类
interface AdapterTarget {
    void request();
    void request2();
    void request3();

}

// 适配者接口：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体类，包含了客户希望使用的业务方法
class Adaptee {
    public void adapteeRequest() {
        System.out.println("适配者接口的业务代码");
    }
}


```

2. 对象适配器模式：对于这种对象的适配器模式，实际上就是通过一个适配器类，把目标类和需要被适配的类进行组合。所以适配器类Adapter一般需要继承或实现Targert，并且还得持有Adaptee的实例引用。

```
// 对象适配器
class ObjectAdapter implements AdapterTarget {
    private Adaptee adaptee;

    public ObjectAdapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.adapteeRequest();
    }

    @Override
    public void request2() {

    }

    @Override
    public void request3() {

    }
}

// 测试
public class AdapterTest {
    public static void main (String[] args) {
        ObjectAdapter adapter = new ObjectAdapter(new Adaptee());
        adapter.request();
    }
}

```


3. 为什么使用适配器模式

假如在旧的接口涉及到其它功能，这时候我们需要增加一个新的参数。如果在旧有的接口做修改，会影响到其它的实现类。所以，我们可以通过定义适配器来扩展接口，实现和老代码的兼容。但是，如果过渡使用适配器模式，会导致代码结构散乱，难以重构。


# 代理模式（附加功能与核心功能解耦）

在代理模式（Proxy Pattern）中，一个类代表另一个类的功能。这种类型的设计模式属于结构型模式。代理模式通过增加中间层，为其他对象提供一种代理以控制对这个对象的访问，还可以想在访问一个类时做一些控制。代理模式分为:
+	静态代理：所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。
+	动态代理：动态代理是在实现阶段不用关心代理类，而在运行阶段才指定哪一个对象。

相关实例如下：
```

/**
 * @author zws
 * @date 2021/3/29 13:19
 * @Description 代理模式
 */

public class ProxyTest {
    public static void main (String[] args) {
        ProxyTool tool = new ProxyTool(new RealTool());
        tool.description();
    }
}

//抽象接口
interface Tool {
    void description();
}

class RealTool implements Tool{
    @Override
    public void description() {
        try {
            System.out.println("这是一个被代理类的描述方法");
            Thread.sleep((int) (Math.random()*1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class ProxyTool implements Tool {
    Tool tool;

    public ProxyTool(Tool tool) {
        this.tool = tool;
    }

    @Override
    public void description() {
        long startTime = System.currentTimeMillis();
        tool.description();
        System.out.println((System.currentTimeMillis()-startTime) + "毫秒");
    }
}
```

代理模式常用在业务系统中开发一些非功能性需求，比如：监控、统计、鉴权、限流、事务、幂等、日志。我们将这些附加功能与业务功能解耦，放到代理类统一处理，让程序员只需要关注业务方面的开发。除此之外，代理模式还可以用在 RPC、缓存等应用场景中。



# 桥接模式（多维度解耦）

如果一个类存在两个（或多个）独立变化的维度，我们通过组合的方式，让这两个（或多个）维度可以独立进行扩展。

桥接模式即将抽象部分与它的实现部分分离开来，使他们都可以独立变化。桥接模式将继承关系转化成关联关系，它降低了类与类之间的耦合度，减少了系统中类的数量，也减少了代码量。

+	抽象化：其概念是将复杂物体的一个或几个特性抽出去而只注意其他特性的行动或过程。在面向对象就是将对象共同的性质抽取出去而形成类的过程。
+	实现化：针对抽象化给出的具体实现。它和抽象化是一个互逆的过程，实现化是对抽象化事物的进一步具体化。
+	脱耦：脱耦就是将抽象化和实现化之间的耦合解脱开，或者说是将它们之间的强关联改换成弱关联，将两个角色之间的继承关系改为关联关系。

桥接模式主要包含如下几个角色：
+	Abstraction：抽象类
+	RefinedAbstraction：扩充抽象类
+	Implementor：实现类接口
+	ConcreteImplementor：具体实现类

相关实例如下：

```

/**
 * @author zws
 * @date 2021/3/29 13:33
 * @Description 桥接模式
 */
public class BridgeTest {
    public static void main (String[] args) {
        Shape shape = new Circle();
        Color color = new White();
        shape.setColor(color);
        shape.draw();
    }
}


// 抽象类
abstract class Shape {
    Color color;

    public void setColor(Color color) {
        this.color = color;
    }

    public abstract void draw();
}
// 扩充抽象类
class Circle extends Shape{

    @Override
    public void draw() {
        color.bepaint("正方形");
    }
}

// 实现类接口
interface Color {
    public void bepaint(String shape);
}
// 具体实现类
class White implements Color{
    @Override
    public void bepaint(String shape) {
        System.out.println("白色的" + shape);
    }

}
```

# 装饰模式（扩展）

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。装饰器模式是为了动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。

相关实例如下：
```
/**
 * @author zws
 * @date 2021/3/29 13:40
 * @Description 装饰器模式
 */
public class DecoratorTest {
    public static void main (String[] args) {
        Bird bird = new Eagle();
        BirdDecorator decorator = new WhiteBirdDecorator(bird);
        decorator.des();
    }
}

// 抽象构件（Component）角色：给出一个抽象接口，以规范准备接收附加责任的对象
interface Bird {
    void des();
}

// 具体构件（Concrete Component）角色：定义一个将要接收附加责任的类
class Eagle implements Bird {
    @Override
    public void des() {
        System.out.println("小型至中型的白昼活动的鹰形类鸟");
    }
}

// 装饰（Decorator）角色：持有一个构件（Component）对象的实例，并实现一个与抽象构件接口一致的接口
abstract class BirdDecorator implements Bird {
    public Bird bird;

    public BirdDecorator(Bird bird) {
        this.bird = bird;
    }

    @Override
    public void des() {
        bird.des();
    }
}

// 具体装饰（Concrete Decorator）角色：负责给构件对象添加上附加的责任
class WhiteBirdDecorator extends  BirdDecorator {
    public WhiteBirdDecorator(Bird bird) {
        super(bird);
    }

    @Override
    public void des() {
        bird.des();
        extend();
    }

    public void extend() {
        System.out.println("白色的鸟");
    }
}
```

# 外观模式（同一接口）

外观模式提供了一个统一的接口，用来访问子系统中的一群接口，外观定义了一个高层接口，让子系统更容易使用。

外观（Facade）模式是“迪米特法则”的典型应用，它有以下主要优点：
+	降低了子系统与客户端之间的耦合度，使得子系统的变化不会影响调用它的客户类。
+	对客户屏蔽了子系统组件，减少了客户处理的对象数目，并使得子系统使用起来更加容易。
+	降低了大型软件系统中的编译依赖性，简化了系统在不同平台之间的移植过程，因为编译一个子系统不会影响其他的子系统，也不会影响外观对象。

相关实例如下：
```
/**
 * @author zws
 * @date 2021/3/29 13:57
 * @Description 外观模式
 */

public class FacadeTest {
    public static void main (String[] args) {
        Facade facade = new Facade();
        facade.method();
    }
}

//外观角色
class Facade {
    private SubSystem01 obj1 = new SubSystem01();
    private SubSystem02 obj2 = new SubSystem02();
    private SubSystem03 obj3 = new SubSystem03();

    public void method() {
        obj1.method1();
        obj2.method2();
        obj3.method3();
    }
}

//子系统角色
class SubSystem01 {
    public void method1() {
        System.out.println("子系统01的method1()被调用！");
    }
}

//子系统角色
class SubSystem02 {
    public void method2() {
        System.out.println("子系统02的method2()被调用！");
    }
}

//子系统角色
class SubSystem03 {
    public void method3() {
        System.out.println("子系统03的method3()被调用！");
    }
}
```

# 享元模式（对象共享）

享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。

模式所涉及的角色：
+	Flyweight： 享元接口，通过这个接口传入外部状态并作用于外部状态；
+	ConcreteFlyweight： 具体的享元实现对象，必须是可共享的，需要封装享元对象的内部状态；
+	UnsharedConcreteFlyweight： 非共享的享元实现对象，并不是所有的享元对象都可以共享，非共享的享元对象通常是享元对象的组合对象；
+	FlyweightFactory： 享元工厂，主要用来创建并管理共享的享元对象，并对外提供访问共享享元的接口；

享元模式的核心在于享元工厂类，享元工厂类的作用在于提供一个用于存储享元对象的享元池，用户需要对象时，首先从享元池中获取，如果享元池中不存在，则创建一个新的享元对象返回给用户，并在享元池中保存该新增对象。

享元对象能做到共享的关键是区分两个状态：
内部状态(Internal State)：存储在享元对象内部并且不会随环境改变而改变的状态
外部状态(External State)：随环境改变而改变的、不可以共享的状态

相关实例如下：
```
// 接口
public interface Shape {
   void draw();
}

// 创建实现接口的实体类
public class Circle implements Shape {
   private String color;
   private int x;
   private int y;
   private int radius;
 
   public Circle(String color){
      this.color = color;     
   }
 
   public void setX(int x) {
      this.x = x;
   }
 
   public void setY(int y) {
      this.y = y;
   }
 
   public void setRadius(int radius) {
      this.radius = radius;
   }
 
   @Override
   public void draw() {
      System.out.println("Circle: Draw() [Color : " + color 
         +", x : " + x +", y :" + y +", radius :" + radius);
   }
}


// 享元工厂

public class ShapeFactory {
   private static final HashMap<String, Shape> circleMap = new HashMap<>();
 
   public static Shape getCircle(String color) {
      Circle circle = (Circle)circleMap.get(color);
 
      if(circle == null) {
         circle = new Circle(color);
         circleMap.put(color, circle);
         System.out.println("Creating circle of color : " + color);
      }
      return circle;
   }
}

public class FlyweightPatternDemo {
   private static final String colors[] = 
      { "Red", "Green", "Blue", "White", "Black" };
   public static void main(String[] args) {
 
      for(int i=0; i < 20; ++i) {
         Circle circle = 
            (Circle)ShapeFactory.getCircle(getRandomColor());
         circle.setX(getRandomX());
         circle.setY(getRandomY());
         circle.setRadius(100);
         circle.draw();
      }
   }
   private static String getRandomColor() {
      return colors[(int)(Math.random()*colors.length)];
   }
   private static int getRandomX() {
      return (int)(Math.random()*100 );
   }
   private static int getRandomY() {
      return (int)(Math.random()*100);
   }
}
```

# 组合模式（部分-整体）

组合模式（Composite Pattern），又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。

相关实例如下：
```
public class Employee {
   private String name;
   private String dept;
   private int salary;
   private List<Employee> subordinates;
 
   //构造函数
   public Employee(String name,String dept, int sal) {
      this.name = name;
      this.dept = dept;
      this.salary = sal;
      subordinates = new ArrayList<Employee>();
   }
 
   public void add(Employee e) {
      subordinates.add(e);
   }
 
   public void remove(Employee e) {
      subordinates.remove(e);
   }
 
   public List<Employee> getSubordinates(){
     return subordinates;
   }
 
   public String toString(){
      return ("Employee :[ Name : "+ name 
      +", dept : "+ dept + ", salary :"
      + salary+" ]");
   }   
}

public class CompositePatternDemo {
   public static void main(String[] args) {
      Employee CEO = new Employee("John","CEO", 30000);
 
      Employee headSales = new Employee("Robert","Head Sales", 20000);
 
      Employee headMarketing = new Employee("Michel","Head Marketing", 20000);
 
      Employee clerk1 = new Employee("Laura","Marketing", 10000);
      Employee clerk2 = new Employee("Bob","Marketing", 10000);
 
      Employee salesExecutive1 = new Employee("Richard","Sales", 10000);
      Employee salesExecutive2 = new Employee("Rob","Sales", 10000);
 
      CEO.add(headSales);
      CEO.add(headMarketing);
 
      headSales.add(salesExecutive1);
      headSales.add(salesExecutive2);
 
      headMarketing.add(clerk1);
      headMarketing.add(clerk2);
 
      //打印该组织的所有员工
      System.out.println(CEO); 
      for (Employee headEmployee : CEO.getSubordinates()) {
         System.out.println(headEmployee);
         for (Employee employee : headEmployee.getSubordinates()) {
            System.out.println(employee);
         }
      }        
   }
}
```

