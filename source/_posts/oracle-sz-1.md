---
title: oracle实战（一）跨表空间及跨库查询
date: 2019-11-07 17:10:29
tags: [oracle]
---
 
一个复杂的系统通常会涉及到几个数据库，而跨数据库查询数据也就是家常便饭了。近日在修改一个存储过程，需要查询其它表空间下其它用户的数据，当开始接触时束手束脚，被大佬提点后如同打通了任督二脉。总结如下：
	同库授权，不同库通过数据链接调用。

### 一、跨表空间查询
如果两个表空间在同一个数据库上，那么很容易就可以访问另一个表空间的数据，只需要授权即可。

1. 授权（Bmda）
`grant select any table to Amda`

2. 查询方式（Amda）
```
SELECT *
       FROM Bmda.T_MP_REP;
```

### 二、跨数据库查询
跨数据库查询一般使用的是 DBLINK 。使用DBLINK的步骤如下：
+	创建database link
+	DBLINK_name@Table_name

1.通过pl/sql图形界面创建

![pl/sql](/image/oracle/191107/dblink.png)

2.通过SQL语句创建

```
  create public database link 
    ops2 connect to -- 连接名
      OPS_SZ identified by -- 用户名
        "123" USING  -- 密码
          'ORCL_SZ'; -- 数据库
```

3.使用

```
select * from 
	TTTT@OPSLINK; -- 表名@连接名
```

> 为了方便测试，跨库其实依旧使用的是同一个数据库的，但写法是通用的，望周知。
### 参考链接

[Oracle DBLINK 简单使用](https://www.cnblogs.com/wangyong/p/6354528.html)