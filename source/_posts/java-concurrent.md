---
title: Java多线程（一）ThreadLocal
date: 2020-11-04 22:39:15
tags: [并发]
---

### 一、基础

0. 什么是ThreadLoal
ThreadLocal 是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不同的变量值完成操作的场景。

内部结构如下：

![ThreadLocal](/image/java-bf/threadlocal.png)


1. 特点

+	ThreadLoal变量是线程局部变量，同一个ThreadLocal中所包含的对象在不同的线程中有不同的副本。ThreadLocal变量通常被"private static"修饰。当一个线程结束时，它所使用的所有 ThreadLocal 相对的实例副本都可被回收。
+	主要用于线程隔离，使不同线程中的变量不会相互影响。


2. 常用方法

+	get	获取ThreadLocal中当前线程共享变量
+	set 设置ThreadLocal中当前线程共享变量
+	remove 移除ThreadLocal中当前线程共享变量
+	initialValue 初始化对象

3. 实例（SimpleDateFormat在多线程环境中的使用）

在`java.text.SimpleDateFormat`类中，诸如“sdf.parse(dateStr), sdf.format(date) ”的方法传入参数都是用Calendar引用来储存的。在多线程的环境下，静态对象的SimpleDateFormat是共享的，会有多线程的问题。工具类定义如下：

```

import java.text.SimpleDateFormat;
import java.util.HashMap;
import java.util.Map;

/**
 * @author zws
 * @date 2020/11/4 22:51
 * @Description Date工具类，用于多线程。静态 SimpleDateFormat中【calendar】是共享的，多线程使用会有问题，使用ThreadLocal封装进行线程隔离。
 */

public class DateUtil  {
    private static Map<String, ThreadLocal<SimpleDateFormat>> map;
    private final static String[] patterns = new String[]{"yyyy-MM-dd HH:mm:ss","yyyy/MM/dd HH:mm:ss"};
    //    锁对象
    private static Object lock = new Object();

    static {
        map = new HashMap<>();
        ThreadLocal<SimpleDateFormat> tlSdf;
        for (String pattern: patterns) {
            tlSdf = new ThreadLocal<SimpleDateFormat>();
            tlSdf.set(new SimpleDateFormat(pattern));
            map.put(pattern, tlSdf);
        }
    }

    public static SimpleDateFormat getSimpleDateFormat (String pattern) {
        ThreadLocal<SimpleDateFormat> local = map.get(pattern);
        if (local == null)  {
            synchronized (lock) {
                local = new ThreadLocal<SimpleDateFormat>(){
                    @Override
                    protected SimpleDateFormat initialValue() {
                        return new SimpleDateFormat(pattern);
                    }
                };
                map.put(pattern, local);
            }
        }
        return local.get();
    }
	
	// 测试
	public static void main(String[] args) throws ParseException {
        System.out.println(DateUtil.getSimpleDateFormat("yyyy/MM/dd").format(new Date()));
		// 2020/11/04
		
		ThreadLocal<String> threadLocal = new ThreadLocal<>();
//      设置值
        threadLocal.set("test");
//      获取值
        System.out.println(threadLocal.get());
//      移除值
        threadLocal.remove();
    }

}
```

4. 使用场景

ThreadLocal 适用于独立变量，例如多线程链接数据库。为了保证连接稳定，每一个线程都需要有一个独立的Session，这时候就可以把数据用ThreadLocal进行包装，实现局部变量了。

5. ThreadLocal和Synchronized的区别

+	Synchronized是通过线程等待，牺牲时间来解决访问冲突
+	ThreadLocal是通过每个线程单独一份存储空间（线程隔离），牺牲空间来解决冲突

### 二、原理解析


***set方法***
```
public T get() {
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null) {
		ThreadLocalMap.Entry e = map.getEntry(this);
		if (e != null) {
			@SuppressWarnings("unchecked")
			T result = (T)e.value;
			return result;
		}
	}
	// ThreadLocalMap位于线程中，不存在就进行初始化
	return setInitialValue();
}

//初始化方法，存在就添加，不存在就创建
private T setInitialValue() {
	T value = initialValue();
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null)
		map.set(this, value);
	else
		createMap(t, value);
	return value;
}
```

***get方法***

```
 public T get() {
	//获取当前线程
	Thread t = Thread.currentThread();
	// 获取ThreadLocalMap
	ThreadLocalMap map = getMap(t);
	if (map != null) {
		ThreadLocalMap.Entry e = map.getEntry(this);
		if (e != null) {
			@SuppressWarnings("unchecked")
			T result = (T)e.value;
			return result;
		}
	}
	return setInitialValue();
}

// 通过线程获取线程局部变量的
ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}
```

在以上的源码中，可以看到有`ThreadLocalMap`这个类，这是定义在`ThreadLocal`中的静态类，用于存储实际的数据。但是，它是以组合的方式并在`Thread`中，获取时其实是从线程拿的数据。

```
static class ThreadLocalMap {
	...
	// WeakReference是弱引用，当没有被强引用指向的时候，gc的情况下，会被回收
	static class Entry extends WeakReference<ThreadLocal<?>> {
		/** The value associated with this ThreadLocal. */
		Object value;

		Entry(ThreadLocal<?> k, Object v) {
			super(k);
			value = v;
		}
	}
	
	private Entry[] table;
	...
}
```


而`ThreadLocal` 会通过 `ThreadLocalMap` 的方法`getEntry`获取相关对象：
```
private Entry getEntry(ThreadLocal<?> key) {
	// 通过hash值获取索引
	int i = key.threadLocalHashCode & (table.length - 1);
	Entry e = table[i];
	if (e != null && e.get() == key)
		return e;
	else
		return getEntryAfterMiss(key, i, e);
}
```

### 三、参考文章

> [ThreadLocal-面试必问深度解析](https://www.jianshu.com/p/98b68c97df9b)