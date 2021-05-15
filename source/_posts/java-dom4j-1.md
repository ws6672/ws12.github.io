---
title: 【java】dom4j的应用（一）
date: 2019-10-26 16:13:25
tags: [java]
---

我是在工作中用到这个包的，这个包可以对XML格式的数据进行处理，还是蛮实用的。

# 一、dom4j的使用

### 1.什么是 dom4j

> 这是来自百度百科的一段解释。
dom4j是一个Java的XML API，是jdom的升级品，用来读写XML文件的。dom4j是一个十分优秀的JavaXML API，具有性能优异、功能强大和极其易使用的特点，它的性能超过sun公司官方的dom技术，同时它也是一个开放源代码的软件，可以在SourceForge上找到它。在IBM developerWorks上面还可以找到一篇文章，对主流的Java XML API进行的性能、功能和易用性的评测，所以可以知道dom4j无论在哪个方面都是非常出色的。如今可以看到越来越多的Java软件都在使用dom4j来读写XML，特别值得一提的是连Sun的JAXM也在用dom4j。



### 2.引用
如果没有使用maven之类的包管理工具，那么你就需要去[官网](https://dom4j.github.io/)下载包了。下载后放置在lib中即可。

如果使用的是maven，则使用下面的依赖即可
```
<dependency>
	<groupId>org.dom4j</groupId>
	<artifactId>dom4j</artifactId>
	<version>2.1.1</version>
</dependency>
```


# 二、使用

### 1.官网入门实例

*【支持url】读取*

```
	public void test() throws DocumentException {
		SAXReader reader = new SAXReader();
		Document document = reader.read("");
    }
```


*iterator 遍历*
```
	public void test() throws DocumentException {
		Document document = DocumentHelper.parseText("");//字符串转换为XML
		Element root = document.getRootElement();
		Iterator<Element> rows = root.elementIterator();
		String text = document.asXML(); //XML转换为字符串 	

    }
```


### 2.XPath语法

*语法*

|表达式|描述|
|:--|:--|
|nodename|选取此节点的所有子节点。|
|/|从根节点选取。|
|//|从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。|
|.|选取当前节点。|
|..|选取当前节点的父节点。|
|@|选取属性。|


*实例*

|路径表达式|结果|
|:--|:--|
|/bookstore/book[1]|选取属于 bookstore 子元素的第一个 book 元素。|
|/bookstore/book[last()]|选取属于 bookstore 子元素的最后一个 book 元素。|
|/bookstore/book[last()-1]|选取属于 bookstore 子元素的倒数第二个 book 元素。|
|/bookstore/book[position()<3]|选取最前面的两个属于 bookstore 元素的子元素的 book 元素。|
|//title[@lang]|选取所有拥有名为 lang 的属性的 title 元素。|
|//title[@lang='eng']|选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。|
|/bookstore/book[price>35.00]|选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。|
|/bookstore/book[price>35.00]/title|选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。|


### 3.在 dom4j 使用 XPath
使用\<dom4j\>的 XPath 表达式可以分析Document或任何Node树（如Attribute， Element或 ProcessingInstruction）。这样，只需一行代码即可在整个文档中进行复杂的导航。例如

```
	@Test
	public void test1() throws DocumentException {
		Document document = DocumentHelper.parseText("<foo><bar><author name="pt">aa</author><author>bb</author></bar></foo>");
		List<Node> list = document.selectNodes("//foo/bar"); //表示根节点是foo，找到它的子节点bar列表
		Node node = document.selectSingleNode("//foo/bar/author");  //找到当个节点，多个查询拿第一个
		System.out.println(node.getText());
		String name = node.valueOf("@name"); //拿属性
		
		List<Node> list = document.selectNodes("//a/@href"); //拿所有连接
	}
```



### 4.快速循环
使用迭代器虽然可以遍历元素，但每一个迭代器都需要创建对象，而在使用结构特别复杂的文档树是不划算的，所以可以使用快速循环方法。

```
public void treeWalk(Document document) {
    treeWalk(document.getRootElement());
}

public void treeWalk(Element element) {
    for (int i = 0, size = element.nodeCount(); i < size; i++) {
        Node node = element.node(i);
        if (node instanceof Element) {
            treeWalk((Element) node);
        }
        else {
            // do something…
        }
    }
}
```

### 5.创建文档

```
public Document createDocument() {
	Document document = DocumentHelper.createDocument();
	Element root = document.addElement("root");

	Element author1 = root.addElement("author")
		.addAttribute("name", "James")
		.addAttribute("location", "UK")
		.addText("James Strachan");

	Element author2 = root.addElement("author")
		.addAttribute("name", "Bob")
		.addAttribute("location", "US")
		.addText("Bob McWhirter");

	return document;
}
```

### 6.写文档到文件

```
FileWriter out = new FileWriter("foo.xml");
document.write(out);
out.close();
```

### 7.JAXP
> 用于XML处理的Java API（JAXP）使应用程序可以使用独立于特定XML处理器实现的API来解析，转换，验证和查询XML文档。

```
public Document styleDocument(Document document, String stylesheet) throws Exception {

	// load the transformer using JAXP
	TransformerFactory factory = TransformerFactory.newInstance();
	Transformer transformer = factory.newTransformer(new StreamSource(stylesheet));

	// now lets style the given document
	DocumentSource source = new DocumentSource(document);
	DocumentResult result = new DocumentResult();
	transformer.transform(source, result);

	// return the transformed document
	Document transformedDoc = result.getDocument();
	return transformedDoc;
}
```