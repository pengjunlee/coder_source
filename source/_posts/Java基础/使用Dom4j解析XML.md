---
title: Java知识点系列之--使用Dom4j解析XML
date: 2020-07-20 13:13:00
updated: 2020-07-20 13:13:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 在Java中如何使用Dom4j解析XML。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img13.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img13.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# 一、Dom4j简介

dom4j是一个Java的XML API，是jdom的升级品，用来读写XML文件的。dom4j是一个十分优秀的JavaXML API，具有性能优异、功能强大和极其易使用的特点，它的性能超过sun公司官方的dom技术，同时它也是一个开放源代码的软件，可以在SourceForge上找到它。在IBM developerWorks上面还可以找到一篇文章，对主流的Java XML API进行的性能、功能和易用性的评测，所以可以知道dom4j无论在哪个方面都是非常出色的。如今可以看到越来越多的Java软件都在使用dom4j来读写XML，特别值得一提的是连Sun的JAXM也在用dom4j。这已经是必须使用的jar包， Hibernate也用它来读写配置文件。 

# 二、文件下载

官网地址：<http://www.dom4j.org/dom4j-1.6.1/>  

# 三、在Java中使用Dom4j解析XML

## 创建Document对象
在DOM4j中，Document对象就代表了整个XML文档，其内部将XML的数据信息以Dom4j树的形式进行储存。

我们可以通过以下三种方式来获得Document对象：  
```Java
        // 读取XML文件,获得document对象
        SAXReader saxReader = new SAXReader();
        Document document1 = saxReader.read(new File("D:\\dom4j\\dom.xml"));
 
        // 解析XML形式的字符串,得到document对象.
        String xmlText = "<response><result>1</result><desc>保存成功</desc></response>";
        Document document2 = DocumentHelper.parseText(xmlText);
        
        //主动创建document对象
        Document document3 =DocumentHelper.createDocument(DocumentHelper.createElement("root").addAttribute("id", "1"));
        System.out.println(document3.getRootElement().attributeValue("id")); //打印结果：1
```

## 操作节点对象
在DOM4j中，使用Element对象来表示Dom4j树中的各个节点，Element对象可以包含有属性、文本内容、命名空间、子节点等内容。

我们通过以对一个名为“dom.xml”的XML文件的操作为例，来对DOM4j节点操作的常用方法进行简要示范，该“dom.xml”的内容如下：  
```Xml
	<?xml version="1.0" encoding="utf-8"?>
	<response>
	<result>1</result>
	<errorDesc>无错误信息</errorDesc>
	<errorCode>E000</errorCode>
	<assignIds>
		<assignId>TEST106608821</assignId>
		<assignId>TEST106608822</assignId>
		<assignId>TEST106608823</assignId>
	</assignIds>
	</response>
```
通过Java代码来操作DOM4j的节点对象，其完整代码如下。
```Java
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.OutputStreamWriter;
	import java.util.Iterator;
	import java.util.List;
	 
	import org.dom4j.Document;
	import org.dom4j.DocumentException;
	import org.dom4j.Element;
	import org.dom4j.io.SAXReader;
	import org.dom4j.io.XMLWriter;
	 
	public class HelloDom4j
	{
	 
	    @SuppressWarnings("unchecked")
	    public static void main(String[] args) throws DocumentException, IOException
	    {
	        SAXReader saxReader = new SAXReader();
	        Document document = saxReader.read("D:\\dom4j\\dom.xml");
	 
	        // 获取Document对象的根节点
	        Element root = document.getRootElement();
	        System.out.println(root.getName());
	 
	        // 取得根节点下的"assignIds"子节点
	        Element assignIds = root.element("assignIds");
	        System.out.println(assignIds.getName());
	 
	        // 取得"assignIds"节点下所有名为"assignId"的子节点，并进行遍历
	        List<Element> elements = assignIds.elements("assignId");
	        Iterator<Element> it = elements.iterator();
	        while (it.hasNext())
	        {
	            Element element = it.next();
	            // 打印节点的文本内容
	            System.out.println(element.getText());
	        }
	 
	        // 在"assignIds"节点下再添加一个"assignId"子节点
	        Element element = assignIds.addElement("assignId");
	 
	        // 设置新节点的文本内容
	        element.setText("TEST106608824");
	 
	        // 删除"assignIds"节点下第一个名为"assignId"的子节点
	        Element firstAssingId = assignIds.element("assignId");
	        assignIds.remove(firstAssingId);
	 
	        // 添加一个CDATA节点
	        Element cddataElement = root.addElement("content");
	        cddataElement.addCDATA("我是CDATA的内容!");
	 
	        // 将Document对象内容保存到XML文件
	        XMLWriter xmlWriter = new XMLWriter(new OutputStreamWriter(new FileOutputStream("D:\\dom4j\\dom4j_dom.xml"), "UTF-8"));
	        xmlWriter.write(document);
	        xmlWriter.close();
	    }
	 
	}
```
程序执行完毕之后，在计算机“**D:\dom4j**”目录下会自动创建一个名为“**dom4j_dom.xml**”的XML文件，其内容如下：  
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<response>
	<result>1</result>
	<errorDesc>无错误信息</errorDesc>
	<errorCode>E000</errorCode>
	<assignIds>
		<assignId>TEST106608822</assignId>
		<assignId>TEST106608823</assignId>
		<assignId>TEST106608824</assignId>
	</assignIds>
	<content><![CDATA[我是CDATA的内容!]]></content>
	</response>
```

## 操作节点对象的属性
我们还是通过对一个XML文件的实际操作为例，来对DOM4j节点对象的属性操作的常用方法进行简要示范。在计算机“**D:\dom4j**”目录下有一个名为“**attribute.xml**”的XML文件，其内容如下： 
```Xml
	<?xml version="1.0" encoding="utf-8"?>
	<Request service='OrderSearchService' lang='zh-CN'>
		<Head name='head'>Hello Dom4j</Head>
		<Body name='body' class="content">
			<OrderSearch name='orderSearch' orderid='2019586321'/>
		</Body>
	</Request>
```
通过Java代码来操作DOM4j节点对象的属性，其完整代码如下。
```Java
	import java.io.FileWriter;
	import java.io.IOException;
	import java.util.Iterator;
	 
	import org.dom4j.Attribute;
	import org.dom4j.Document;
	import org.dom4j.DocumentException;
	import org.dom4j.Element;
	import org.dom4j.io.SAXReader;
	import org.dom4j.io.XMLWriter;
	 
	public class HelloDom4j
	{
	 
	    @SuppressWarnings("unchecked")
	    public static void main(String[] args) throws DocumentException, IOException
	    {
	        SAXReader saxReader = new SAXReader();
	        Document document = saxReader.read("D:\\dom4j\\attribute.xml");
	 
	        // 获取根节点
	        Element root = document.getRootElement();
	        System.out.println(root.getName()); // 打印结果：Request
	 
	        // 获取根节点的"service"属性
	        Attribute attribute = root.attribute("service");
	 
	        // 获取并打印属性的值
	        System.out.println(attribute.getText()); // 打印结果：OrderSearchService
	        System.out.println(attribute.getData()); // 打印结果：OrderSearchService
	        System.out.println(attribute.getValue()); // 打印结果：OrderSearchService
	 
	        // 删除根节点的"lang"属性
	        Attribute lang = root.attribute("lang");
	        System.out.println(lang.getData()); // 打印结果：zh-CN
	        root.remove(attribute);
	 
	        // 为根节点添加"id"属性，并设置值为"request"
	        root.addAttribute("id", "request");
	 
	        // 获取"Head"节点
	        Element head = root.element("Head");
	 
	        // 修改"Head"节点的"name"属性值为"dom4j"
	        Attribute nameAttr = head.attribute("name");
	        System.out.println(nameAttr.getData());// 打印结果：head
	        nameAttr.setValue("dom4j");
	        System.out.println(nameAttr.getData());// 打印结果：dom4j
	 
	        // 获取"OrderSearch"节点
	        Element orderSearch = root.element("Body").element("OrderSearch");
	 
	        // 遍历"OrderSearch"节点的所有属性
	        Iterator<Attribute> it = orderSearch.attributeIterator();
	        while (it.hasNext())
	        {
	            Attribute attr = it.next();
	            System.out.println(attr.getText()); // 打印结果：orderSearch 2019586321
	        }
	 
	        // 将Document对象内容保存到XML文件
	        XMLWriter xmlWriter = new XMLWriter(new FileWriter("D:\\dom4j\\dom4j_attribute.xml"));
	        xmlWriter.write(document);
	        xmlWriter.close();
	 
	    }
	 
	}
```
程序执行完毕之后，在计算机“**D:\dom4j**”目录下会自动创建一个名为“**dom4j_attribute.xml**”的XML文件，其内容如下：
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<Request lang="zh-CN" id="request">
		<Head name="dom4j">Hello Dom4j</Head>
		<Body name="body" class="content">
			<OrderSearch name="orderSearch" orderid="2019586321"/>
		</Body>
	</Request>
```

## 保存Document对象到XML文件
```Java
	// 无需设置字符集编码
    XMLWriter xmlWriter = new XMLWriter(new FileWriter("D:\\dom4j\\dom4j_attribute.xml"));
    xmlWriter.write(document);
    xmlWriter.close();
 
	// 需要设置字符集编码
    OutputFormat format=OutputFormat.createPrettyPrint(); //createPrettyPrint()自动缩进,createCompactFormat()自动压缩
    format.setEncoding("UTF-8");

    XMLWriter xmlWriter = new XMLWriter(new FileWriter("D:\\dom4j\\dom4j_attribute.xml"),format);
    xmlWriter.write(document);
    xmlWriter.close();
```

## 字符串与XML转换
我们继续以前例中的“attribute.xml”文件为例，对其进行转换示范。
```Java
	import org.dom4j.Document;
	import org.dom4j.DocumentException;
	import org.dom4j.DocumentHelper;
	import org.dom4j.Element;
	import org.dom4j.io.SAXReader;
	 
	public class HelloDom4j
	{
	 
	    public static void main(String[] args) throws DocumentException
	    {
	        // 将XML转换为字符串
	        SAXReader saxReader = new SAXReader();
	        Document document = saxReader.read("D:\\dom4j\\attribute.xml");
	 
	        String documentStr = document.asXML();
	        System.out.println(documentStr);
	        /*
	         * 打印结果：
	         * <?xml version="1.0" encoding="utf-8"?>
	         * <Request service="OrderSearchService" lang="zh-CN">
	         * <Head name="head">Hello Dom4j</Head>
	         * <Body name="body" class="content">
	         *      <OrderSearch name="orderSearch" orderid="2019586321"/>
	         * </Body>
	         * </Request>
	         */
	 
	        Element root = document.getRootElement();
	 
	        String rootStr = root.asXML();
	        System.out.println(rootStr);
	        /*
	         * 打印结果：
	         * <Request service="OrderSearchService" lang="zh-CN">
	         * <Head name="head">Hello Dom4j</Head>
	         * <Body name="body" class="content">
	         *      <OrderSearch name="orderSearch" orderid="2019586321"/>
	         * </Body>
	         * </Request>
	         */
	 
	        // 将字符串转换为XML
	        String xmlStr = "<Head name='head'>Hello Dom4j</Head>";
	        Document xmlDoc = DocumentHelper.parseText(xmlStr);
	    }
	 
	}
```