---
title: Java集合源码分析（一）ArrayList 
date: 2020-02-15 14:49:47
tags: [java]
---

# 总述




# 一、AbstractCollection 
所属Collection类型需要继承的类，该抽象类中包括一些常用的方法以及部分方法的通用实现。

### 定义
抽象集合类，是父类为`Collection`
`public abstract class AbstractCollection<E> implements Collection<E>`

### 抽象方法
```
 public abstract Iterator<E> iterator();
 public abstract int size();
```

### 通用方法【已实现】
```
public boolean isEmpty();
public boolean contains(Object o);
```

#####	contains 包含
```
// 通过迭代器迭代集合，比较引用
public boolean contains(Object o) {
	Iterator<E> it = iterator();
	if (o==null) {
		while (it.hasNext())
			if (it.next()==null)
				return true;
	} else {
		while (it.hasNext())
			if (o.equals(it.next()))
				return true;
	}
	return false;
}
```

#####	toArray 转换为数组

*将现有的数据集合转换为数组返回*
```
    public Object[] toArray() {
        // Estimate size of array; be prepared to see more or fewer elements
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) // fewer elements than expected
				//2. Arrays.copyOf 生成新的数组，不会影响原来的数组，
                return Arrays.copyOf(r, i);
			// 1.保存迭代器中的数据到数组中
            r[i] = it.next();
        }
        return it.hasNext() ? finishToArray(r, it) : r;
    }
```


**
```
	public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    return Arrays.copyOf(r, i);
                } else {
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }
```

> [Java集合框架详解（全）](https://www.cnblogs.com/bingyimeiling/p/10255037.html)