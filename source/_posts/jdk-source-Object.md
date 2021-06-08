---
title: jdk8 源码 —— Object
date: 2021-06-07 22:08:36
tags: [jdk-source]
---

Object 类是类层次结构的根。 每个类都有一个Object作为超类。所有对象，包括数组，都实现了这个类的方法。


1. getClass()
```
public final native Class<?> getClass();
```


返回此Object的运行时类。 返回的Class对象是被表示类的`static synchronized`方法锁定的对象。实际类型是`Class<? extends |X|>`,|X| 是对调用getClass表达式静态类型的擦除（不需要强制转换）。


2. hashCode()

```
public native int hashCode();
```

返回对象的哈希码值，支持此方法是为了方便哈希表的使用：

+	如果根据equals(Object)方法两个对象相等，则对两个对象中的每一个调用hashCode方法必须产生相同的整数结果
+	在 Java 应用程序执行期间，只要在同一个对象上多次调用它， hashCode方法必须始终返回相同的整数，前提是在对象的equals比较中使用的信息没有被修改
+	类Object定义的 hashCode 方法确实为不同的对象返回不同的整数

3. equals(Object obj)

该方法用于指示其他某个对象是否“等于”这个对象，equals方法在非空对象引用上实现等价关系：
```
public boolean equals(Object obj) {
	return (this == obj);
}
```

相关特点如下：

+	自反的：对于任何非空引用值x ， x.equals(x)应该返回true 
+	对称的：对于任何非空引用值x和y ， x.equals(y)应返回true当且仅当y.equals(x)返回true 
+	可传递的：对于任何非空引用值x 、 y和z ，如果x.equals(y)返回true并且y.equals(z)返回true ，那么x.equals(z)应该返回true
+	一致的：对于任何非空引用值x和y ， x.equals(y)多次调用始终返回true或始终返回false ，前提是没有修改对象的equals比较中使用的信息
+	对于任何非空引用值x ， x.equals(null)应返回false

请注意，每当重写此方法时，通常都需要重写hashCode方法，以维护hashCode方法的一般约定，即相等的对象必须具有相等的哈希码。


4. clone() 

用于创建并返回此对象的副本。 “复制”的确切含义可能取决于对象的类别。

```
protected native Object clone() throws CloneNotSupportedException;
```

通常的含义是，对于任何对象x，存在表达式：
```
x.clone() != x
x.clone().getClass() == x.getClass()
x.clone().equals(x)
```

按照惯例，此方法返回的对象应独立于此对象（正在克隆）。 为了实现这种独立性，可能需要在返回之前修改super.clone返回的对象的一个​​或多个字段。 通常，这意味着复制包含被克隆对象的内部“深层结构”的任何可变对象，并将对这些对象的引用替换为对副本的引用。 如果一个类只包含原始字段或对不可变对象的引用，那么通常情况下， super.clone返回的对象中没有字段需要修改。

类Object的方法clone执行特定的克隆操作。 首先，如果该对象的类没有实现接口Cloneable ，则抛出`CloneNotSupportedException` 。 请注意，所有数组都被认为实现了接口Cloneable并且数组类型T[]的clone方法的返回类型是T[] ，其中 T 是任何引用或原始类型。 否则，此方法会创建此对象的类的新实例，并使用此对象的相应字段的内容来初始化其所有字段，就像通过赋值一样； 字段的内容本身不会被克隆。 因此，此方法执行此对象的“浅拷贝”，而不是“深拷贝”操作。

类Object本身不实现接口Cloneable ，因此在类为Object的对象上调用clone方法将导致在运行时抛出异常。


5.  toString()
通常， toString方法返回一个“文本表示”此对象的字符串。 结果应该是一个简洁但信息丰富的表示，易于人们阅读。 建议所有子类都覆盖此方法。
```
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

```

类Object的toString方法返回一个字符串，该字符串由对象是其实例的类的名称、a 符号“@ ”和对象哈希码的无符号十六进制表示组成。

6. notify、notifyAll、wait

相关方法定义如下：
```
public final native void notify();
public final native void notifyAll();
```

6.1 notify方法

```
public final native void wait(long timeout) throws InterruptedException;

public final void wait(long timeout, int nanos) throws InterruptedException {
	if (timeout < 0) {
		throw new IllegalArgumentException("timeout value is negative");
	}

	if (nanos < 0 || nanos > 999999) {
		throw new IllegalArgumentException(
							"nanosecond timeout value out of range");
	}

	if (nanos > 0) {
		timeout++;
	}

	wait(timeout);
}

public final void wait() throws InterruptedException {
	wait(0);
}

nanos以纳秒为单位测量的实时量由下式给出：
	1000000*timeout+nanos
```

notify方法用于唤醒在此对象的监视器上等待的单个线程。如果有任何线程正在等待该对象，则选择其中一个线程被唤醒；选择是任意的并且由实现决定；线程通过调用wait方法之一在对象的监视器上进行等待。被唤醒的线程将无法继续运行，直到当前线程放弃对该对象的锁定。 而被唤醒的线程将以通常的方式与可能正在积极竞争以同步此对象的任何其他线程进行竞争； 例如，被唤醒的线程在成为下一个锁定该对象的线程时没有可靠的特权或劣势。

notify方法只能由作为此对象监视器的所有者的线程调用，线程通过以下三种方式之一成为对象监视器的所有者：

+	通过执行该对象的同步实例方法。
+	通过执行同步对象的synchronized块。
+	对于Class，通过执行该类的同步静态方法。

而notifyAll方法用于唤醒在此对象监视器上等待的所有线程，线程通过调用wait方法之一在对象的监视器上转换为等待状态。


wait方法会导致当前线程等待，直到另一个线程为此对象调用notify()方法或notifyAll()方法，或者指定的时间已经过去。（当前线程必须拥有此对象的监视器）

6.2 wait方法

此方法使当前线程（称为T ）将自己置于此对象的等待集中，然后放弃对此对象的任何和所有同步声明。 线程T出于线程调度目的而被禁用并处于休眠状态，直到发生以下四种情况之一：

+	其他线程中的一个为此对象调用了notify方法，而线程T恰好被任意选择为要唤醒的线程。
+	其他一些线程为此对象调用notifyAll方法。
+	其他一些线程中断（interrupt）线程T 。
+	过了指定的实时时间。 但是，如果timeout为零，则不考虑实时时间，线程只是等待直到收到通知。


当线程T从该对象的等待集中移除，并重新启用线程调度；然后与其他线程竞争在对象上同步的权利； 一旦它获得了对象的控制权，它对对象的所有同步声明都将恢复到之前的状态 —— 也就是说，恢复到调用wait方法时的情况，然后线程T从wait方法的调用中返回。 因此，从wait方法返回时，对象和线程T的同步状态与调用wait方法时完全相同


线程也可以在没有被通知、中断或超时的情况下唤醒，即所谓的虚假唤醒。 虽然这在实践中很少发生，但应用程序必须通过测试应该导致线程被唤醒的条件来防止它，如果条件不满足则继续等待。 换句话说，等待应该总是在循环中发生。例如：
```
 synchronized (obj) {
	   while (<condition does not hold>)
		   obj.wait(timeout);
	   ... // Perform action appropriate to condition
   }
```

此方法只能由作为此对象监视器的所有者的线程调用。如果错误，会抛出以下异常：

```
IllegalArgumentException – 如果超时值为负。
IllegalMonitorStateException – 如果当前线程不是对象监视器的所有者。
InterruptedException – 如果任何线程在当前线程等待通知之前或期间中断了当前线程。 抛出此异常时清除当前线程的中断状态。
```

7. finalize
当垃圾收集器确定不再有对对象的引用时，由垃圾收集器在对象上调用。 子类覆盖finalize方法以处理系统资源或执行其他清理。
```
protected void finalize() throws Throwable { }
	ref.WeakReference , ref.PhantomReference
```

finalize的一般约定是，当 Java™ 虚拟机确定不再有任何方法可以让任何尚未死的线程访问此对象时调用它，除非作为操作的结果由其他一些准备完成的对象或类的完成所采取。 finalize方法可以执行任何操作，包括使该对象再次可供其他线程使用； 然而， finalize的通常目的是在对象被不可撤销地丢弃之前执行清理操作。 例如，表示输入/输出连接的对象的 finalize 方法可能会执行显式 I/O 事务以在对象被永久丢弃之前中断连接。


Java 编程语言不保证哪个线程将调用任何给定对象的finalize方法。 但是，可以保证调用 finalize 的线程在调用 finalize 时不会持有任何用户可见的同步锁。 如果 finalize 方法抛出未捕获的异常，则忽略该异常并终止该对象的终结。

在为一个对象调用了finalize方法之后，在 Java 虚拟机再次确定没有任何方法可以让任何尚未死亡的线程访问该对象之前，不会采取进一步的操作，包括可能的操作由其他准备完成的对象或类，此时该对象可能会被丢弃。

对于任何给定对象，Java 虚拟机永远不会多次调用finalize方法。finalize方法抛出的任何异常都会导致此对象的终止被暂停，但否则会被忽略。
