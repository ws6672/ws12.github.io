---
title: 数据结构（五）串
date: 2020-04-20 22:02:49
tags: [算法]
---

串是由零个或多个字符组成的有限序列，又名字符串。


### 一、基本概念

***写法***

![串](/image/sjjg/c0.png)

***概念***

+	空格串：只包含空格的串
+	子串：字符串中任意个连续字符组成的子序列可以称之为子串
+	主串：包含子串
+	串的比较：串的比较是通过字符间的编码进行的，即字符在编码集中的对应序号
+	标准ASCLL码：七位二进制，可以表示128个字符
+	拓展ASCLL码：八位二进制，可以表示256个字符
+	Unicode编码：用16位二进制表示的字符，约65万个。

> 注1：Unicode编码的前256位与拓展ASCLL码完成相同，所以Unicode是兼容ASCLL码的

***串比较***

![串比较](/image/sjjg/cbj.png)



### 二、抽象数据类型

![串抽象数据类型](/image/sjjg/ccxsjlx.png)


### 三、模式匹配——暴力匹配 

如下图所示，比较粗暴的模式匹配是通过两个for循环来遍历匹配

![串模式匹配](/image/sjjg/cpspp.png)


### 四、KMP模式匹配算法 

Knuth-Morris-Pratt 字符串查找算法，简称为 “KMP算法”，常用于在一个文本串S内查找一个模式串P 的出现位置，这个算法由Donald Knuth、Vaughan Pratt、James H. Morris三人于1977年联合发表，故取这3人的姓氏命名此算法。

KMP模式匹配算法的重点是找到目标字符串（模式串）中已经匹配过的前缀，并略去其匹配过程，所以核心是利用next数组进行迁越跳转。


***部分匹配表***

+	`前缀`指必须匹配第一个字符，而不包含最后一个字符
+	`后缀`指必须匹配最后一个字符，而不包含第一个字符

以`ABCDABD`为例：
```
"A"的前缀和后缀都为空集，共有元素的长度为0；
"AB"的前缀为[A]，后缀为[B]，共有元素的长度为0；
"ABC"的前缀为[A, AB]，后缀为[BC, C]，共有元素的长度0；
"ABCD"的前缀为[A, AB, ABC]，后缀为[BCD, CD, D]，共有元素的长度为0；
"ABCDA"的前缀为[A, AB, ABC, ABCD]，后缀为[BCDA, CDA, DA, A]，共有元素为"A"，长度为1；
"ABCDAB"的前缀为[A, AB, ABC, ABCD, ABCDA]，后缀为[BCDAB, CDAB, DAB, AB, B]，共有元素为"AB"，长度为2；
"ABCDABD"的前缀为[A, AB, ABC, ABCD, ABCDA, ABCDAB]，后缀为[BCDABD, CDABD, DABD, ABD, BD, D]，共有元素的长度为0。

搜索值：ABCDABD
匹配值（next[j]）：0000120
跃迁方式：j = next[j];
注：j表示子串中匹配进行到的位置
```

***next数组***

next数组的值是代表着字符串的前缀与后缀相同的最大长度,(不能包括自身)。通过部分匹配表得到的`匹配值`就是next数组的值。

![next数组](/image/sjjg/kmpnext.png)


***实例***

```
//    获取next数组,前缀与后缀相互匹配
//    前缀需要包含目标字符串的第一个字符，后缀需要包含目标字符串的最后一个字符
    public int[] next(Str target) {
        int[] next = new int[target.length()];
        char[] cr = target.chars;
        next[0] = 0;
//        i 部分匹配索引；now 匹配个数
        int i = 1, now = 0;

        while (i < cr.length) {
//            匹配每个字符
            if (cr[now] == cr[i]) {
                now++;
                next[i] = now;
                i++;
            } else if (now != 0) {
//                前缀匹配失败，重置前缀
                now = 0;
            } else {
//                匹配失败,递增
                next[i] = 0;
                i++;
            }
        }

        return next;
    }

```


### 五、串的实现

***串的顺序存储***

```
public class Str {
    private char[] chars;
    private int size;
    private final int INIT_SIZE = 10;

    public Str() {
        chars = new char[INIT_SIZE];
        this.size = 0;
    }
    public Str(int size) {
        chars = new char[size];
        this.size = 0;
    }

    public Str(char[] chars) {
        this.chars = chars;
        this.size = chars.length;
    }

    public boolean empty() {
        return size == 0 ?true:false;
    }

    public int length() {
        return size;
    }
    public void clear() {
        if (!empty()) {
            for (int i=0; i<size;i++) {
                chars[i] = '\0';
            }
            size=0;
        }
    }

    // 连接串
    public Str concat(Str newStr) {
        int init_size = chars.length + newStr.length();
        char[] nchars = new char[init_size];
        int i=0;
        for (int j=0;j<chars.length;j++) {
            nchars[i++] = chars[j];
        }
        for (int j=0;j<newStr.length();j++) {
            nchars[i++] = newStr.chars[j];
        }
        Str str = new Str(nchars);
        return str;
    }

    @Override
    protected Str clone() throws CloneNotSupportedException {
        super.clone();
        int init_length = chars.length;
        char[] nChars = new char[init_length];
        for (int i=0; i<init_length; i++) {
            nChars[i] = chars[i];
        }
        return new Str(nChars);
    }

//  串匹配 ——暴力匹配
    public int index(Str target) {
        int index = 0;
        char[] tc = target.chars;
        if (chars.length < tc.length) {
            return -1;
        }
        for (; index<chars.length; index++) {
            int j=0;
//            首字符匹配
            if (chars[index] != tc[j]) {
                continue;
            }

            for (;j<target.chars.length; j++) {
                if (chars[index+j] != tc[j]) {
                    break;
                }
                if (j == tc.length-1) {
                    return index;
                }
            }

        }
        return -1;
    }

//  基于KMP算法的串匹配
    public int kmpIndex(Str target) {
        int[] next = next(target);
        int index = 0, j=0;
        char[] tc = target.chars;
        if (chars.length < tc.length) {
            return -1;
        }
        while (index<chars.length) {
             if (chars[index] == tc[j]) {
                 index++;
                 j++;
             } else if (j != 0) {
//                 匹配失败,部分匹配
                 j = next[j-1];
             } else {
                 index++;
             }
             /*完全匹配*/
             if (j == tc.length) {
                 return index - j;
             }
        }
        return -1;
    }

//    获取next数组,前缀与后缀相互匹配
//    前缀需要包含目标字符串的第一个字符，后缀需要包含目标字符串的最后一个字符
    public int[] next(Str target) {
        int[] next = new int[target.length()];
        char[] cr = target.chars;
        next[0] = 0;
//        i 部分匹配索引；now 匹配个数
        int i = 1, now = 0;

        while (i < cr.length) {
//            匹配每个字符
            if (cr[now] == cr[i]) {
                now++;
                next[i] = now;
                i++;
            } else if (now != 0) {
//                前缀匹配失败，重置前缀
                now = 0;
            } else {
//                匹配失败,递增
                next[i] = 0;
                i++;
            }
        }

        return next;
    }

    public static void main(Str[] args) {
        char[] chars = {'a','b','c','a','b','c','a','b','d'};
        char[] target = {'a','b','c','a','b','d'};
        Str str = new Str(chars);
        System.out.println(str.length());
        System.out.println(str.empty());
        System.out.println(str.kmpIndex(new Str(target)));
        System.out.println(str.concat( new Str(target)));

    }

}

```

***串的链式存储***

通过链表保存字符，具体实现就不展开了。

### 六、参考文章

> 大话数据结构——第5章 串
[KMP(一) 模式匹配算法推导 --《部分匹配表》](https://www.jianshu.com/p/dbf290bffa3b)



