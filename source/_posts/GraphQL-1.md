---
title: GraphQL （零）初识
date: 2020-03-19 11:47:10
tags:
---

### GraphQL 
GraphQL 是一种描述请求数据方法的语法，通常用于客户端从服务端加载数据。GraphQL 有以下三个主要特征：

它允许客户端指定具体所需的数据。
它让从多个数据源汇总取数据变得更简单。
它使用了类型系统来描述数据。
 
##### 产生原因
GraphQL 是由 Facebook 开发的，用于解决他们巨大、老旧的架构的数据请求问题。但是即使是比 Facebook 小很多的 app，也同样会碰上一些传统 REST API 的局限性问题。

**旧的数据查询方式**：

![old](/image/graphql/GraphQL_oldsearch.png)
不同的数据需要配置不同的查询语句。

**GraphQL的查询方式**：
![new](/image/graphql/GraphQL_newSearch.png)
将多个数据查询整合到一个查询中，通过`GraphQL`转换进行查询。


理论上，一个 GraphQL API 主要由三个部分组成：schema（类型），queries（查询） 以及 resolvers（解析器）



> [我经常听到的 GraphQL 到底是什么？](https://www.jianshu.com/p/3565966eec6e)