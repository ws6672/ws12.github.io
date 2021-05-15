---
title: 数据结构（四）栈与队列
date: 2020-04-17 10:43:10
tags: [算法]
---


栈是只能在栈顶进行插入与删除的线性表，后进先出
队列只允许一端插入另一端删除，先进先出

### 一、栈

1. 特点

+	栈是只能在表尾进行插入与删除的线性表，后进先出（LIFO）		
+	允许插入与删除的一端称之为栈顶，另一端称之为栈底；不含任何元素的栈称为空栈
+	栈是一个特殊的线性表，具有前驱后继的关系
+	栈的插入操作称为`入栈`、`压栈`；删除操作称为`出栈`

![出入栈](/image/sjjg/crz.png)

2. 抽象数据类型

![抽象数据类型](/image/sjjg/zcxsjlx.png)

3. 顺序栈

当栈以顺序存储结构存储，可以称之为`顺序栈`。

顺序栈的进栈|出栈操作：

![顺序栈](/image/sjjg/sxz.png)

具体代码实现：

抽象父类：

```
public abstract class AbstractStack<E> {
    abstract boolean addTop(E e);
    abstract boolean deleteTop();

    abstract void clear();
    abstract int length();
    public void push(E e) {
        addTop(e);
    }
    public void pop() {
        deleteTop();
    }

}

```


栈：

```
public class Stack<E> extends AbstractStack<E> implements Iterable {

    private Object[] obs;
    private int top;
    private final int INIT_LENGTH = 10;

    public Stack() {
        obs = new Object[INIT_LENGTH];
        top = 0;
    }

    public Stack(int length) {
        obs = new Object[length];
        top = 0;
    }

    @Override
    boolean addTop(E e) {
        if (top < obs.length) {
            obs[top++] = e;
            return true;
        } else {
            return false;
        }
    }

    @Override
    boolean deleteTop() {
       if (top == 0) {
           return false;
       } else {
           obs[--top] = null;
           return true;
       }
    }

    @Override
    void clear() {
        for (int i=0;i<top;i++) {
            obs[i] = null;
        }
        top = 0;
    }

    @Override
    int length() {
        return top;
    }



    @Override
    public Iterator iterator() {
        return new Iterator() {
            private int index = 0;
            @Override
            public boolean hasNext() {
                return index < top?true:false;
            }

            @Override
            public Object next() {
                return obs[index++];
            }
        };
    }


    public static void main(String[] args) {
        Stack<String> stringStack = new Stack<>();
        stringStack.push("aa");
        stringStack.push("bb");
        stringStack.pop();
        stringStack.push("cc");
        Iterator iterator = stringStack.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

4. 链栈

当栈以链接存储结构存储，可以称之为`链栈`。


具体代码实现：

```
public class LinkedStack<E> extends AbstractStack<E> implements Iterable<E>{
    private StackNode top;
    private int size;

    public LinkedStack() {
        this.top = null;
        size = 0;
    }

    public StackNode getTop() {
        return top;
    }

    public void setTop(StackNode top) {
        this.top = top;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }

    @Override
    boolean addTop(E e) {
        try {
            StackNode node = new StackNode();
            node.setE(e);
            node.setNext(top);
            top = node;
            ++size;
        } catch (Exception ex) {
            ex.printStackTrace();
            return false;
        }
        return true;
    }

    @Override
    boolean deleteTop() {
        if (size != 0 ) {
            StackNode temp = top.getNext();
            top = null;
            top = temp;
            --size;
            return true;
        } else {
            return false;
        }

    }

    @Override
    void clear() {
        top = null;
        size = 0;
    }

    @Override
    int length() {
        return size;
    }

    @Override
    public Iterator<E> iterator() {
        return new Iterator<E>() {
            private int index = 0;
            private StackNode temp = top;
            @Override
            public boolean hasNext() {
                if (index < size) {
                    return true;
                } else {
                    return false;
                }
            }

            @Override
            public E next() {
                E e = temp.getE();
                temp = temp.getNext();
                ++index;
                return e;
            }
        };
    }

    class StackNode {
        private StackNode next;
        private E e;

        public StackNode getNext() {
            return next;
        }

        public void setNext(StackNode next) {
            this.next = next;
        }

        public E getE() {
            return e;
        }

        public void setE(E e) {
            this.e = e;
        }
    }

    public static void main(String[] args) {
        LinkedStack<Integer> stack = new LinkedStack<>();
        stack.push(11);
        stack.push(22);
        stack.pop();
        stack.push(33);
        Iterator iterator = stack.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```


5. 栈的应用
+	递归
+	四则运算——波兰表达式

### 二、队列

1. 特点

+	队列只允许一端插入另一端删除，先进先出
+	队列中，允许插入的一端称之为队尾，允许删除的一端称之为队尾

![队列](/image/sjjg/dl.png)

***

2. 抽象数据类型

![队列抽象数据类型](/image/sjjg/dlcxsjlx.png)


抽象队列：
```
public abstract class AbstractQueue<E> {
    private int size;

    void initQueue() {
        size = 0;
    }
    void clear() {
        size = 0;
    }
    int length() {
        return size;
    }

    boolean empty() {
        return size == 0? true:false;
    }

    void enQueue (E e) {
        try {
            addTail(e);
            size++;
        }  catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    protected abstract void addTail(E e) throws Exception;

//    模板方法，有基于数组或链表实现的队列，实现方法不同，将删除的部分算法由子类处理
    E delQueue () {
        try {
            E e = getHead();
            deleteHead();
            size--;
            return e;
        }  catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

//    删除头节点
    protected abstract void deleteHead() throws Exception;
//  获取头节点
    abstract E getHead();
}
```


***

3. 循环队列

![顺序队列](/image/sjjg/sxdl.png)

队列的顺序存储基于数组实现，但是存在严重的不足。如上所示，队列中有五个空间，队头弹出一个而队尾只能插入一个。



基于数组实现的队列：

```
public class Queue<E> extends AbstractQueue<E> implements Iterable<E>{
    private Object[] queue;
    private final int INIT_LENGTH = 10;
    private int head;
    private int tail;
//    初始化
    public Queue() {
        initQueue();
    }
    public Queue(int size) {
        initQueue(size);
    }
    @Override
    void initQueue() {
        super.initQueue();
        head = 0;
        tail = 0;
        queue = new Object[INIT_LENGTH];
    }

    @Override
    protected void addTail(E e) throws Exception {
        if ( tail < queue.length) {
            queue[tail++] = e;
        } else {
            throw new Exception("队列已满");
        }
    }

    @Override
    protected void deleteHead() throws Exception{
        if (tail == 0) {
            throw new Exception("队列为空");
        } else {
            for (int i=head; i<tail-1;i++) {
                queue[i] = queue[i+1];
            }
            queue[--tail] = null;
        }
    }

    @Override
    E getHead() {
        return (E)queue[head];
    }

    void initQueue(int i) {
        super.initQueue();
        queue = new Object[i];
    }

    @Override
    public Iterator<E> iterator() {
        return new Iterator<E>() {
            private int index = 0;
            @Override
            public boolean hasNext() {
               return index<length() ? true:false;
            }

            @Override
            public E next() {
                return (E) queue[index++];
            }
        };
    }

    public static void main(String[] args) {
        Queue<String> queue = new Queue<>();
        queue.enQueue("aa");
        queue.enQueue("bb");
        System.out.println("删除元素："+ queue.delQueue());
        queue.enQueue("cc");
        Iterator iterator = queue.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}

```


![假溢出](/image/sjjg/jyc.png)

当然，在实际使用中，会将后面的元素全部向前挪一位。但是，如果元素过多，程序性能也会受影响。

其实，队头不需要局限在数组的下标0，我们可以将顺序队列转换为循环队列。


基于数组的循环队列：

```
public class CircleQueue<E> extends AbstractQueue<E> implements Iterable{
    private Object[] queue;
    private final int INIT_LENGTH = 10;
    private int head;
    private int tail;
    //    初始化
    public CircleQueue() {
        initQueue();
    }
    public CircleQueue(int size) {
        initQueue(size);
    }
    @Override
    void initQueue() {
        super.initQueue();
        head = 0;
        tail = 0;
        queue = new Object[INIT_LENGTH];
    }

    @Override
    protected void addTail(E e) throws Exception {
        // 元素个数等于数组长度
        if (length() != queue.length) {
            queue[tail] = e;
            // 队尾已是最后一个元素
            if (tail == queue.length-1) {
                tail = 0;
            } else {
                tail++;
            }
        } else {
            throw new Exception("队列已满");
        }
    }

    @Override
    protected void deleteHead() throws Exception{
        if (empty()) {
            throw new Exception("队列为空");
        } else {
            queue[head] = null;
            if (head == queue.length-1) {
                head = 0;
            } else {
                head++;
            }
        }
    }

    @Override
    E getHead() {
        return (E)queue[head];
    }

    void initQueue(int i) {
        super.initQueue();
        queue = new Object[i];
    }

    @Override
    public Iterator<E> iterator() {
        return new Iterator<E>() {
            private int index = head;
//            剩余元素个数
            private int length = length();
            @Override
            public boolean hasNext() {
                return (index-tail != 0 || !empty()) && length!=0 ? true:false;
            }

            @Override
            public E next() {
                E e = (E) queue[index];
                if (index == queue.length -1) {
                    index = 0;
                } else {
                    index++;
                }
                length--;
                return e;
            }
        };
    }

    public static void main(String[] args) {
        CircleQueue<Integer> queue = new CircleQueue<>();
        queue.enQueue(11);
        queue.enQueue(22);
        queue.enQueue(33);
        queue.enQueue(44);
        queue.enQueue(55);
        queue.enQueue(66);
        queue.enQueue(77);
        queue.enQueue(88);
        queue.enQueue(99);
        System.out.println("删除节点："+queue.delQueue());
        System.out.println("删除节点："+queue.delQueue());
        System.out.println("删除节点："+queue.delQueue());
        queue.enQueue(100);
        queue.enQueue(101);
        queue.enQueue(102);
        queue.enQueue(103);
        Iterator iterator = queue.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```
***
4. 队列的链式结构

如下所示，我们会从队头删除元素，从队尾续接元素。

![链式结构](/image/sjjg/lsdl.png)

```
public class LinkedQueue<E> extends AbstractQueue<E> implements Iterable{

    private QueueNode head;
    private QueueNode tail;

    @Override
    protected void addTail(E e) throws Exception {
        QueueNode node = new QueueNode();
        node.setE(e);
        if (head == null) {
            head = node;
        }
        if (tail != null) {
            tail.setNext(node);
        }
        tail = node;
    }

    @Override
    protected void deleteHead() throws Exception {
        if (empty()) {
            throw new Exception("队列为空");
        } else {
            QueueNode node = head.getNext();
            head = null;
            head = node;
        }
    }

    @Override
    E getHead() {
        return head.getE();
    }

    @Override
    public Iterator iterator() {
        return new Iterator() {
            private int index = 0;
            private QueueNode node = head;
            @Override
            public boolean hasNext() {
                return length()-index>0?true:false;
            }

            @Override
            public Object next() {
                E e = node.getE();
                node = node.getNext();
                index++;
                return e;
            }
        };
    }

//    链表节点
    class QueueNode {
        private QueueNode next;
        private E e;

        public QueueNode getNext() {
            return next;
        }

        public void setNext(QueueNode next) {
            this.next = next;
        }

        public E getE() {
            return e;
        }

        public void setE(E e) {
            this.e = e;
        }
    }

    public static void main(String[] args) {
        LinkedQueue<Integer> queue = new LinkedQueue<>();
        queue.enQueue(11);
        queue.enQueue(22);
        queue.enQueue(33);
        queue.enQueue(44);
        System.out.println(queue.delQueue());
        System.out.println(queue.delQueue());
        Iterator iterator = queue.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }

    }
}

```
### 三、参考文章

>	大话数据结构——第4章 栈与队列