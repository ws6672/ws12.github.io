---
title: 面试记录（一）某创——外贸——三轮
date: 2020-08-21 12:24:36
tags: [ms]
---

### 一. 笔试


1. `char 型变量中能不能存储一个中文汉字，为什么`

char 类型可以存储一个中文汉字，因为 Java 中使用的编码是 Unicode（不选择任何特定的编码，直接
使用字符在字符集中的编号，这是统一的唯一方法），一个 char 类型占 2 个字节（16 比特），所以放一个中文是没问题的。

2. `一个类有两个内部类，编译会产生几个 class文件`

3个（一个主类，两个内部类）

3. `简述事务的四个特性是什么`

ACID
原子性 事务是一个完整的操作
一致性 事务内的操作要么一起失败，要么一起成功 
隔离性 多个事务是并发操作，不相互影响
持久性 修改能永久保存

4. `double 类型数据进行运算，存在什么问题，怎么解决？`

double类型的数据做加和操作 时会丢失精度，例如
```
int a = 3;
double b = 0.03;
double c = 0.03;

double d = a + b + c;

System.out.println("first d:" + d);

结果：first d:3.0599999999999996
```

建议进行加操作时，使用 `BigDecimal`类。

5. `如下sql语句str的长度超过10000会怎么样？`

`UPDATE table_name SET agenthkid=agenthkid where id IN(str)`

执行失败，MySQL 的SQL语句长度默认不可以大于1M.


6. `如何格式化数字 “3456789” 成为字符串 “3,456,789.00” 和“003456789”`

使用 java格式化数字 NumberFormat 及DecimalFormat

7. `String、StringBuffer 和 StringBuilder 的区别`

+	String：不可变，线程安全
+	StringBuffer：可变，线程安全
+	StringBuilder：可变，线程安全

8. `JAVA SERVIET API中 forward() 和 redirect()的区别？`

+	请求方不同。redirect:客户端发起的请求 forward:服务端发起的请求;
+	参数传递不同。forward 使用同一个请求，redirect 重新开始一个request
+	直接转发方式（Forward）,间接转发方式（Redirect）实际是两次HTTP请求，


9. `写一个方法实现字符串反转 abc==》cba`

```
String reStr(String str) {
	int rt=str.length()-1;
	StringBuilder sb = new StringBuilder();
	for(;rt>=0; rt--) {
		sb.append(str.charAt(rt));
	}
	return sb.toString();
}
```

10. `表名 student，表中有 A B C 三列，用SQL语句实现：当A列大于B列时选择A列，否则选择B列；当B列大于C列时选择B列，否则选择C列`

使用 SQL语句的 `case`语法：
```
select

case 
	when A>B then A else B end,
case 
	when B>C then B else C end
from D
```


11. `截取网站域名， 实现 http://www.abc.com/he/2t ==>www.abc.com; http://haorestre.uk/teacher/t1.html ==>haorestre.uk`

大概代码如下：
```
String str = "http://www.abc.com/he/2t";
System.out.println(str.split("/")[2]);
```

用“/”切割域名，输出第二个数组


12. 有一个类 class Student(private Long no, private String name), 有两个List<Student>（A,B）;写一个方法取出A中没有在B中出现过的student？（no 相同时 视为同一个 student）

```
方法一暴力法，双重循环，O(n^2)

List get(List<Student> A, List<Student> B) {
	int i,j;
	List<Student> C = new ArrayList<Student> ();
	for(i=0;i<A.size();i++) {
		for(j=0; j<B.size(); j++) {
			if(A.get(i).getNo()==B.get(j).getNO()) {
				break;
			}
			if(j==B.size()-1) {
				C.add(A.get(i));
			}
		}
	}
	return C;
}


方法二效率更高，内层的数组借助map的底层结构转换为红黑树，数据量越大效率越高

List get(List<Student> A, List<Student> B) {
	Map<Long,Student> map = new HashMap<Long,Student>();
	List<Student> C = new ArrayList<Student> ();
	for(Student stu：B) {
		map.put(stu.getNO(), stu);
	}
	for(Student stu：A) {
		if(map.get(stu.getNO()) == null) {
			C.add(stu);
		}		
	}
	
}

```

13. `写一个数据库连接类，并实现数据库查询（思路）`

JDBC
+	加载驱动
+	获取连接
+	设置预语句
+	获取执行结果
+	关闭连接