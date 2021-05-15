---
title: 数据结构（七）图
date: 2020-04-27 19:24:00
tags: [算法]
---

![图](/image/sjjg/graph.png)


### 一、基本概念

***边***

![有向边](/image/sjjg/yxb.png)


+	顶点：图中的数据元素
+	边：任意两个顶点之间都可能有关系，顶点之间的逻辑关系可以称之为边
	+	无向边：两顶点之间的边没有方向
	+	有向边：两顶点之间的边有方向，也可称之为弧

***图***

![图种类](/image/sjjg/tzl.png)

+	简单图：不存在有向边且同一边没有重复出现
+	无向完全图：无向图中，任意两个顶点之间都存在边
+	有向完全图：有向图中，任意两个顶点之间都存在方向相反的两条弧，则称之为有向完全图
+	稀疏图：有很少条边或弧的图称之为稀疏图，反之称之为稠密图
+	网：带权的图
	+	权：与图的边或者弧有关的数据称之为权
+	子图：A的顶点与边结构都包含在B中，则称A为B的子图

***顶点与边的关系***

![顶点与边的关系](/image/sjjg/dbbgx.png)

***连通图***

![连通图](/image/sjjg/ltt.png)


### 二、抽象数据类型

![图抽象数据类型](/image/sjjg/tcxsjjg.png)



### 三、存储结构


***邻接矩阵***
图包含了顶点和边，我们可以通过一维数组存储顶点，二维数组存储边。理论上，我们可以通过邻接矩阵来存储图。

如下图所示，无边图的邻接矩阵是一个对称矩阵，即每个节点都没有到自身的边。
![邻接矩阵](/image/sjjg/ljjz.png)

![邻接矩阵有向](/image/sjjg/ljjz-yx.png)


JAVA代码示例: 
+	网
+	结构：逻辑结构是有向带权图，物理结构是邻接矩阵
+	特点：边操作易，顶点操作难

```
public class AMGraph<T> {
//    顶点
    private Object[] nodes;
//    边
    private int[][] edges;
//    访问数组
    private boolean[] visited;
    private final int INIT_SIZE = 10;

    private AMGraph() {
        nodes = new Object[INIT_SIZE];
        edges = new int[INIT_SIZE][INIT_SIZE];
        visited = new boolean[INIT_SIZE];
    }
    private AMGraph (int size) {
        nodes = new Object[size];
        edges = new int[size][size];
        visited = new boolean[size];
    }

    /**
    * @Description 创建图
    * @param nodes 顶点数组
    * @return 图——邻接矩阵
    */
    public static AMGraph create(String[] nodes) {
        AMGraph amGraph;
        if (10 != nodes.length) {
            amGraph = new AMGraph(nodes.length);
        } else {
            amGraph = new AMGraph();
        }

        for (int i=0; i<nodes.length;i++) {
            amGraph.nodes[i] = nodes[i];
        }
        return amGraph;
    }

    /**
    * @Description 添加边
    * @param head 弧头
    * @param tail 弧尾
    * @param wegt 权重
    */
    public void addEdge (int head, int tail, int wegt) throws Exception {
        if (head >= nodes.length && tail >= nodes.length) {
            throw new Exception("结点不存在");
        } else if (head<0 && tail<0) {
            throw new Exception("结点不存在");
        } else {
            edges[head][tail] = wegt;
        }
    }
    public void addEdge (T t, T t1, int wegt) throws Exception {
        addEdge(getIndex(t), getIndex(t1), wegt);
    }

    /**
    * @Description 获取顶点索引
    * @param t 元素
    * @return 数组下标索引
    */
    public int getIndex (T t) {
         for (int i=0; i< nodes.length; i++) {
            if (t.equals(nodes[i])) {
                return i;
            }
        }
         return -1;
    }

//    遍历边，用于遍历所有有向边
    public void edges() {
        for (int i = 0; i<visited.length; i++) {
            visited[i] = false;
        }
        for (int i = 0; i<visited.length; i++) {
            edges(i, 0);
        }

    }
    private void edges(int index, int j) {
//        遍历所有结点判定
        if (j<visited.length ) {
//            if (!visited[j]) {
                if (edges[index][j] != 0) {
                    System.out.println(nodes[index]+"——"+edges[index][j]+"——>"+nodes[j]);
                }
//            }
            if (j<visited.length-1) {
                edges(index, ++j);
            }
        }
        visited[index++] = true;
    }


    public static void main(String[] args) {
        AMGraph<String> amGraph = AMGraph.create(new String[]{"A","B","C","D","E","F","G","H","I"});

        try {
             amGraph.addEdge("A", "B",9);
             amGraph.addEdge("A", "F",5);
             amGraph.addEdge("B", "G",4);
             amGraph.addEdge("B", "C",7);
             amGraph.addEdge("B", "I",6);
             amGraph.addEdge("F", "G",2);
             amGraph.addEdge("F", "E",55);
             amGraph.addEdge("C", "I",43);
             amGraph.addEdge("C", "D",1);
             amGraph.addEdge("I", "D",6);
             amGraph.addEdge("G", "D",9);
             amGraph.addEdge("G", "H",3);
             amGraph.addEdge("H", "D",2);
             amGraph.addEdge("H", "E",22);
             amGraph.addEdge("D", "E",12);

             amGraph.edges();
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {

        }
    }
}

```


> 注：如果是网(边带权重)，那么将数组中表示右边的值1改为边的权重即可。


***邻接表***

对于边数相对顶点数较少的图来说，使用二维数组存储是很浪费的，大量的数组位都是0，不会用于存储有效信息。

所以，我们可以依旧用数组存储顶点，用链表存储边或者弧。这种数组和链表结合的存储方式称之为邻接表。

如下所示，当使用邻接表表示有边图的时候，表示出度；使用逆邻接表，表示入度。
![邻接表](/image/sjjg/ljb.png)

***十字链表***

结合邻接表和逆邻接表就是十字链表，同时表示一个顶点的出度与入度，适合存放有向表。

![十字链表](/image/sjjg/szlb.png)


***邻接多重表***

![邻接多重表](/image/sjjg/ljdcb.png)


***边集数组***

![边集数组](/image/sjjg/bjsz.png)



### 四、遍历

图的遍历是从一个顶点出发访遍图中其余顶点，且每个顶点有且只有一次访问，这个过程称之为`图的遍历`.
+	深度优先遍历（DFS）:从顶点V出发，访问有路径相通的未邻接顶点
+	广度优先遍历（BFS）:从顶点V出发，先访问它的所有相邻顶点，再通过相邻顶点访问其它顶点

如下所示，
![矩阵-图](/image/sjjg/jzt.png)


***JAVA-深度遍历***
```

public class AMGraph<T> {
	
	......
	
    public void dfs() {
//        初始化访问
        for (int i = 0; i<visited.length; i++) {
            visited[i] = false;
        }
        for (int i = 0; i<visited.length; i++) {
            dfs(i, 0);
        }

    }
	// 深度优先算法，持续向下一层遍历，直到无法向下或剩余点都是已遍历的
    private void dfs(int h, int t) {
        visited[h] = true;
        for (; t<visited.length; t++) {
//            顶点未遍历且存在边
            if (!visited[t] && edges[h][t] != 0) {
                //                深度遍历
                System.out.println(nodes[h]+"——"+edges[h][t]+"——>"+nodes[t]);
                dfs(t, 0);
            }
        }
    }

    public static void main(String[] args) {
        AMGraph<String> amGraph = AMGraph.create(new String[]{"A","B","C","D","E","F","G","H","I"});

        try {
             amGraph.addEdge("A", "B",9);
             amGraph.addEdge("A", "F",5);
             amGraph.addEdge("B", "G",4);
             amGraph.addEdge("B", "C",7);
             amGraph.addEdge("B", "I",6);
             amGraph.addEdge("F", "G",2);
             amGraph.addEdge("F", "E",55);
             amGraph.addEdge("C", "I",43);
             amGraph.addEdge("C", "D",1);
             amGraph.addEdge("I", "D",6);
             amGraph.addEdge("G", "D",9);
             amGraph.addEdge("G", "H",3);
             amGraph.addEdge("H", "D",2);
             amGraph.addEdge("H", "E",22);
             amGraph.addEdge("D", "E",12);

             amGraph.dfs();
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {

        }
    }
}
```

遍历结果：
```
A——9——>B
B——7——>C
C——1——>D
D——12——>E
C——43——>I
B——4——>G
G——3——>H
A——5——>F
```

***JAVA-广度遍历***

```
public class AMGraph<T> {
	
	......
//    访问数组
    private boolean[] visited;

//    广度遍历
    public void bfs() {
        for (int i = 0; i<visited.length; i++) {
            visited[i] = false;
        }
        for (int i = 0; i<visited.length; i++) {
            bfs(i);
        }
    }
	//在广度优先算法中，将与顶点邻接的点都保存到队列中，然后先进先出；我这里用列表代替，异曲同工
    private void bfs(int h) {
        visited[h] = true;
        List<Integer> tails = new ArrayList<>();
//        广度优先
        for (int j=0; j<nodes.length; j++) {
            if (!visited[j] && edges[h][j] != 0) {
                visited[j] = true;
                System.out.println(nodes[h]+"——"+edges[h][j]+"——>"+nodes[j]);
                tails.add(j);
            }
        }
        for (int index: tails) {
            bfs(index);
        }
    }

    public static void main(String[] args) {
        AMGraph<String> amGraph = AMGraph.create(new String[]{"A","B","C","D","E","F","G","H","I"});

        try {
             amGraph.addEdge("A", "B",9);
             amGraph.addEdge("A", "F",5);
             amGraph.addEdge("B", "G",4);
             amGraph.addEdge("B", "C",7);
             amGraph.addEdge("B", "I",6);
             amGraph.addEdge("F", "G",2);
             amGraph.addEdge("F", "E",55);
             amGraph.addEdge("C", "I",43);
             amGraph.addEdge("C", "D",1);
             amGraph.addEdge("I", "D",6);
             amGraph.addEdge("G", "D",9);
             amGraph.addEdge("G", "H",3);
             amGraph.addEdge("H", "D",2);
             amGraph.addEdge("H", "E",22);
             amGraph.addEdge("D", "E",12);

             amGraph.bfs();
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {

        }
    }
}

```

遍历结果：
```
A——9——>B
A——5——>F
B——7——>C
B——4——>G
B——6——>I
C——1——>D
D——12——>E
G——3——>H
```


### 五、最小生成树

一个有 n 个结点的连通图的生成树是原图的极小连通子图，且包含原图中的所有 n 个结点，并且有保持图连通的最少的边。

+	最小生成树一般是指将带权图转换为总权最小、涉及所有节点的树结构。
+	最小生成树可以用kruskal（克鲁斯卡尔）算法或prim（普里姆）算法求出。


***prim（普里姆）算法***

![普里姆](/image/sjjg/plm.png)

JAVA代码如下：

```
// 图-邻接矩阵- 网（带权无向图）
public class AMGraphN<T> {
    //    顶点
    private Object[] nodes;
    //    边
    private int[][] edges;
    //    访问数组
    private boolean[] visited;
    private final int INIT_SIZE = 10;

    private AMGraphN() {
        nodes = new Object[INIT_SIZE];
        edges = new int[INIT_SIZE][INIT_SIZE];
        visited = new boolean[INIT_SIZE];
    }
    private AMGraphN (int size) {
        nodes = new Object[size];
        edges = new int[size][size];
        visited = new boolean[size];
    }

    /**
     * @Description 创建图
     */
    public static AMGraphN create(String[] nodes) {
        AMGraphN amGraph;
        if (10 != nodes.length) {
            amGraph = new AMGraphN(nodes.length);
        } else {
            amGraph = new AMGraphN();
        }

        for (int i=0; i<nodes.length;i++) {
            amGraph.nodes[i] = nodes[i];
        }
        return amGraph;
    }

    /**
     * @Description 添加边
     * @param head 弧头
     * @param tail 弧尾
     * @param wegt 权重
     */
    public void addEdge (int head, int tail, int wegt) throws Exception {
        if (head >= nodes.length && tail >= nodes.length) {
            throw new Exception("结点不存在");
        } else if (head<0 && tail<0) {
            throw new Exception("结点不存在");
        } else {
            edges[head][tail] = wegt;
            edges[tail][head] = wegt;
        }
    }
    public void addEdge (T t, T t1, int wegt) throws Exception {
        addEdge(getIndex(t), getIndex(t1), wegt);
    }

    /**
     * @Description 获取顶点索引
     * @param t 元素
     * @return 数组下标索引
     */
    public int getIndex (T t) {
        for (int i=0; i< nodes.length; i++) {
            if (t.equals(nodes[i])) {
                return i;
            }
        }
        return -1;
    }

    //    遍历边
    public void edges() {
        for (int i = 0; i<visited.length; i++) {
            visited[i] = false;
        }
        for (int i = 0; i<visited.length; i++) {
            edges(i, 0);
        }

    }
	   public void plm() {
//        已处理顶点
        List<Integer> completed = new ArrayList<>();
//        1.init
        completed.add(0);
        for (int i = 0; i<visited.length; i++) {
            visited[i] = false;
        }
        visited[0] = true;

        do {
            int min_wdgt = -1;
            int mh=-1, mt=-1;
//            2. 获取到已处理集合路径最短的顶点
            for (int index: completed) {
                for (int j=0; j<nodes.length; j++) {
                    if (!visited[j] && edges[index][j] != 0) {
                        int cost = edges[index][j];
                        if (min_wdgt == -1 || cost < min_wdgt) {
                            min_wdgt = cost;
                            mh = index;
                            mt = j;
                        }
                    }
                }
            }
//            3.输出
            System.out.println(nodes[mh]+"——"+edges[mh][mt]+"——"+nodes[mt]);
            visited[mt]=true;
            completed.add(mt);
        } while (completed.size() < nodes.length);
    }

    public static void main(String[] args) {
        AMGraphN<String> amGraph = AMGraphN.create(new String[]{"V1","V2","V3","V4","V5","V6"});

        try {
            amGraph.addEdge("V1", "V2",6);
            amGraph.addEdge("V1", "V3",1);
            amGraph.addEdge("V1", "V4",5);
            amGraph.addEdge("V2", "V3",5);
            amGraph.addEdge("V3", "V4",5);
            amGraph.addEdge("V2", "V5",3);
            amGraph.addEdge("V3", "V5",6);
            amGraph.addEdge("V3", "V6",4);
            amGraph.addEdge("V4", "V6",2);
            amGraph.addEdge("V5", "V6",6);
            amGraph.plm();
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {

        }
    }


}

```

运算结果：
```
V1——1——V3
V3——4——V6
V6——2——V4
V3——5——V2
V2——3——V5
```

算法优化：
	添加环路判断，在找出最小节点后，需要在加入树之前通过在当前树中可否通过一个节点到达另一个节点

***kruskal（克鲁斯卡尔）算法***

算法思想

1.将图的所有连接线去掉，只剩顶点
2.从图的边集数组中找到权值最小的边，将边的两个顶点连接起来
3.继续寻找权值最小的边，将两个顶点之间连接起来，如果选择的边使得最小生成树出现了环路，则放弃该边，选择权值次小的边
4.直到所有的顶点都被连接在一起并且没有环路，最小生成树就生成了。


### 六、最短路径

![最短路径](/image/sjjg/zdlj.png)
 

 

***迪杰斯特拉算法（Dijkstra）***

特点：
+	`单源最短路径`，求某个顶点到其它顶点的最短路径
+	遵循`路径长度递增的次序`原则
+	有环图，正序逆序皆可

步骤如下：
![迪杰斯特拉](/image/sjjg/djstl.png)

*JAVA-无向图最短路径-迪杰斯特拉*
注：数据参考上图

```
// 无向图
public class AMGraphN<T> {
	...
    /**
    * @Description 最短路径-迪杰斯特拉算法
    * @param source 源点
    * @param target 终点
    * @return
    */
    public void dijkstra(T source, T target) {
//        1.init
        int si = getIndex(source);
        int ti = getIndex(target);
//        已处理顶点
        for (int i = 0; i<visited.length; i++) {
            visited[i] = false;
        }
        visited[si] = true;
		
//      源点到各顶点的路径
        Map<Integer, Integer> map = new HashMap<>();
        map.put(si, 0);

        do {
            int min_cost = -1;
            int min_index = -1;
            for (int i=0; i<nodes.length; i++) {
                if (visited[i]) {
                    int tg = dijkstra(i);
                    if (tg!=-1&&(min_cost == -1 || min_cost > edges[i][tg]+map.get(i))) {
                        min_cost = edges[i][tg]+map.get(i);
                        min_index = tg;
                    }
                }
            }
            map.put(min_index, min_cost);
            visited[min_index] = true;
        } while (map.size() < nodes.length);
        System.out.println(source+"->"+target+": "+map.get(ti));
    }

    /**
    * @Description 迪杰斯特拉-步骤一：获取某个顶点最短路径顶点
    * @param node 顶点
    * @return 顶点索引
    */
    private int dijkstra(int node) {
        int min_cost = -1, min_index = -1;
        for (int j=0; j<nodes.length; j++) {
//            判定成功条件：顶点没有被遍历，不是从起点到起点，存在边
             if (!visited[j] && j != node && edges[node][j] != 0) {
                 if (min_cost == -1 || edges[node][j] < min_cost) {
                     min_cost = edges[node][j];
                     min_index = j;
                 }
             }
        }
        return min_index;
    }

    public static void main(String[] args) {
        AMGraphN<String> amGraph = AMGraphN.create(new String[]{"V0","V1","V2","V3","V4"});

        try {
            amGraph.addEdge("V0", "V1",4);
            amGraph.addEdge("V0", "V2",6);
            amGraph.addEdge("V0", "V3",9);
            amGraph.addEdge("V1", "V2",1);
            amGraph.addEdge("V1", "V4",4);
            amGraph.addEdge("V2", "V4",2);
            amGraph.addEdge("V3", "V2",2);
            amGraph.dijkstra("V1","V4");
        }  catch (Exception e) {
            e.printStackTrace();
        }  finally {

        }
    }
}
```

***弗洛伊德算法***

基本思想如下：

弗洛伊德算法定义了两个二维矩阵：
+	矩阵D记录顶点间的最小路径 
	+	例如D[0][3]= 10，说明顶点0 到 3 的最短路径为10；
+	矩阵P记录顶点间最小路径中的中转点 
	+	例如P[0][3]= 1 说明，0 到 3的最短路径轨迹为：0 -> 1 -> 3。
+	它通过3重循环，k为中转点，v为起点，w为终点，循环比较D0[v][w] 和 D0[v][k] + D0[k][w] 最小值，如果D0[v][k] + D0[k][w] 为更小值，则把D0[v][k] + D0[k][w] 覆盖保存在D1[v][w]中。


***应用***

导航时获取最短路径

### 七、拓扑排序（有向无环图）

![拓扑排序](/image/sjjg/tppx.png)

***实例***

如下所示，我们学习高等代数需要代数基础，而代数则不需要前置知识。

![拓扑排序1](/image/sjjg/tpjgsl.png)

拓扑排序步骤：
+	步骤一：我们会在无环有向图中，先选择没有前驱的顶点（通常这样的点会有多个，选择后驱较少的即可）
+	步骤二：删除它的对应边，然后将顶点加入到拓扑排序中
+	步骤三：重复步骤一

注：由于没有前驱的顶点不只一个，所以拓扑排序一般可以有多个排序

***应用***

解决工程的顺序问题；

关键路径的使用
+	在项目管理中，关键路径是指网络终端元素的元素的序列，该序列具有最长的总工期并决定了整个项目的最短完成时间。
+	关键路径的工期决定了整个项目的工期。任何关键路径上的终端元素的延迟将直接影响项目的预期完成时间（例如在关键路径上没有浮动时间）。
+	一个项目可以有多个，并行的关键路径。另一个总工期比关键路径的总工期略少的一条并行路径被称为次关键路径。
+	最初，关键路径方法只考虑终端元素之间的逻辑依赖关系。关键链方法中增加了资源约束。
+	关键路径方法是由杜邦公司发明的。
 
 

### 八、参考资料

> [最小生成树Prim算法理解](https://blog.csdn.net/yeruby/article/details/38615045)
[第6章 图-迪杰斯特拉算法](https://www.bilibili.com/video/av52374596/)
大话数据结构-第七章