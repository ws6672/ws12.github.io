---
title: 数据结构（六）树
date: 2020-04-20 22:21:19
tags: [算法]
---

![树](/image/sjjg/tree.png)
+	`n>0`时，根节点唯一，不可能同时存在多个根节点；
+	`m>0`时，子树的个数没有限制，但是它们一定是互不相交的。


### 一、基本概念


+	结点分类
	+	![结点分类](/image/sjjg/treejdfl.png)

+	结点关系
	+	结点A有两个子树，所以结点A可以称之为结点B和C的双亲结点
	+	同一个双亲的孩子间互称为兄弟结点
	+	而结点A下的所有节点可以称之为结点A的子孙结点

+	深度：树中结点的最大层次可以称之为树的深度或高度。
+	有序树：树的左右结点如果无法互换，而是有层次的，可以称之为有序树。
+	森林：是m课互不相交的树的集合

### 二、树的抽象数据类型

![树的抽象数据类型](/image/sjjg/scxsjlx.png)

### 三、存储结构

+	双亲表示法
	+	![双亲表示法1](/image/sjjg/sqbsf.png)
+	孩子表示法
	+	结构1：所有结点拥有相同的链表，存在空间浪费的情况
	![孩子表示法](/image/sjjg/hzbsf.png)
	+	结构2：链表存储子节点个数，链表不一致，运算时有损耗
	![孩子表示法2](/image/sjjg/hzbsf2.png)
+	孩子兄弟表示法（将树转换为二叉树）
	+	![孩子兄弟表示法](/image/sjjg/hzxdbsf.png)


### 四、二叉树

***定义***

![二叉树](/image/sjjg/ecs.png)

***特点***

![二叉树特点](/image/sjjg/ecstd.png)


***特殊二叉树***

+	斜树：所有结点只有左子树或者只有右子树都可以称之为斜树
+	满二叉树：所有结点都存在左子树和右子树，且所有叶结点都在同一层次
	+	非叶子结点度一定为二
+	![完全二叉树](/image/sjjg/wqecs.png)

***性质***

![二叉树性质](/image/sjjg/ecsxz.png)


***存储结构***

基于二叉树的特殊性，我们可以通过顺序存储结构（数组）和链式存储结构（链表）来存储二叉树的结点。
但如果是一棵右斜树，那么我们 申请2^k-1的空间会造成极大的浪费，所以顺序存储结构一般用于完全二叉树。

为了更好的适用性，实际使用的时候应当考虑使用链表。

***遍历二叉树***

![遍历二叉树](/image/sjjg/ecsbl.png)

![遍历方式](/image/sjjg/pxt.png)

遍历方式如下：
+	前序遍历：根结点-左子树-右子树
+	中序遍历：左子树-根结点-右子树
+	后序遍历：左子树-右子树-根结点
+	层次遍历：从根结点开始，自上而下逐层访问


JAVA实例

```
     /**
     * @Description 前序遍历：根结点-左子树-右子树
     */
    void preOrder() {
        if (root != null) {
            preOrder(root);
        }
    }
    void preOrder (BitNode node) {
        if (node != null) {
            System.out.println(node.getEle());
//          先遍历
            if (node.getLc() != null) {
                preOrder(node.getLc());
            }
            if (node.getRc() != null) {
                preOrder(node.getRc());
            }
        }
    }

    /**
     * @Description 中序遍历：左子树-根结点-右子树
     */
    void midOrder() {
        if (root != null) {
            midOrder(root);
        }
    }
    private void midOrder(BitNode node) {
        if (node != null) {
            if (node.getLc() != null) {
                midOrder(node.getLc());
            }
            System.out.println(node.getEle());
            if (node.getRc() != null) {
                midOrder(node.getRc());
            }
        }
    }


    /**
    * @Description 后序遍历：左子树-右子树-根结点
    */
    void postOrder() {
        if (root != null) {
            postOrder(root);
        }
    }
    private void postOrder(BitNode node) {
        if (node != null) {
            if (node.getLc() != null) {
                postOrder(node.getLc());
            }
            if (node.getRc() != null) {
                postOrder(node.getRc());
            }
            System.out.println(node.getEle());
        }
    }
```

>	注1：遍历方式中，都是先左结点后右结点访问的
注2：前中后序，指的是根结点的访问顺序


### 五、哈夫曼树（最优二叉树） 

哈夫曼树：给定N个权值作为N个叶子结点，构造一棵二叉树，若该树的带权路径长度达到最小，称这样的二叉树为最优二叉树，也称为哈夫曼树(Huffman Tree)。哈夫曼树是带权路径长度最短的树，权值较大的结点离根较近。


以学生成绩为例子，存在如下的代码：
```
if( a < 60 )
	System.out.print("不及格");
else if( a < 70 )
	System.out.print("及格");
else if( a < 90 )
	System.out.print("良好");
else
	System.out.print("优秀");
```

但是在实际中，并不是每种学生的数量都一样，可能不及格的只占5%，而良好的占70%，当数据量较大时该算法效率会偏低。所以，我们可以根据实际将它转换为哈夫曼树。基于类型占学生总数的比，在哈夫曼树中占比（权重）高的接近根节点，即度越小；占比低的树的度越大。

***哈夫曼树的几个概念***
+	结点的路径长度：从根结点到该结点的路径上的连接数。
+	树的路径长度：树中每个叶子结点的路径长度之和。
+	结点带权路径长度：结点的路径长度与结点权值的乘积。
+	树的带权路径长度：WPL(Weighted Path Length)是树中所有叶子结点的带权路径长度之和。
	+	WPL的值越小，说明构造出来的二叉树性能越优，这种最优二叉树又称为哈夫曼树。

***存储结构***
```
struct {
	weight：  权值
	data： 值
	leftChild：左孩子
	rightChild： 右孩子
}
```
***构建哈夫曼树***

对于已知的一组叶子的权值Ｗ 1 ，Ｗ 2...... ，Ｗ n
① 首先把 n 个叶子结点看做 n 棵树（仅有一个结点的二叉树），n棵树组成一个森林。
② 把森林中权值最小和次小的两棵树合并成一棵树(小的放左边，大的放右边)，该树根结点的权值是两棵子树权值之和，这时森林中还有 n-1 棵树。
③ 重复第②步直到森林中只有一棵为止。此树就是哈夫曼树。

![哈夫曼树](/image/sjjg/hfms.png)



***JAVA实例***

```
/**
 * @author zws
 * @date 2020/4/26 17:30
 * @Description 哈夫曼树
 */
public class HfmTree {
    HfmNode parent;
    public static void main(String[] args) {
        List<HfmNode> hfmNodeList = new ArrayList<>();
        hfmNodeList.add(new HfmNode(5,"D"));
        hfmNodeList.add(new HfmNode(15,"C"));
        hfmNodeList.add(new HfmNode(70,"B"));
        hfmNodeList.add(new HfmNode(10,"A"));
        HfmTree tree = new HfmTree();
        tree.build(hfmNodeList);
    }

    private void build(List<HfmNode> list) {
        int i = 0;
        HfmNode ln,rn;
        while (list.size()>1) {
            sort(list);
            ln = list.remove(0);
            rn = list.remove(0);

            parent = new HfmNode(ln.getWgt()+rn.getWgt(), ln.getValue()+rn.getValue());
            parent.setRc(rn);
            parent.setLc(ln);
            list.add(parent);
        }
        preOrder();
    }
    public void sort (List<HfmNode> list) {
        /*	自定义比较器
            如果返回 -1 说明o1 < o2
            如果返回 0  说明o1 = o2
            如果返回 1  说明o1 > o2
        */
        list.sort(new Comparator<HfmNode>() {
            @Override
            public int compare(HfmNode o1, HfmNode o2) {
               int temp = o1.getWgt() - o2.getWgt();
               if (temp == 0) {
                   return temp;
               } else {
                   return temp>0 ? 1:-1;
               }
            }
        });

    }
	// 前序遍历
    void preOrder() {
        if (parent != null) {
            preOrder(parent);
        }
    }
    void preOrder (HfmNode node) {
        if (node != null) {
            System.out.println(node.getWgt()+node.getValue());
//          先遍历
            if (node.getLc() != null) {
                preOrder(node.getLc());
            }
            if (node.getRc() != null) {
                preOrder(node.getRc());
            }
        }
    }
}
//    结点
class HfmNode  {
    private HfmNode lc;
    private HfmNode rc;
    private int wgt;
    private String value;

    public int getWgt() {
        return wgt;
    }

    public void setWgt(int wgt) {
        this.wgt = wgt;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public HfmNode getLc() {
        return lc;
    }

    public void setLc(HfmNode lc) {
        this.lc = lc;
    }

    public HfmNode getRc() {
        return rc;
    }

    public void setRc(HfmNode rc) {
        this.rc = rc;
    }

    public HfmNode() {
    }

    public HfmNode(int wgt, String value) {
        this.wgt = wgt;
        this.value = value;
    }
}
```


 
### 六、参考文章
> 大话数据结构第6章——树
[大话数据结构十六：哈夫曼树(最优二叉树)](https://blog.csdn.net/jim8757/article/details/84497792)