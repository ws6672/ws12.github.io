---
title: 创建型模式——实例
date: 2021-03-29 12:37:09
tags: [设计模式]
---



# 单例模式

1. 懒汉式

```
public class Singleton {
    private static Singleton instance;
    private Singleton() {
    }
    public synchronized static Singleton getInstance() {
//       懒汉式 需要时再初始化
         if (instance == null) {
             instance = new Singleton();
         } else {
             System.out.println("实例对象已创建");
         }
         return instance;
    }
}
```

2. 饿汉式
```
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton() {
    }
    public synchronized static Singleton getInstance() {
//        饿汉式 定义时初始化
		return instance;
    }
}
```

3. DCL（即Double Check Lock，双重检查锁定）

```
public class Singleton {
    private volatile static Singleton instance;
    private Singleton() {
    }
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

4. 静态内部类

第一次加载Singleton类时不会初始化instance，只有在第一次调用getInstance()方法时，虚拟机会加载SingletonHolder类，初始化instance。保证线程安全，单例对象的唯一，也延迟了单例的初始化

```
public class Singleton {

    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
    private static class SingletonHolder {
        private static Singleton instance = new Singleton();
    }
}
```

5. 枚举单例

默认枚举实例的创建是线程安全的，即使反序列化也不会生成新的实例，任何情况下都是一个单例。

```
public enum Singleton {
    INSTANCE;
    public void doSomething() {
        System.out.println("do something");
    }
}
```

# 原型模式

用原型实例可以指定创建对象的种类，并通过拷贝这些原型创建新的对象。

[原型模式](/image/degin-pattern/proto_use.png)

例如：
```
/**
 * @author zws
 * @date 2021/3/27 14:59
 * @Description 原型模式(Prototype模式)是指：用原型实例指定创建对象的种类，并且通过拷 贝这些原型，创建新的对象 。
 *  原型模式是一种创建型设计模式，允许一个对象再创建另外一个可定制的对象， 无需知道如何创建的细节 。通过将一个原型对象传给那个要发动创建的对象，这个要发动创建 的对象通过请求原型对象拷贝它们自己来实施创建，即 对象.clone()。
 */
public class Prototype implements Cloneable{
    private String name;
    private Date date;

    public void setName(String name) {
        this.name = name;
    }

    public void setDate(Date date) {
        this.date = date;
    }

    public static void main (String[] args) throws CloneNotSupportedException, InterruptedException {
        Prototype prototype = new Prototype("test", new Date());
        Prototype clone = prototype.clone();
        Thread.sleep(1000);
        prototype.setDate(new Date());
        prototype.setName("result");

        System.out.println(prototype.name+":"+prototype.date);
        System.out.println(clone.name+":"+clone.date);
    }
    public Prototype() {
    }

    public Prototype(String name, Date date) {
        this.name = name;
        this.date = date;
    }

    @Override
    protected Prototype clone() throws CloneNotSupportedException {
        Prototype prototype = null;
        try {
            prototype = (Prototype) super.clone();
//            如果属性包含，自定义类需要实现clone方法，才能实现深拷贝，否则会指向同一块内存空间
//            prototype.test = (Test) test.clone();
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {
            return prototype;
        }
    }
}
```

原型模式需要注意的几点：
+	克隆对象不会调用构造方法
+	访问权限对原型模式无效
+	当我们的类初始化需要消耗很多的资源时，就可以使用原型模式，因为我们的克隆不会执行构造方法，避免了初始化占有的时间和空间
+	一个对象被其她对象访问，并且能够修改时，访问权限都无效了，什么都能修改


# 建造者模式（创建与使用分离）

建造者模式将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。在用户不知道对象的建造过程和细节的情况下就可以直接创建复杂的对象。用户只需要给出指定复杂对象的类型和内容 建造者模式负责按顺序创建复杂对象（把内部的建造过程和细节隐藏起来)。

相关实例如下：

```
/**
 * @author zws
 * @date 2021/3/27 15:22
 * @Description 建造者模式
 */
public class Builder {
    public static void main (String[] args) throws Exception {
        User user = User.builder().userName("test").id(1).address("bj").password("111").build();
        System.out.println(user.toString());
    }
}

class User {
    @Override
    public String toString() {
        return super.toString();
    }

    private Integer id;
    private String username;
    private String password;
    private String address;
//    private String status;
//    private String role;
//    private String realname;
//    private Date register_time;
//    private String register_ip;
//    private Date login_time;
//    private String login_ip;

    private User(Integer id, String username, String password, String address) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.address = address;
    }
    public static UserBuilder builder() {
        return new UserBuilder();
    }
    static class UserBuilder {
        private Integer id;
        private String username;
        private String password;
        private String address;

        private UserBuilder() {
        }

        public UserBuilder id(Integer id) {
            this.id = id;
            return this;
        }

        public UserBuilder userName(String username) {
            this.username = username;
            return this;
        }

        public UserBuilder password(String password) {
            this.password = password;
            return this;
        }

        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }
        public User build() throws Exception {
            if (username==null||password==null) {
                throw new Exception("用户名或密码不可为空");
            }
            return new User(id, username, password, address);
        }
    }
}
```

# 工厂模式

工厂模式可以分为
+	类模式，即工厂方法模式；
+	对象模式，即抽象工厂模式。


工厂方法模式实例：
```
/**
 * @author zws
 * @date 2021/3/23 18:07
 * @Description 工厂方法 一个抽象工厂 多个实际工厂，一个抽象产品 多个实际产品，一个实际工厂对应一个实际产品
 */
public class FactoryMethod {
    public static void main (String[] args) {
        AbstractFactory factory = new ConcreteFactory1();
        factory.newProduct().show();
    }
}

//抽象产品 产品规范
interface Product {
    void show();
}
//具体产品 产品细节
class ConcreteProduct1 implements Product {

    @Override
    public void show() {
        System.out.println("实际产品1");
    }
}
class ConcreteProduct2 implements Product {
    @Override
    public void show() {
        System.out.println("实际产品2");
    }
}
//抽象工厂 创建产品的接口
abstract class AbstractFactory {
    abstract Product newProduct();
}
//具体工厂 创建产品
class ConcreteFactory1 extends AbstractFactory {

    @Override
    Product newProduct() {
        System.out.println("实际工厂1-->");
        return new ConcreteProduct1();
    }
}

class ConcreteFactory2 extends AbstractFactory {
    @Override
    Product newProduct() {
        System.out.println("实际工厂2-->");
        return new ConcreteProduct2();
    }
}
```

抽象工厂模式实例如下：
```
/**
 * @author zws
 * @date 2021/3/23 18:19
 * @Description 抽象工厂模式：在工厂方法中，只考虑同种等级的产品；但是，在实际应用中，多产品才是主流。而 抽象工厂模式则考虑多等级产品的生产。
 */
public class AbstractFactoryMethod {
    public static void main (String[] args) {
        AbstractFactory2 factory = new MyFactory1();
        factory.newProduct1().show();
        factory.newProduct2().show();
        AbstractFactory2 factory2 = new MyFactory2();
        factory2.newProduct1().show();
        factory2.newProduct2().show();
    }
}

//抽象工厂 创建产品的接口
abstract class AbstractFactory2 {
    abstract Product2 newProduct1();
    abstract Product2 newProduct2();
}
interface Product2 {
    void show();
}

class Product211 implements Product2 {
    @Override
    public void show() {
        System.out.println("实际产品1-1");
    }
}

class Product212 implements Product2 {
    @Override
    public void show() {
        System.out.println("实际产品1-2");
    }
}
class Product221 implements Product2 {
    @Override
    public void show() {
        System.out.println("实际产品2-1");
    }
}
class Product222 implements Product2 {
    @Override
    public void show() {
        System.out.println("实际产品2-2");
    }
}

class MyFactory1 extends AbstractFactory2 {
    public MyFactory1() {
        System.out.println("实际工厂1");
    }

    @Override
    Product2 newProduct1() {
        return new Product211();
    }

    @Override
    Product2 newProduct2() {
        return new Product212();
    }
}

class MyFactory2 extends AbstractFactory2 {
    public MyFactory2() {
        System.out.println("实际工厂2");
    }
    @Override
    Product2 newProduct1() {
        return new Product221();
    }

    @Override
    Product2 newProduct2() {
        return new Product222();
    }
}
```

