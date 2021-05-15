---
title: 算法（一）查询
date: 2020-04-30 14:33:15
tags: [算法]
---

![查询](/image/sf/cz.png)


### 一、基本概念

![查询-基本概念](/image/sf/cz-jbgn.png)



### 二、二分查找 

二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法。但是，折半查找要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。 

 

JAVA代码

```
public static int binarySearch(Integer[] srcArray, int des) {
    //定义初始最小、最大索引
    int start = 0;
    int end = srcArray.length - 1;
    //确保不会出现重复查找，越界
    while (start <= end) {
        //计算出中间索引值
		// java中有三种移位运算符
		// <<:左移运算符，num << 1,相当于num乘以2
		// >>:右移运算符，num >> 1,相当于num除以2
		// >>>:无符号右移，忽略符号位，空位都以0补齐

        int middle = (end + start)>>>1 ;//防止溢出
        if (des == srcArray[middle]) {
            return middle;
        //判断下限
        } else if (des < srcArray[middle]) {
            end = middle - 1;
        //判断上限
        } else {
            start = middle + 1;
        }
    }
    //若没有，则返回-1
    return -1;
}
```
 
### 三、线性索引查找

+	在实际开发中，数据一般都是根据id进行存储的，而我们查询数据通常是依据索引来查找的。
+	索引是把关键字与对应记录相关联的过程
+	线性索引是把索引项集合组织为线性结构，也称之为索引表。
	+	稠密索引
		+	一个记录对应一个索引项
	+	分块索引
		+	分块有序，块间有序，块内无序。
	+	倒排索引
		+	搜索引擎的基础技术；通过属性确定记录，而不是通过记录确定属性
		+	应用：网页SEO。
			+	网页有关键字，被检索到后会被记录下来，然后搜索时过根据关键字来获取对应的网页。


***索引查找（分块索引）***
术语
+	主表。即要查找的序列。
+	索引项。一般我们会将主表分成几个块，每个块建立一个索引，这个索引就叫索引项。
+	索引表。即索引项的集合。

查找方式：
	索引表二分查找，索引项顺序查找

例如：

```
索引表		索引项
10		101	102	103
20		201	202	203
30		301	302	303
（二分查找）	（顺序查找）
		
```

### 四、二叉排序树

特点：删除中效；插入、查询高效

![二叉排序树](/image/sf/ecpxs.png)


***删除操作***
+	叶子结点
	+	直接删除
+	非叶子结点
	+	方式1：左子树作为父节点；右子树作为左节点的右子树的最右孩子
	+	方式2：右子树作为父节点；左子树作为右节点的左子树的最左孩子
	
	
JAVA代码
```
/**
 * @Description 二叉平衡树
 */
public class BSTree<T> {
    private BSNode root;

    public BSTree() {
        this.root = new BSNode();
    }
// 构建
    public void build(List<Integer> list) {
        do {
            BSNode temp = root;
//            1.集合取值
            int nodeData = list.remove(0);

            while (temp.getData() != null) {
//                2.数据小于等于结点,左侧分配；大于,右侧分配
                if(temp.getData() >= nodeData) {
                    if (temp.getLn() == null) {
                        temp.setLn(new BSNode());
                    }
                    temp = temp.getLn();
                } else {
                    if (temp.getRn() == null) {
                        temp.setRn(new BSNode());
                    }
                    temp = temp.getRn();
                }

            }
            temp.setData(nodeData);

        } while (list.size() >0);

    }

    /**
     * @Description 前序遍历：根结点-左子树-右子树
     */
    void preOrder() {
        if (root != null) {
            preOrder(root);
        }
    }
    void preOrder (BSNode node) {
        if (node != null) {
            System.out.println(node.getData());
//          先遍历
            if (node.getLn() != null) {
                preOrder(node.getLn());
            }
            if (node.getRn() != null) {
                preOrder(node.getRn());
            }
        }
    }

    public void search(int data) {
        BSNode temp = root;
        while (temp != null) {
            System.out.print(temp.getData()+"->");
            if (temp.getData().equals(data)) {
               break;
            } else if (temp.getData() >= data) {
                temp = temp.getLn();
            } else {
                temp = temp.getRn();
            }
        }
        System.out.println();
        System.out.println(data);
    }
	
	
	
// 删除
    public void del (int data) {
        BSNode temp = root;
        BSNode pre = null;
        while (temp != null) {
//            System.out.print(temp.getData()+"->");

            if (temp.getData().equals(data)) {
//              删除操作
//                1.叶子结点-直接删除
                if (temp.getRn()==null && temp.getLn()==null) {
                    if (pre.getData() >= data) {
                        pre.setLn(null);
                    } else {
                        pre.setRn(null);
                    }
                } else {
//                     2.非叶子结点
                    BSNode node;
                    if (root.getData().equals(data)) {
                        root = temp.getLn();
                        node = root;
                    } else if (pre.getData() >= data) {
//                        2.1 设置左子树
                        pre.setLn(temp.getLn());
                        node = pre.getLn();
                    } else {
                        pre.setRn(temp.getLn());
                        node = pre.getRn();
                    }

//                        2.2 设置右子树-左子树最右结点的右孩子
                    while (node.getRn() != null) {
                        node = node.getRn();
                    }

                    node.setRn(temp.getRn());
                }

                break;
            } else if (temp.getData() >= data) {
                pre = temp;
                temp = temp.getLn();
            } else {
                pre = temp;
                temp = temp.getRn();
            }

        }
    }
// 添加
    public void insert(int nodeData) {
        BSNode temp = root;
//            1.集合取值

        while (temp.getData() != null) {
//                2.数据小于等于结点,左侧分配；大于,右侧分配
            if(temp.getData() >= nodeData) {
                if (temp.getLn() == null) {
                    temp.setLn(new BSNode());
                }
                temp = temp.getLn();
            } else {
                if (temp.getRn() == null) {
                    temp.setRn(new BSNode());
                }
                temp = temp.getRn();
            }
        }
        temp.setData(nodeData);
    }


    public static void main(String[] args) {
        BSTree bsTree = new BSTree();
        List<Integer> list = new ArrayList<>();
        list.add(62);
        list.add(88);
        list.add(58);
        list.add(47);
        list.add(35);
        list.add(73);
        list.add(51);
        list.add(99);
        list.add(37);
        list.add(93);
        bsTree.build(list);
        bsTree.del(62);
        bsTree.preOrder();
    }


}

class BSNode  {
    private Integer data;
    private BSNode ln;
    private BSNode rn;
	......
}
```

### 五、平衡二叉树（AVL）

![平衡二叉树](/image/sf/phecs.png)

***平衡二叉树转换***

![平衡二叉树转换](/image/sf/phecs-zh.png)


***JAVA代码***

```

```
### 六、多路查找树（B树）

