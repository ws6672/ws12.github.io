---
title: 数据结构（三）线性表
date: 2020-04-16 17:14:37
tags: [算法]
---

线性表是零个或者多个元素的有限序列。

### 一、定义

![线性表](/image/sjjg/xxb.png)

抽象数据类型定义：

![线性表数据类型](/image/sjjg/xxbsjlx.png)


### 二、顺序表——（数组）顺序存储结构

1. 数组特点

+	数组是可以再内存中连续存储多个元素的结构，在内存中的分配也是连续的。（逻辑结构是连续的，物理结构是连续的）
+	数组元素通过下标访问，从0开始。

2. 优点

+	按照索引查询元素速度快
+	按照索引遍历数组方便

3. 缺点

+	数组难以扩容
+	添加、删除慢，因为需要移动其它元素
+	只能存储同种类型的数据

4. Java代码示例

```
/**
 * @Description 线性表-顺序表
 */
public class ArrayList<T> implements Iterable<T> {
//    不支持泛型数组，所以用Object数组，因为Object是所有子类的父类，转换不会有问题
    private  Object[] t;
    private final Integer LIST_SIZE = 10;
    private int size;


    public ArrayList() {
        this.t = new Object[LIST_SIZE];
        size = 0;
    }
    public ArrayList(int size) {
        this.t =  new Object[size];
        size = 0;
    }
    /**
    * @Description 判断顺序表是否为空
    * @return true 为空，false为 非空
    */
    public boolean isEmpty () {
        return t.length == 0;
    }
    /**
    * @Description 清空顺序表
    */
    public void clear() {
        for (int i = 0; i<t.length; i++) {
            t[i] = null;
        }
        size = 0;
    }
    /**
    * @Description 通过下标获取元素
    * @param i 数组下标
    * @return  元素
    */
    public T get(int i) {
        try {
            if (i < 0 || i>=t.length) {
                throw new  Exception("索引越界");
            }
            return (T)t[i];
        }  catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public void insert(T e) {
        this.t[size++] = (java.lang.Object) e;
    }
    public T delete (int i) {
        T result = null;
        try {
            if (i < 0 || i>=t.length || i>=this.size) {
                throw new  Exception("索引越界");
            }
            result = (T) t[i];

            for (int j=i; j<this.size -1; j++) {
                t[j] = t[j+1];
            }
            t[--size] = null;

            return result;
        }  catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    public int length() {
        return this.size;
    }


    /**
    * @Description （通过实现 Iterable接口）获取迭代器
    * @return 返回一个迭代器
    */
    @Override
    public Iterator<T> iterator() {
        return new Iterator<T>() {
            private int index = 0;
            @Override
            public boolean hasNext() {
               return index <  size ? true: false;
            }

            @Override
            public T next() {
                return (T)t[index++];
            }
        };
    }

    @Override
    public void forEach(Consumer<? super T> action) {

    }

    @Override
    public Spliterator<T> spliterator() {
        return null;
    }

    /**
    * @Description 测试
    */
    public static void main(String[] args) {
        ArrayList<Integer> arrayList = new ArrayList<Integer>();
        arrayList.insert(1);
        arrayList.insert(5);
        arrayList.insert(3);
        arrayList.delete(0);
        Iterator iterator = arrayList.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

}
```

5. 顺序表的不足之处

顺序表虽然查询、遍历方便。但是添加、删除等操作需要移动其他元素，如果业务是频繁增加、删除的，那么这个数据结构就不太适用了。这时，我们可以使用链接存储的线性表——链表。

6. 为什么不直接实现Iterator接口


在集合源码中，可以看到无论是List还是Set相关的类在迭代的时候，都是通过实现`Iterable`接口实现的，而不是通过`Iterator`接口。因为，直接实现`Iterator`接口，会导致集合使用多个迭代器的时候，共用一个相同的索引，会导致混乱。所以应当通过`Iterable`接口实现迭代器功能。例如：

```
public class MyArrayList<T> implements Iterable<T> {
...
	// 获取迭代器是通过new一个新的迭代器实现的，避免了多个迭代器索引共用
	@Override
    public Iterator<T> iterator() {
        return new Iterator<T>() {
            private int index = 0;
            @Override
            public boolean hasNext() {
               return index < t.length ? true: false;
            }

            @Override
            public T next() {
                return t[index++];
            }
        };
    }
...
}

```

7. 为什么不能使用泛型数组

在JAVA中，如果数据结构需要更加通用化，应当使用泛型。但是，顺序表的底层是数组，而Java中是不能够直接使用泛型数组的。原因如下：
+	数据必须明确知道内部元素的类型，而数组会记住这个类型，并且在插入数据的时候进行类型检测，如果检测不通过，会抛出`java.lang.ArrayStoreException`异常。
+	而泛型是不能作为一个数组类型的。因为泛型是通过`类型擦除`来实现的,所有泛型相关的信息在编译时会丢失，所以泛型数组定义后在编译时无法确认数组类型，所以`Java不允许泛型数组`。


### 二、链表——链接存储结构

1. 特点

+	通过一组任意的存储单元存储数据，这些单元可以是连续的也可以是非连续的，即元素可以存储在任意空置的单元中
+	为了维护元素的关系，ai的元素除了存储数据，还需要存储 a(i+1)元素，即下一个元素的地址
+	存储元素数据信息的域称之为数据域；存储元素地址信息的域称之为指针域

![链表节点](/image/sjjg/lsjdjg.png)

2. 头指针与头节点

![头指针与头节点](/image/sjjg/tzdytjd.png)


3. Java代码示例

```
public class LinkedList<E>  implements Iterable<E> {
    private Node head; //首节点
    private Node tail; //尾节点
    private int size; //总长度

    public LinkedList() {
        head = null;
        tail = head;
    }

    public boolean isEmpty () {
        return length() == 0;
    }
    //  获取链表长度
    public int length() {
        return this.size;
    }
    // 添加元素
    public void insert(E e) {
        Node node = new Node(e);
        if (head == null) {
            head = new Node();
            tail = head;
        }
        tail.setNext(node);
        tail = node;
        ++size;
    }
    // 删除
    public void delete(int i) {
        try {
            if (i < 0  || i>=this.size) {
                throw new  Exception("索引越界");
            }
            Node temp = head.next, pre = head;
            for (int j = 0; j<i; j++) {
                pre = temp;
                temp = temp.next;
            }
            pre.next = temp.next;
            temp = null;
            --size;
        }  catch (Exception e) {
            e.printStackTrace();
        }
    }
    // 修改
    public void update(int i, E ele) {
        try {
            if (i < 0  || i>=this.size) {
                throw new  Exception("索引越界");
            }
            Node temp = head.next;
            for (int j = 0; j<i; j++) {
                temp = temp.next;
            }
            temp.setElement(ele);
        }  catch (Exception e) {
            e.printStackTrace();
        }
    }
    // 查询
    public E search(int i) {
        try {
            if (i < 0  || i>=this.size) {
                throw new  Exception("索引越界");
            }
            Node temp = head.next;
            for (int j = 0; j<i; j++) {
                temp = temp.next;
            }
            return temp.getElement();
        }  catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    // 清空
    public void clear(E e) {
        head = null;
        tail = null;
        size = 0;
    }

    @Override
    public Iterator<E> iterator() {
        return new Iterator<E>() {
            private int index = 0;
            private Node next = head;
            @Override
            public boolean hasNext() {
                return index <  size ? true: false;
            }

            @Override
            public E next() {
                E e = next.getNext().getElement();
                next = next.getNext();
                index++;
                return e;
            }
        };
    }

    /**
    * @Description 链表节点，包含 下个节点，数据 两个属性
    */
    class Node {
        public Node(Node next, E e) {
            this.next = next;
            this.e = e;
        }
        private Node next;
        private E e;

        public E getElement() {
            return e;
        }

        public Node getNext() {
            return next;
        }

        public void setNext(Node next) {
            this.next = next;
        }

        public Node(E e) {
            this.e = e;
            this.next = null;
        }

        public Node() {
        }

        public Node(Node next) {
            this.next = next;
        }

        public void setElement(E e) {
            this.e = e;
        }
    }

    public static void main(String[] args) {
        LinkedList<Integer> linkedList = new LinkedList<>();
        linkedList.insert(11);
        linkedList.insert(2);
        linkedList.insert(333);
        linkedList.insert(4444);
        linkedList.update(0, 1);
        System.out.println("search:" + linkedList.search(linkedList.length()-1));
        linkedList.delete(1);

        Iterator iterator = linkedList.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

4. 链表与顺序表的优缺点

![链表与顺序表的优缺点](/image/sjjg/lbysxb.png)

综上，链表适合多增加多删除的业务；顺序表适合多查询多遍历的业务。


>  静态链表:链表底层用数组表示，逻辑结构仍然是链表；而顺序表是底层是数组，逻辑结构也是数组

 

### 三、循环链表

在循环链表中，总是使尾指针指向头指针，其它的不变。如下图所示：

![循环链表](/image/sjjg/xhlb.png)


基于这个逻辑，可以将`insert`的方法修改为如下的代码：
```
  // 添加元素
    public void insert(E e) {
        Node node = new Node(e);
        if (head == null) {
            head = new Node();
            tail = head;
        }
        tail.setNext(node);
        tail = node;
		tail.setNext(head); // ADD
        ++size;
    }
```



### 四、双向链表

双向链表是在单链表的基础上，再添加一个前驱结点的指针域，如此可以向前遍历了。

双向链表结构如下：

![双向链表](/image/sjjg/sxlb.png)


双向链表部分代码如下：
```
public class DoubleLinkedList<E>  implements Iterable{
    private DoublyNode head; //首节点
    private DoublyNode tail; //尾节点
	...
	class DoublyNode {
        public DoublyNode(DoublyNode next, E e) {
            this.next = next;
            this.e = e;
        }
        private DoublyNode next;
        private E e;
        private DoublyNode pre;

        public E getElement() {
            return e;
        }

        public DoublyNode getNext() {
            return next;
        }

        public void setNext(DoublyNode next) {
            this.next = next;
        }

        public DoublyNode(E e) {
            this.e = e;
            this.next = null;
        }

        public DoublyNode() {
        }

        public DoublyNode(DoublyNode next) {
            this.next = next;
        }

        public void setElement(E e) {
            this.e = e;
        }

        public DoublyNode getPre() {
            return pre;
        }
        public void setPre(DoublyNode pre) {
            this.pre = pre;
        }
    }
 // 添加元素
    public void insert(E e) {
        DoublyNode node = new DoublyNode(e);
        if (head == null) {
            head = new DoublyNode();
            tail = head;
        }
        tail.next = node;
        // 设置新节点的前驱后继
        node.pre = tail;
        node.next = head;
        head.pre = node;
        // 设置新节点
        tail = node;
        ++size;
    }
	...
}

```

### 五、总结

线性表是零个或者多个元素的有限序列，分类如下：

+	顺序表
+	链表 
	+	单链表
	+	静态链表
	+	循环链表
	+	双向链表
	

### 七、参考文章

>	大话数据结构——第3章 线性表