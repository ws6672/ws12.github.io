---
title: 设计模式（三）行为型模式
date: 2020-04-10 23:33:20
tags: [设计模式]
---

行为型模式分为`类行为模式`和`对象行为模式`。前者采用`继承机制`来在类间分派行为，后者采用`组合或聚合`在对象间分配行为。由于组合关系或聚合关系比继承关系耦合度低，满足“合成复用原则”，所以对象行为模式比类行为模式具有更大的灵活性。

***

|范围\目的|行为型模式|
|:--|:--|
|类模式|模板方法、解释器|
|对象模式|策略<br/>命令<br/>职责链<br/>状态<br/>观察者<br/>中介者<br/>迭代器<br/>访问者<br/>备忘录|0
 

***

# 模板方法模式——定义算法模板

> 模板方法模式通过继承关系，将模板定义在抽象父类中，将不同的部分作为接口由子类实现。

***定义***
模板方法（Template Method）模式的定义如下：定义一个操作中的算法骨架，而将算法的一些步骤延迟到子类中，使得子类可以不改变该算法结构的情况下重定义该算法的某些特定步骤。它是一种类行为型模式。

该模式的主要优点如下。
+	它封装了不变部分，扩展可变部分。它把认为是不变部分的算法封装到父类中实现，而把可变部分算法由子类继承实现，便于子类继续扩展。
+	它在父类中提取了公共的部分代码，便于代码复用。
+	部分方法是由子类实现的，因此子类可以通过扩展方式增加相应的功能，符合开闭原则。

该模式的主要缺点如下。
+	对每个不同的实现都需要定义一个子类，这会导致类的个数增加，系统更加庞大，设计也更加抽象。
+	父类中的抽象方法由子类实现，子类执行的结果会影响父类的结果，这导致一种反向的控制结构，它提高了代码阅读的难度。

***结构***

模板方法模式包含以下主要角色。

(1) 抽象类（Abstract Class）：负责给出一个算法的轮廓和骨架。它由一个模板方法和若干个基本方法构成。这些方法的定义如下。

① 模板方法：定义了算法的骨架，按某种顺序调用其包含的基本方法。

② 基本方法：是整个算法中的一个步骤，包含以下几种类型。
抽象方法：在抽象类中申明，由具体子类实现。
具体方法：在抽象类中已经实现，在具体子类中可以继承或重写它。
钩子方法：在抽象类中已经实现，包括用于判断的逻辑方法和需要子类重写的空方法两种。

(2) 具体子类（Concrete Class）：实现抽象类中所定义的抽象方法和钩子方法，它们是一个顶级逻辑的一个组成步骤。

![算法模板](/image/degin-pattern/mbms.png)


***实例***

```
public class TemplateMethodPattern {
    public static void main(String[] args) {
        AbstractClass abstractClass = new ConcreteClass();
        abstractClass.TemplateMethod();
    }
}

// 抽象模板方法
abstract class AbstractClass {
    //模板方法
    public void TemplateMethod() {
        method1();
        method3();
        method2();
    }
    public void method1() {
        System.out.println("method1");
    }

    public void method2() {
        System.out.println("method2");
    }
    abstract void method3();
}

// 具体模板
class ConcreteClass extends AbstractClass {

    @Override
    void method3() {
        System.out.println("method3:具体模板");
    }
}
```


***

# 策略模式——定义算法接口

> 策略模式通过一个公共的接口定义了一系列算法接口，而具体实现由子类负责，调用者无法了解具体实现，但是需要可以区分不同的策略。

***定义***

策略（Strategy）模式的定义：
+	该模式定义了一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的客户。策略模式属于对象行为模式，它通过对算法进行封装，把使用算法的责任和算法的实现分割开来，并委派给不同的对象对这些算法进行管理。

策略模式的主要优点如下。
+	多重条件语句不易维护，而使用策略模式可以避免使用多重条件语句。
+	策略模式提供了一系列的可供重用的算法族，恰当使用继承可以把算法族的公共代码转移到父类里面，从而避免重复的代码。
+	策略模式可以提供相同行为的不同实现，客户可以根据不同时间或空间要求选择不同的。
+	策略模式提供了对开闭原则的完美支持，可以在不修改原代码的情况下，灵活增加新算法。
+	策略模式把算法的使用放到环境类中，而算法的实现移到具体策略类中，实现了二者的分离。

其主要缺点如下。
+	客户端必须理解所有策略算法的区别，以便适时选择恰当的算法类。
+	策略模式造成很多的策略类。

***结构***

策略模式的主要角色如下。
+	抽象策略（Strategy）类：定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，环境角色使用这个接口调用不同的算法，一般使用接口或抽象类实现。
+	具体策略（Concrete Strategy）类：实现了抽象策略定义的接口，提供具体的算法实现。
+	环境（Context）类：持有一个策略类的引用，最终给客户端调用。

![策略模式](/image/degin-pattern/clms.png)


***实例***

```
public class StrategyPattern {

    public static void main(String[] args) {
        Context context = new Context();
        context.setStrategy(new ConcreteStrategyA());
        context.StrategyMethod1();
    }
}

// 抽象策略类
interface Strategy {
    void StrategyMethod1();
    void StrategyMethod2();
}

// 具体策略类
class ConcreteStrategyA implements Strategy {
    @Override
    public void StrategyMethod1() {
        System.out.println("ConcreteStrategyA:1");
    }

    @Override
    public void StrategyMethod2() {
        System.out.println("ConcreteStrategyA:2");
    }
}
// 环境类
class Context {
    private Strategy strategy;
    public Strategy getStrategy()
    {
        return strategy;
    }
    public void setStrategy(Strategy strategy)
    {
        this.strategy=strategy;
    }
    public void StrategyMethod1()
    {
        strategy.StrategyMethod1();
    }
}
```


***



# 命令模式——责任分割

>  命令模式基于责任分割，将方法的请求与执行分开，即请求与执行的并非同一个对象。


***定义***

命令（Command）模式的定义如下：将一个请求封装为一个对象，使发出请求的责任和执行请求的责任分割开。这样两者之间通过命令对象进行沟通，这样方便将命令对象进行储存、传递、调用、增加与管理。

命令模式的主要优点如下。
+	降低系统的耦合度。命令模式能将调用操作的对象与实现该操作的对象解耦。
+	增加或删除命令非常方便。采用命令模式增加与删除命令不会影响其他类，它满足“开闭原则”，对扩展比较灵活。
+	可以实现宏命令。命令模式可以与组合模式结合，将多个命令装配成一个组合命令，即宏命令。
+	方便实现 Undo 和 Redo 操作。命令模式可以与后面介绍的备忘录模式结合，实现命令的撤销与恢复。

其缺点是：
+	可能产生大量具体命令类。因为计对每一个具体操作都需要设计一个具体命令类，这将增加系统的复杂性。

***结构***

命令模式包含以下主要角色。
+	抽象命令类（Command）角色：声明执行命令的接口，拥有执行命令的抽象方法 execute()。
+	具体命令角色（Concrete    Command）角色：是抽象命令类的具体实现类，它拥有接收者对象，并通过调用接收者的功能来完成命令要执行的操作。
+	实现者/接收者（Receiver）角色：执行命令功能的相关操作，是具体命令对象业务的真正实现者。
+	调用者/请求者（Invoker）角色：是请求的发送者，它通常拥有很多的命令对象，并通过访问命令对象来执行相关请求，它不直接访问接收者。

![命令模式](/image/degin-pattern/mlms.png)


***实例***

```
public class CommandPattern {
    public static void main(String[] args) {
        Command command = new ConcreteCommand();
        Invoker invoker = new Invoker(command);
        invoker.call();
    }
}
// 调用者
class Invoker {
    private Command command;

    public Invoker(Command command) {
        this.command = command;
    }

    public void setCommand(Command command) {
        this.command = command;
    }
//    调用命令
    public void call() {
        System.out.println("调用者开始工作");
        command.execute();
    }
}

// 抽象命令接口
interface Command {
    void execute();
}

// 具体命令类
class ConcreteCommand implements Command {
    private Receiver receiver;

    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }

    public ConcreteCommand() {
        receiver = new Receiver();
    }

    @Override
    public void execute() {
        receiver.action();
    }
}

class Receiver {
    public void action()
    {
        System.out.println("接收者的action()方法被调用...");
    }
}

```


***

# 责任链模式——职责传递
> 责任链模式将方法的请求与执行分开，通过类似链表的结构存储请求处理者，进行职责传递

***定义***

责任链（Chain of Responsibility）模式的定义：为了避免请求发送者与多个请求处理者耦合在一起，将所有请求的处理者通过前一对象记住其下一个对象的引用而连成一条链；当有请求发生时，可将请求沿着这条链传递，直到有对象处理它为止。

> 注意：责任链模式也叫职责链模式。

在责任链模式中，客户只需要将请求发送到责任链上即可，无须关心请求的处理细节和请求的传递过程，所以责任链将请求的发送者和请求的处理者解耦了。

责任链模式是一种对象行为型模式，其主要优点如下。
+	降低了对象之间的耦合度。该模式使得一个对象无须知道到底是哪一个对象处理其请求以及链的结构，发送者和接收者也无须拥有对方的明确信息。
+	增强了系统的可扩展性。可以根据需要增加新的请求处理类，满足开闭原则。
+	增强了给对象指派职责的灵活性。当工作流程发生变化，可以动态地改变链内的成员或者调动它们的次序，也可动态地新增或者删除责任。
+	责任链简化了对象之间的连接。每个对象只需保持一个指向其后继者的引用，不需保持其他所有处理者的引用，这避免了使用众多的 if 或者 if···else 语句。
+	责任分担。每个类只需要处理自己该处理的工作，不该处理的传递给下一个对象完成，明确各类的责任范围，符合类的单一职责原则。

其主要缺点如下。
+	不能保证每个请求一定被处理。由于一个请求没有明确的接收者，所以不能保证它一定会被处理，该请求可能一直传到链的末端都得不到处理。
+	对比较长的职责链，请求的处理可能涉及多个处理对象，系统性能将受到一定影响。
+	职责链建立的合理性要靠客户端来保证，增加了客户端的复杂性，可能会由于职责链的错误设置而导致系统出错，如可能会造成循环调用。

***结构***

职责链模式主要包含以下角色。
+	抽象处理者（Handler）角色：定义一个处理请求的接口，包含抽象处理方法和一个后继连接。
+	具体处理者（Concrete Handler）角色：实现抽象处理者的处理方法，判断能否处理本次请求，如果可以处理请求则处理，否则将该请求转给它的后继者。
+	客户类（Client）角色：创建处理链，并向链头的具体处理者对象提交请求，它不关心处理细节和请求的传递过程。

![责任链模式](/image/degin-pattern/zrlms.png)


***责任链模式与命令模式的区别***

命令模式：多个命令只提交给一个执行该命令的对象
责任链模式：一个请求提交给多个能执行该命令的对象								

***实例***

```
public class ChainOfResponsibilityPattern {

    public static void main(String[] args) {
        Handler handler = new HandlerA();
        handler.setNext(new HandlerB());
        handler.handleRequest("C");
        handler.handleRequest("B");
        handler.handleRequest("A");
    }
}
//抽象处理者角色
abstract class Handler {
    private Handler handler;

    public void setNext(Handler handler) {
        this.handler = handler;
    }

    public Handler getNext() {
        return handler;
    }

    abstract void handleRequest(String request);
    void requestTranslate (String request) {
        if (getNext() != null) {
            getNext().handleRequest(request);
        } else {
            System.out.println("empty");
        }
    }
}

// 具体处理者
class HandlerA extends Handler {
    @Override
    void handleRequest(String request) {
        if (request.equals("A")) {
            System.out.println("HandlerA 处理该请求");
        } else {
            requestTranslate(request);
        }
    }
}

class HandlerB extends Handler {
    @Override
    void handleRequest(String request) {
        if (request.equals("B")) {
            System.out.println("HandlerB 处理该请求");
        } else {
            requestTranslate(request);
        }
    }
}
```


***


# 状态模式——状态切换

> 状态模式允许一个对象在其内部状态改变的时候改变其行为。这个对象看上去就像是改变了它的类一样。
状态持有Context，用于状态切换时改变它的行为；下一个状态由上一个状态决定。


***定义***

状态（State）模式的定义：对有状态的对象，把复杂的“判断逻辑”提取到不同的状态对象中，允许状态对象在其内部状态发生改变时改变其行为。

状态模式是一种对象行为型模式，其主要优点如下。
+	状态模式将与特定状态相关的行为局部化到一个状态中，并且将不同状态的行为分割开来，满足“单一职责原则”。
+	减少对象间的相互依赖。将不同的状态引入独立的对象中会使得状态转换变得更加明确，且减少对象间的相互依赖。
+	有利于程序的扩展。通过定义新的子类很容易地增加新的状态和转换。

状态模式的主要缺点如下。
+	状态模式的使用必然会增加系统的类与对象的个数。
+	状态模式的结构与实现都较为复杂，如果使用不当会导致程序结构和代码的混乱。


***结构***

状态模式包含以下主要角色。
+	环境（Context）角色：也称为上下文，它定义了客户感兴趣的接口，维护一个当前状态，并将与状态相关的操作委托给当前状态对象来处理。
+	抽象状态（State）角色：定义一个接口，用以封装环境对象中的特定状态所对应的行为。
+	具体状态（Concrete    State）角色：实现抽象状态所对应的行为。

![状态模式](/image/degin-pattern/ztms.png)

***实例***
在以下的例子中，环境类有AB两个状态，初始化为A；行为调用时，根据状态执行不同的行为，再切换到下一个状态。如果有多个状态，那么久很适合使用这个模式，但如果状态太多，会增加过多的类。

```
public class StatePatternClient {
    public static void main(String[] args) {
        StateContext stateContext = new StateContext();
        stateContext.handle();
        stateContext.handle();
        stateContext.handle();
    }
}

// 环境类
class StateContext {
    private State state;
    public State getState() {
        return state;
    }

    public void setState(State state) {
        this.state = state;
    }

//    初始化状态
    public StateContext() {
        this.state = new StateA();
    }
    public void handle() {
        state.Handle(this);
    }
}

//抽象状态类
abstract class State
{
    public abstract void Handle(StateContext context);
}

//具体状态类
class StateA extends State {
    public StateA() {
        System.out.println("切换为A");
    }

    @Override
    public void Handle(StateContext context) {
        System.out.println("now-状态A");
        context.setState(new StateB());
    }
}
class StateB extends State {
    public StateB() {
        System.out.println("切换为B");
    }

    @Override
    public void Handle(StateContext context) {
        System.out.println("now-状态B");
        context.setState(new StateA());
    }
}
```

***



# 观察者模式——发布订阅

>  订阅目标后，目标改变后会发布信息提醒所有观察者

***定义***

观察者（Observer）模式的定义：
+	指多个对象间存在一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。这种模式有时又称作发布-订阅模式、模型-视图模式，它是对象行为型模式。

观察者模式是一种对象行为型模式，其主要优点如下:
+	降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系。
+	目标与观察者之间建立了一套触发机制。

它的主要缺点如下：
+	目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用。
+	当观察者对象很多时，通知的发布会花费很多时间，影响程序的效率。

***结构***

观察者模式的主要角色如下。
+	抽象主题（Subject）角色：也叫抽象目标类，它提供了一个用于保存观察者对象的聚集类和增加、删除观察者对象的方法，以及通知所有观察者的抽象方法。
+	具体主题（Concrete    Subject）角色：也叫具体目标类，它实现抽象目标中的通知方法，当具体主题的内部状态发生改变时，通知所有注册过的观察者对象。
+	抽象观察者（Observer）角色：它是一个抽象类或接口，它包含了一个更新自己的抽象方法，当接到具体主题的更改通知时被调用。
+	具体观察者（Concrete Observer）角色：实现抽象观察者中定义的抽象方法，以便在得到目标的更改通知时更新自身的状态。


![观察者模式](/image/degin-pattern/gczms.png)

***实例***

```
public class ObserverPattern {
    public static void main(String[] args) {
        ObserverSubject subject = new ConcreteObserverSubject();
        subject.add(new ObserverA());
        subject.add(new ObserverB());
        subject.notifyObserver();
    }
}

//抽象观察者
interface Observer
{
    void response(); //反应
}

// 具体观察者
class ObserverA implements Observer {
    @Override
    public void response() {
        System.out.println("A response");
    }
}
class ObserverB implements Observer {
    @Override
    public void response() {
        System.out.println("B response");
    }
}


//抽象目标
abstract class ObserverSubject {
    List<Observer> observers = new ArrayList<>();
    void add(Observer observer) {
        observers.add(observer);
    }
    void remove(Observer observer) {
        observers.remove(observer);
    }
    public abstract void notifyObserver(); //通知观察者方法
}

//具体目标
class ConcreteObserverSubject extends ObserverSubject {
    @Override
    public void notifyObserver() {
        for (Observer observer: observers) {
            observer.response();
        }
    }
}

```

***



# 中介者模式——将对象的网状结构转换为星状结构

>  中介者模式通过中介类通讯，是典型的迪米特法则的应用

***定义***

中介者（Mediator）模式的定义：定义一个中介对象来封装一系列对象之间的交互，使原有对象之间的耦合松散，且可以独立地改变它们之间的交互。中介者模式又叫调停模式，它是迪米特法则的典型应用。

中介者模式是一种对象行为型模式，其主要优点如下:
+	降低了对象之间的耦合性，使得对象易于独立地被复用。
+	将对象间的一对多关联转变为一对一的关联，提高系统的灵活性，使得系统易于维护和扩展。

***结构***

中介者模式包含以下主要角色。
+	抽象中介者（Mediator）角色：它是中介者的接口，提供了同事对象注册与转发同事对象信息的抽象方法。
+	具体中介者（ConcreteMediator）角色：实现中介者接口，定义一个 List 来管理同事对象，协调各个同事角色之间的交互关系，因此它依赖于同事角色。
+	抽象同事类（Colleague）角色：定义同事类的接口，保存中介者对象，提供同事对象交互的抽象方法，实现所有相互影响的同事类的公共功能。
+	具体同事类（Concrete Colleague）角色：是抽象同事类的实现者，当需要与其他同事对象交互时，由中介者对象负责后续的交互。

![中介者模式](/image/degin-pattern/zjzms.png)


***实例***

```
public class MediatorPattern{
    public static void main(String[] args)
    {
        Mediator md=new ConcreteMediator();
        Colleague c1,c2;
        c1=new ConcreteColleague1();
        c2=new ConcreteColleague2();
        md.register(c1);
        md.register(c2);
        c1.send();
        System.out.println("-------------");
        c2.send();
    }
}

//抽象中介者
abstract class Mediator {
    public abstract void register(Colleague colleague);
    public abstract void relay(Colleague cl); //转发
}
//具体中介者
class ConcreteMediator extends Mediator
{
    private List<Colleague> colleagues=new ArrayList<Colleague>();
    @Override
    public void register(Colleague colleague)
    {
        if(!colleagues.contains(colleague))
        {
            colleagues.add(colleague);
            colleague.setMedium(this);
        }
    }
    @Override
    public void relay(Colleague cl)
    {
        for(Colleague ob:colleagues)
        {
            if(!ob.equals(cl))
            {
                ((Colleague)ob).receive();
            }
        }
    }
}
//抽象同事类
abstract class Colleague
{
    protected Mediator mediator;
    public void setMedium(Mediator mediator)
    {
        this.mediator=mediator;
    }
    public abstract void receive();
    public abstract void send();
}
//具体同事类
class ConcreteColleague1 extends Colleague
{
    @Override
    public void receive()
    {
        System.out.println("具体同事类1收到请求。");
    }
    @Override
    public void send()
    {
        System.out.println("具体同事类1发出请求。");
        mediator.relay(this); //请中介者转发
    }
}
//具体同事类
class ConcreteColleague2 extends Colleague
{
    @Override
    public void receive()
    {
        System.out.println("具体同事类2收到请求。");
    }
    @Override
    public void send()
    {
        System.out.println("具体同事类2发出请求。");
        mediator.relay(this); //请中介者转发
    }
}
```

***

# 迭代器模式——迭代遍历

>  通过基于数据构建的迭代器遍历数据，隐藏细节。

***定义***

迭代器（Iterator）模式的定义：提供一个对象来顺序访问聚合对象中的一系列数据，而不暴露聚合对象的内部表示。迭代器模式是一种对象行为型模式，其主要优点如下。
+	访问一个聚合对象的内容而无须暴露它的内部表示。
+	遍历任务交由迭代器完成，这简化了聚合类。
+	它支持以不同方式遍历一个聚合，甚至可以自定义迭代器的子类以支持新的遍历。
+	增加新的聚合类和迭代器类都很方便，无须修改原有代码。
+	封装性良好，为遍历不同的聚合结构提供一个统一的接口。

其主要缺点是：增加了类的个数，这在一定程度上增加了系统的复杂性。


***结构***

迭代器模式是通过将聚合对象的遍历行为分离出来，抽象成迭代器类来实现的，其目的是在不暴露聚合对象的内部结构的情况下，让外部代码透明地访问聚合的内部数据。现在我们来分析其基本结构与实现方法。


迭代器模式主要包含以下角色。
+	抽象聚合（Aggregate）角色：定义存储、添加、删除聚合对象以及创建迭代器对象的接口。
+	具体聚合（ConcreteAggregate）角色：实现抽象聚合类，返回一个具体迭代器的实例。
+	抽象迭代器（Iterator）角色：定义访问和遍历聚合元素的接口，通常包含 hasNext()、first()、next() 等方法。
+	具体迭代器（Concretelterator）角色：实现抽象迭代器接口中所定义的方法，完成对聚合对象的遍历，记录遍历的当前位置。


![迭代器模式](/image/degin-pattern/ddqms.png)


***实例***

```

public class IteratorPattern
{
    public static void main(String[] args)
    {
        Aggregate ag=new ConcreteAggregate();
        ag.add("中山大学");
        ag.add("华南理工");
        ag.add("韶关学院");
        System.out.print("聚合的内容有：");
        Iterator it=ag.getIterator();
        while(it.hasNext())
        {
            Object ob=it.next();
            System.out.print(ob.toString()+"\t");
        }
        Object ob=it.first();
        System.out.println("\nFirst："+ob.toString());
    }
}
//抽象聚合
interface Aggregate
{
    public void add(Object obj);
    public void remove(Object obj);
    public Iterator getIterator();
}
//具体聚合
class ConcreteAggregate implements Aggregate
{
    private List<Object> list=new ArrayList<Object>();
    @Override
    public void add(Object obj)
    {
        list.add(obj);
    }
    @Override
    public void remove(Object obj)
    {
        list.remove(obj);
    }
    @Override
    public Iterator getIterator()
    {
        return(new ConcreteIterator(list));
    }
}
//抽象迭代器
interface Iterator
{
    Object first();
    Object next();
    boolean hasNext();
}
//具体迭代器
class ConcreteIterator implements Iterator
{
    private List<Object> list=null;
    private int index=-1;
    public ConcreteIterator(List<Object> list)
    {
        this.list=list;
    }
    @Override
    public boolean hasNext()
    {
        if(index<list.size()-1)
        {
            return true;
        }
        else
        {
            return false;
        }
    }
    @Override
    public Object first()
    {
        index=0;
        Object obj=list.get(index);;
        return obj;
    }
    @Override
    public Object next()
    {
        Object obj=null;
        if(this.hasNext())
        {
            obj=list.get(++index);
        }
        return obj;
    }
}
```

***



# 访问者模式——抽取操作

>  将对数据结构中元素的处理抽取出来

***定义***

访问者（Visitor）模式的定义：将作用于某种数据结构中的各元素的操作分离出来封装成独立的类，使其在不改变数据结构的前提下可以添加作用于这些元素的新的操作，为数据结构中的每个元素提供多种访问方式。它将对数据的操作与数据结构进行分离，是行为类模式中最复杂的一种模式。

访问者（Visitor）模式是一种对象行为型模式，其主要优点如下
+	扩展性好。能够在不修改对象结构中的元素的情况下，为对象结构中的元素添加新的功能。
+	复用性好。可以通过访问者来定义整个对象结构通用的功能，从而提高系统的复用程度。
+	灵活性好。访问者模式将数据结构与作用于结构上的操作解耦，使得操作集合可相对自由地演化而不影响系统的数据结构。
+	符合单一职责原则。访问者模式把相关的行为封装在一起，构成一个访问者，使每一个访问者的功能都比较单一。

访问者（Visitor）模式的主要缺点如下
+	增加新的元素类很困难。在访问者模式中，每增加一个新的元素类，都要在每一个具体访问者类中增加相应的具体操作，这违背了“开闭原则”。
+	破坏封装。访问者模式中具体元素对访问者公布细节，这破坏了对象的封装性。
+	违反了依赖倒置原则。访问者模式依赖了具体类，而没有依赖抽象类。

***结构***

访问者模式包含以下主要角色。
+	抽象访问者（Visitor）角色：定义一个访问具体元素的接口，为每个具体元素类对应一个访问操作 visit() ，该操作中的参数类型标识了被访问的具体元素。
+	具体访问者（ConcreteVisitor）角色：实现抽象访问者角色中声明的各个访问操作，确定访问者访问一个元素时该做什么。
+	抽象元素（Element）角色：声明一个包含接受操作 accept() 的接口，被接受的访问者对象作为 accept() 方法的参数。
+	具体元素（ConcreteElement）角色：实现抽象元素角色提供的 accept() 操作，其方法体通常都是 visitor.visit(this) ，另外具体元素中可能还包含本身业务逻辑的相关操作。
+	对象结构（Object Structure）角色：是一个包含元素角色的容器，提供让访问者对象遍历容器中的所有元素的方法，通常由 List、Set、Map 等聚合类实现。


![访问者模式](/image/degin-pattern/fwzms.png)


***实例***

```
public class VisitorPattern {
    public static void main(String[] args) {
        ObjectStructure os=new ObjectStructure();
        os.add(new ElementA());
        os.add(new ElementB());
        Visitor visitor=new VisitorA();
        os.accept(visitor);
        System.out.println("------------------------");
        visitor=new VisitorB();
        os.accept(visitor);
    }
}

//抽象访问类
interface Visitor {
    void visitor(ElementA a);
    void visitor(ElementB b);
}

//具体访问类
class VisitorA implements Visitor {

    @Override
    public void visitor(ElementA a) {
        System.out.println("A访问："+a.operationA());
    }

    @Override
    public void visitor(ElementB b) {
        System.out.println("A访问："+b.operationB());
    }
}
class VisitorB implements Visitor {
    @Override
    public void visitor(ElementA a) {
        System.out.println("B访问："+a.operationA());
    }

    @Override
    public void visitor(ElementB b) {
        System.out.println("B访问："+b.operationB());
    }
}

//抽象元素类
interface Element
{
    void accept(Visitor visitor);
}
//具体元素类
class ElementA implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visitor(this);
    }
    String operationA() {
        return "A";
    }
}

class ElementB implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visitor(this);
    }
    String operationB() {
        return "B";
    }
}

//对象结构角色-数据结构
class ObjectStructure
{
    private List<Element> list=new ArrayList<Element>();
    public void accept(Visitor visitor)
    {
        Iterator<Element> i=list.iterator();
        while(i.hasNext())
        {
            ((Element) i.next()).accept(visitor);
        }
    }
    public void add(Element element)
    {
        list.add(element);
    }
    public void remove(Element element)
    {
        list.remove(element);
    }
}
```



***



# 备忘录模式——对象快照

>   通过一个拷贝副本保存在管理者角色中实现备忘的功能

***定义***

备忘录（Memento）模式的定义：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便以后当需要时能将该对象恢复到原先保存的状态。该模式又叫快照模式。

备忘录模式是一种对象行为型模式，其主要优点如下。
+	提供了一种可以恢复状态的机制。当用户需要时能够比较方便地将数据恢复到某个历史的状态。
+	实现了内部状态的封装。除了创建它的发起人之外，其他对象都不能够访问这些状态信息。
+	简化了发起人类。发起人不需要管理和保存其内部状态的各个备份，所有状态信息都保存在备忘录中，并由管理者进行管理，这符合单一职责原则。

其主要缺点是：资源消耗大。如果要保存的内部状态信息过多或者特别频繁，将会占用比较大的内存资源。


***结构***

备忘录模式的主要角色如下。
+	发起人（Originator）角色：记录当前时刻的内部状态信息，提供创建备忘录和恢复备忘录数据的功能，实现其他业务功能，它可以访问备忘录里的所有信息。
+	备忘录（Memento）角色：负责存储发起人的内部状态，在需要的时候提供这些内部状态给发起人。
+	管理者（Caretaker）角色：对备忘录进行管理，提供保存与获取备忘录的功能，但其不能对备忘录的内容进行访问与修改。

![备忘录模式](/image/degin-pattern/bwlms.png)



***实例***

```
public class MementoPattern
{
    public static void main(String[] args)
    {
        Originator or=new Originator();
        Caretaker cr=new Caretaker();
        or.setState("S0");
        System.out.println("初始状态:"+or.getState());
        cr.setMemento(or.createMemento()); //保存状态
        or.setState("S1");
        System.out.println("新的状态:"+or.getState());
        or.restoreMemento(cr.getMemento()); //恢复状态
        System.out.println("恢复状态:"+or.getState());
    }
}
//备忘录
class Memento
{
    private String state;
    public Memento(String state)
    {
        this.state=state;
    }
    public void setState(String state)
    {
        this.state=state;
    }
    public String getState()
    {
        return state;
    }
}
//发起人
class Originator
{
    private String state;
    public void setState(String state)
    {
        this.state=state;
    }
    public String getState()
    {
        return state;
    }
    public Memento createMemento()
    {
        return new Memento(state);
    }
    public void restoreMemento(Memento m)
    {
        this.setState(m.getState());
    }
}
//管理者
class Caretaker
{
    private Memento memento;
    public void setMemento(Memento m)
    {
        memento=m;
    }
    public Memento getMemento()
    {
        return memento;
    }
}
```


***



# 解释器模式——封装对象状态

>  在软件开发中，会遇到有些问题多次重复出现，而且有一定的相似性和规律性。如果将它们归纳成一种简单的语言，那么这些问题实例将是该语言的一些句子，这样就可以用“编译原理”中的解释器模式来实现了。


***定义***

解释器（Interpreter）模式的定义：给分析对象定义一个语言，并定义该语言的文法表示，再设计一个解析器来解释语言中的句子。也就是说，用编译语言的方式来分析应用中的实例。这种模式实现了文法表达式处理的接口，该接口解释一个特定的上下文。

这里提到的文法和句子的概念同编译原理中的描述相同，“文法”指语言的语法规则，而“句子”是语言集中的元素。例如，汉语中的句子有很多，“我是中国人”是其中的一个句子，可以用一棵语法树来直观地描述语言中的句子。

解释器模式是一种类行为型模式，其主要优点如下。
+	扩展性好。由于在解释器模式中使用类来表示语言的文法规则，因此可以通过继承等机制来改变或扩展文法。
+	容易实现。在语法树中的每个表达式节点类都是相似的，所以实现其文法较为容易。

解释器模式的主要缺点如下。
+	执行效率较低。解释器模式中通常使用大量的循环和递归调用，当要解释的句子较复杂时，其运行速度很慢，且代码的调试过程也比较麻烦。
+	会引起类膨胀。解释器模式中的每条规则至少需要定义一个类，当包含的文法规则很多时，类的个数将急剧增加，导致系统难以管理与维护。
+	可应用的场景比较少。在软件开发中，需要定义语言文法的应用实例非常少，所以这种模式很少被使用到。

***结构***

解释器模式常用于对简单语言的编译或分析实例中，为了掌握好它的结构与实现，必须先了解编译原理中的“文法、句子、语法树”等相关概念。
1) 文法
文法是用于描述语言的语法结构的形式规则。没有规矩不成方圆，例如，有些人认为完美爱情的准则是“相互吸引、感情专一、任何一方都没有恋爱经历”，虽然最后一条准则较苛刻，但任何事情都要有规则，语言也一样，不管它是机器语言还是自然语言，都有它自己的文法规则。例如，中文中的“句子”的文法如下。
〈句子〉::=〈主语〉〈谓语〉〈宾语〉
〈主语〉::=〈代词〉|〈名词〉
〈谓语〉::=〈动词〉
〈宾语〉::=〈代词〉|〈名词〉
〈代词〉你|我|他
〈名词〉7大学生I筱霞I英语
〈动词〉::=是|学习

注：这里的符号“::=”表示“定义为”的意思，用“〈”和“〉”括住的是非终结符，没有括住的是终结符。
2) 句子
句子是语言的基本单位，是语言集中的一个元素，它由终结符构成，能由“文法”推导出。例如，上述文法可以推出“我是大学生”，所以它是句子。
3) 语法树
语法树是句子结构的一种树型表示，它代表了句子的推导结果，它有利于理解句子语法结构的层次。图 1 所示是“我是大学生”的语法树。


解释器模式包含以下主要角色。
+	抽象表达式（Abstract Expression）角色：定义解释器的接口，约定解释器的解释操作，主要包含解释方法 interpret()。
+	终结符表达式（Terminal    Expression）角色：是抽象表达式的子类，用来实现文法中与终结符相关的操作，文法中的每一个终结符都有一个具体终结表达式与之相对应。
+	非终结符表达式（Nonterminal Expression）角色：也是抽象表达式的子类，用来实现文法中与非终结符相关的操作，文法中的每条规则都对应于一个非终结符表达式。
+	环境（Context）角色：通常包含各个解释器需要的数据或是公共的功能，一般用来传递被所有解释器共享的数据，后面的解释器可以从这里获取这些值。
+	客户端（Client）：主要任务是将需要分析的句子或表达式转换成使用解释器对象描述的抽象语法树，然后调用解释器的解释方法，当然也可以通过环境角色间接访问解释器的解释方法。

![解释器模式](/image/degin-pattern/jsqms.png)

***实例***

```
//抽象表达式类
interface AbstractExpression
{
    public Object interpret(String info);    //解释方法
}
//终结符表达式类
class TerminalExpression implements AbstractExpression
{
    public Object interpret(String info)
    {
        //对终结符表达式的处理
    }
}
//非终结符表达式类
class NonterminalExpression implements AbstractExpression
{
    private AbstractExpression exp1;
    private AbstractExpression exp2;
    public Object interpret(String info)
    {
        //非对终结符表达式的处理
    }
}
//环境类
class Context
{
    private AbstractExpression exp;
    public Context()
    {
        //数据初始化
    }
    public void operation(String info)
    {
        //调用相关表达式类的解释方法
    }
}
```

***

