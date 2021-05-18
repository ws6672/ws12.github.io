---
title: tarjin算法
date: 2021-05-18 11:45:51
tags: [co-simulate]
---

在阅读daccosim运行功能相关源码时，包含了一种求解有向图强连通分量的线性时间的算法，该算法也可以在线性时间内求出无向图的割点与桥。

# 一、Tarjan

1. Tarjan 算法

Tarjan 算法是一种由Robert Tarjan提出的求解有向图强连通分量的线性时间的算法。Tarjan 算法是图论中非常实用 / 常用的算法之一，能解决强连通分量，双连通分量，割点和桥，求最近公共祖先（LCA）等问题。

Tarjan 算法是基于深度优先搜索的算法，用于求解图的连通性问题。Tarjan 算法可以在线性时间内求出无向图的割点与桥，进一步地可以求解无向图的双连通分量；同时，也可以求解有向图的强连通分量、必经点与必经边。

2. 强连通

如果两个顶点可以相互通达，则称两个顶点强连通(strongly connected)。如果有向图G的每两个顶点都强连通，称G是一个强连通图。有向图的极大强连通子图，称为强连通分量(strongly connected components)。

如果非强连通图有向图的极大强连通子图，称为强连通重量(strongly connected components)


3. 相关定义

+	割点：若从图中删除节点 x 以及所有与 x 关联的边之后，图将被分成两个或两个以上的不相连的子图，那么称 x 为图的割点。

+	桥：若从图中删除边 e 之后，图将分裂成两个不相连的子图，那么称 e 为图的桥或割边。

+	时间戳：用来标记图中每个节点在进行深度优先搜索时被访问的时间顺序，定义DFN(u)为节点u搜索的次序编号(时间戳)。

+	追溯值：追溯值用来表示从当前节点作为搜索树的根节点出发，能够访问到的所有节点中，时间戳最小的值，定义Low(u)为u或u的子树能够追溯到的最早的栈中节点的次序号


当DFN(u)=Low(u)时，以u为根的搜索子树上所有节点是一个强连通分量。

4. 求解割点和割边

对于图G(V,E)，根据定义，如果要求解割点则需要三步：
+	BFS跑一遍图，记录下G(V,E)的连通分量为C。
+	枚举所有顶点v_i并删除，再用BFS跑一边删除顶点后的子图，求出子图的连通分量C_i。
+	比较C和C_i，如果C_i < C则说明v_i是割点，反之不是



4. 算法内容

Tarjan算法是基于对图深度优先搜索的算法，每个强连通分量为搜索树中的一棵子树。搜索时，把当前搜索树中未处理的节点加入一个堆栈，回溯时可以判断栈顶到栈中的节点是否为一个强连通分量。相关步骤如下：

+	当首次搜寻到点u时DFN[u]=LOW[u]=time;
+	每当搜寻到一个点，把该点压入栈顶;
+	当u和v有边相连时:
	+	如果v不在栈中（树枝边），DFS(v)，并且LOW[u] = min{LOW(u),LOW(v)};
	+	如果v在栈中（前向边/后向边），此时LOW[u] = min{LOW[u],DFN[v]}
+	当DFN[u]=LOW[u]时，将它以及在它之上的元素弹出栈，此时，弹出栈的结点形成一个强连通重量;
+	持续搜寻，晓得图被遍历结束。
+	因为在这个过程中每个点只被拜访一次，每条边也只被拜访一次，所以Tarjan算法的工夫复杂度是O(n+m).


例如下图所示
![tarjin算法](/image/co-simulation/tarjin.png)

+	从节点1开始DFS，把遍历到的节点加入栈中。搜索到节点u=6时，DFN[6]=LOW[6]，找到了一个强连通分量。退栈到u=v为止，{6}为一个强连通分量。
+	返回节点5，发现DFN[5]=LOW[5]，退栈后{5}为一个强连通分量。
+	返回节点3，继续搜索到节点4，把4加入堆栈。发现节点4向节点1有后向边，节点1还在栈中，所以LOW[4]=1。节点6已经出栈，(4,6)是横叉边，返回3，(3,4)为树枝边，所以LOW[3]=LOW[4]=1。
+	继续回到节点1，最后访问节点2。访问边(2,4)，4还在栈中，所以LOW[2]=DFN[4]=5。返回1后，发现DFN[1]=LOW[1]，把栈中节点全部取出，组成一个连通分量{1,3,4,2}。
+	至此，算法结束。经过该算法，求出了图中全部的三个强连通分量{1,3,4,2},{5},{6}。
+	可以发现，运行Tarjan算法的过程中，每个顶点都被访问了一次，且只进出了一次堆栈，每条边也只被访问了一次，所以该算法的时间复杂度为O(N+M)。

5. 相关代码
以下的代码是daccosim源码中，为求解强联通分量定义的相关类
```
/**
	 * @Author zws
	 * @Description （源码解读）Tarjan 算法：一种由Robert Tarjan提出的求解有向图强连通分量的算法
	 * @Date 10:01 2021/5/18
	 **/
	private static class Tarjan {
		private int V, preCount;
		private int[] low;
		private boolean[] visited;
		private List<Integer>[] graph;
//		强连通分量
		private List<List<Integer>> sccComp;
		private Stack<Integer> stack;

		List<List<Arrow>> getSCComponents(Graph graph) {
			V = graph.nodes().size();
			this.graph = toTarjanGraph(graph);
			low = new int[V];
//			visited 表示已遍历的顶点
			visited = new boolean[V];
			stack = new Stack<>();
			sccComp = new ArrayList<>();
			for (int v = 0; v < V; v++)
				if (!visited[v])
					dfs(v);

			return toArrows(graph, sccComp);
		}

		/**
		 * @Author zws
		 * @Description （源码解读）强连通分量转换为Arrow
		 * @Date 11:25 2021/5/18
		 * @Param graph
		 * @Param sccComp 强连通分量
		 * @return List<List<Arrow>>
		 **/
		private List<List<Arrow>> toArrows(Graph graph, List<List<Integer>> sccComp) {
			List<List<Arrow>> result = new ArrayList<>();
			Map<GraphNode, Integer> nodeIndex = range(0, graph.nodes().size()).boxed().collect(toMap(i -> graph.nodes().get(i), i -> i));
			for (int i = 0; i < sccComp.size(); i++) {
				result.add(new ArrayList<>());
				for (Arrow arrow : graph.arrows()) {
					if (sccComp.get(i).contains(nodeIndex.get(arrow.fromNode())) &&
							sccComp.get(i).contains(nodeIndex.get(arrow.toNode())))
						result.get(i).add(arrow);
				}
			}
			return result.stream().filter(l -> !l.isEmpty()).collect(Collectors.toList());
		}


		/**
		 * @Author zws
		 * @Description （源码解读） Graph --> Tarjan图
		 * @Date 10:12 2021/5/18
		 * @Param graph
		 * @return List<Integer>[]
		 **/
		@SuppressWarnings("unchecked")
		private List<Integer>[] toTarjanGraph(Graph graph) {
			List<Integer>[] result = new List[graph.nodes().size()];
//			<顶点，序列号>
			Map<GraphNode, Integer> nodeIndex = range(0, graph.nodes().size()).boxed().collect(toMap(i -> graph.nodes().get(i), i -> i));
			range(0, graph.nodes().size()).forEach(i -> result[i] = new ArrayList<>());
//			list[i] 表示顶点；list[i].get(0) 表达顶点指向的第一个顶点
			for (Arrow arrow : graph.arrows())
				result[nodeIndex.get(arrow.fromNode())].add(nodeIndex.get(arrow.toNode()));
			return result;
		}

		/**
		 * @Author zws
		 * @Description （源码解读）tarjin图-深度搜索
		 * 时间戳：用来标记图中每个节点在进行深度优先搜索时被访问的时间顺序，定义DFN(u)为节点u搜索的次序编号(时间戳)。
		 * 追溯值：追溯值用来表示从当前节点作为搜索树的根节点出发，能够访问到的所有节点中，时间戳最小的值，定义Low(u)为u或u的子树能够追溯到的最早的栈中节点的次序号
		 * graph[v] 联通的顶点
		 * @Date 10:13 2021/5/18
		 * @Param v
		 **/

		private void dfs(int v) {
			low[v] = preCount++;
			visited[v] = true;
			stack.push(v);
			int min = low[v];
			for (int w : graph[v]) {
				if (!visited[w])
					dfs(w);
				if (low[w] < min)
					min = low[w];
			}
			if (min < low[v]) {
				low[v] = min;
				return;
			}
			List<Integer> component = new ArrayList<>();
			int w;
			do {
				w = stack.pop();
				component.add(w);
				low[w] = V;
			} while (w != v);
			sccComp.add(component);
		}
	}
```

# 二、参考资料

> [Tarjan 算法&模板](https://www.cnblogs.com/shadowland/p/5872257.html)
[tarjan算法](https://baike.baidu.com/item/tarjan%E7%AE%97%E6%B3%95/10687825?fr=aladdin#1)

