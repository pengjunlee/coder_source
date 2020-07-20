---
title: Java知识点系列之--字符编码
date: 2020-07-20 13:11:00
updated: 2020-07-20 13:11:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 一起来了解一下Java中的字符编码问题。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
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
> 参考书籍：《深入分析Java Web技术内幕（修订版）》----许令波著 

编码问题一直在困扰着程序开发人员，这在Java中尤其突出，因为Java是跨平台语言，字符在不同平台之间进行传输时经常需要进行编码切换。  

# 为什么要编码？
众所周知，计算机其实是很笨的，它只识别数字`0`和`1`，所以，无论什么内容在计算机内最终都必须以`01串`的形式进行存储，字符也不例外。在计算机中存储信息的最小单元是1个字节，即8个bit，一个字节所能表示的字符个数最多为256个（0~255，二进制11111111=十进制255）。但是，人类要表示的符号太多了，用一个字节来表示显然是远远不够的，必须用更多的字节来表示，比如：用两个字节最多可以表示65535个字符，用4个字节表示的字符数可多达4294967295个。一个字符究竟应该用多少个字节来表示才合适呢？这就是编码所实现的功能，不同的编码使用不同的字节数或数值来表示字符。 

#常用编码格式

## ASCII码
由于计算机是美国人发明的，因此，最早只有128个字母被编码到计算机里，也就是大小写英文字母、数字和一些符号，这个编码表被称为`ASCII（American Standard Code for Information Interchange，美国信息交换标准码）`，采用一个字节的低7位来表示，0~31是控制字符，如换行、回车、删除等，32~127是打印字符，包括那些可以通过键盘输入并且能够显示出来的字符，比如大写字母、小写字母、数字等。

## ISO-8859-1
128个字符显然是不够用的，于是ISO组织在ASCII码基础上又制定了一系列标准来扩展ASCII编码，它们是ISO-8859-1至ISO-8859-15，其中ISO-8859-1涵盖了大多数西欧语言字符，所以应用得最广泛。ISO-8859-1仍然是单字节编码，它总共能表示256个字符。

## GB2312
如果要处理中文的话，一个字节显然还是不够用，所以，中国制定了GB2312编码，用来把中文编进去。GB2312是双字节编码，总的编码范围是A1~F7，其中A1~A9是符号区，总共包含682个符号；B0~F7是汉字区，包含6763个汉字。

## GBK
GBK全称是《汉字内码扩展规范》，是国家技术监督局为Windows 95所制定的新的汉字内码规范，它的出现是为了扩展GB2312，并加入更多的汉字。它的编码范围是8140~FEFE（去掉XX7F），总共有23940个码位，它能表示21003个汉字，它的编码是和GB2312兼容的，也就是说用GB2312编码的汉字可以用GBK来解码，并且不会有乱码。

## UTF-16
说到UTF-16必须提到Unicode（Universal Code 统一码），全世界有上百种语言，日本把日文编到Shift_JIS里，韩国把韩文编到Euc-kr里，如此，假如各国都按照各国各自的标准来设计编码，在某些多语言混用的场景下，就会不可避免地出现编码冲突。为了解决这一问题，ISO提出了Unicode标准，它试图将世界上所有语言都统一到一套编码里。而实际上Unicode仅仅只是一个规范，它使用0~65535之间的数字来表示所有字符，其中0~127这128个数字表示的字符与ASCII完全一样。至于要把0~65535这些数字以怎样的形式转换为01串保存到计算机中，Unicode并未声明其具体的实现方式，于是就出现了`UTF（Unicode Transformation Format）`，包括：**UTF-16**和**UTF-8**。

UTF-16具体定义了Unicode字符在计算机中的存取方法，它用两个字节来表示Unicode的转化格式，采用定长的表示方法，即不论什么字符都用两个字节表示，两个字节就是16个bit，所以叫UTF-16。

## UTF-8
UTF-16统一采用两个字节来表示一个字符，虽然在表示上非常简单、方便，但是也有其缺点，有很大一部分字符用一个字节就可以表示的现在要用两个字节表示，存储空间放大了一倍，在现在的网络带宽还非常有限的情况下，这样会增大网络传输的流量，而且也没有必要。

UTF-8采用一种变长技术，每个编码区域有不同的字码长度。不同的字符可以由1~6个字节组成。

UTF-8有以下编码规则：

- 如果是1个字节，最高位（第8位）为0，则表示这是1个ASCII字符（00~7F）。可见，所有ASCII编码已经是UTF-8了。
- 如果是1个字节，以11开头，则连续的1的个数暗示这个字符的字节数，例如;110xxxxx代表它是双字节UTF-8字符的首字节。
- 如果是1个字节，以10开始，表示它不是首字节，则需要向前查找才能得到当前字符的首字节。

<div align=center>

![字符编码示意图](http://pengjunlee.3vzhuji.net/static/javacore/18.png "字符编码示意图")
<div align=left>

# 在I/O操作中存在的编码

InputStreamReader类可以将I/O操作中读取到的字节流转换为字符流，其部分源代码如下:  
```Java
	package java.io;
	 
	import java.nio.charset.Charset;
	import java.nio.charset.CharsetDecoder;
	import sun.nio.cs.StreamDecoder;
	 
	public class InputStreamReader extends Reader {
	 
	    private final StreamDecoder sd;
	 
	    /**
	     * Creates an InputStreamReader that uses the default charset.
	     *
	     * @param  in   An InputStream
	     */
	    public InputStreamReader(InputStream in) {
		super(in);
	        try {
		    sd = StreamDecoder.forInputStreamReader(in, this, (String)null); // ## check lock object
	        } catch (UnsupportedEncodingException e) {
		    // The default encoding should always be available
		    throw new Error(e);
		}
	    }
	 
	    /**
	     * Creates an InputStreamReader that uses the named charset.
	     *
	     * @param  in
	     *         An InputStream
	     *
	     * @param  charsetName
	     *         The name of a supported
	     *         {@link java.nio.charset.Charset </code>charset<code>}
	     *
	     * @exception  UnsupportedEncodingException
	     *             If the named charset is not supported
	     */
	    public InputStreamReader(InputStream in, String charsetName)
	        throws UnsupportedEncodingException
	    {
		super(in);
		if (charsetName == null)
		    throw new NullPointerException("charsetName");
		sd = StreamDecoder.forInputStreamReader(in, this, charsetName);
	    }
	 
	    /**
	     * Creates an InputStreamReader that uses the given charset. </p>
	     *
	     * @param  in       An InputStream
	     * @param  cs       A charset
	     *
	     * @since 1.4
	     * @spec JSR-51
	     */
	    public InputStreamReader(InputStream in, Charset cs) {
	        super(in);
		if (cs == null)
		    throw new NullPointerException("charset");
		sd = StreamDecoder.forInputStreamReader(in, this, cs);
	    }
	 
	    /**
	     * Creates an InputStreamReader that uses the given charset decoder.  </p>
	     *
	     * @param  in       An InputStream
	     * @param  dec      A charset decoder
	     *
	     * @since 1.4
	     * @spec JSR-51
	     */
	    public InputStreamReader(InputStream in, CharsetDecoder dec) {
	        super(in);
		if (dec == null)
		    throw new NullPointerException("charset decoder");
		sd = StreamDecoder.forInputStreamReader(in, this, dec);
	    }
	}
```
`InputStreamReader`内部包含一个`StreamDecoder`实例引用，对具体字节到字符的解码实现，其实是由StreamDecoder来完成的，在StreamDecoder解码过程中必须由用户指定Charset编码格式，若用户未指定Charset，则将使用本地环境中的默认字符集，如在中文环境中将使用GBK编码。

写的情况也类似，OutputStreamWriter委托StreamEncoder将字符编码成字节。  
```Java
	package java.io;
	 
	import java.nio.charset.Charset;
	import java.nio.charset.CharsetEncoder;
	import sun.nio.cs.StreamEncoder;
	 
	public class OutputStreamWriter extends Writer {
	 
	    private final StreamEncoder se;
	 
	    /**
	     * Creates an OutputStreamWriter that uses the named charset.
	     *
	     * @param  out
	     *         An OutputStream
	     *
	     * @param  charsetName
	     *         The name of a supported
	     *         {@link java.nio.charset.Charset </code>charset<code>}
	     *
	     * @exception  UnsupportedEncodingException
	     *             If the named encoding is not supported
	     */
	    public OutputStreamWriter(OutputStream out, String charsetName)
		throws UnsupportedEncodingException
	    {
		super(out);
		if (charsetName == null)
		    throw new NullPointerException("charsetName");
		se = StreamEncoder.forOutputStreamWriter(out, this, charsetName);
	    }
	 
	    /**
	     * Creates an OutputStreamWriter that uses the default character encoding.
	     *
	     * @param  out  An OutputStream
	     */
	    public OutputStreamWriter(OutputStream out) {
		super(out);
		try {
		    se = StreamEncoder.forOutputStreamWriter(out, this, (String)null);
		} catch (UnsupportedEncodingException e) {
		    throw new Error(e);
	        }
	    }
	 
	    /**
	     * Creates an OutputStreamWriter that uses the given charset. </p>
	     *
	     * @param  out
	     *         An OutputStream
	     *
	     * @param  cs
	     *         A charset
	     *
	     * @since 1.4
	     * @spec JSR-51
	     */
	    public OutputStreamWriter(OutputStream out, Charset cs) {
		super(out);
		if (cs == null)
		    throw new NullPointerException("charset");
		se = StreamEncoder.forOutputStreamWriter(out, this, cs);
	    }
	 
	    /**
	     * Creates an OutputStreamWriter that uses the given charset encoder.  </p>
	     *
	     * @param  out
	     *         An OutputStream
	     *
	     * @param  enc
	     *         A charset encoder
	     *
	     * @since 1.4
	     * @spec JSR-51
	     */
	    public OutputStreamWriter(OutputStream out, CharsetEncoder enc) {
		super(out);
		if (enc == null)
		    throw new NullPointerException("charset encoder");
		se = StreamEncoder.forOutputStreamWriter(out, this, enc);
	    }
	}
```
以下是使用InputStreamReader和OutputStreamWriter进行字节流到字符流的一个简单示例。 
```Java
	public static void main(String[] args) throws IOException {
 
		String file = "d:/stream.txt";
		String charset = "UTF-8";
		String string = "这是要保存的中文字符";
 
		FileOutputStream fos = new FileOutputStream(file);
		OutputStreamWriter osw = new OutputStreamWriter(fos, charset);
		try {
			osw.write(string);
 
		} finally {
			osw.close();
		}
 
		FileInputStream fis = new FileInputStream(file);
		InputStreamReader isr = new InputStreamReader(fis, charset);
		StringBuffer sb = new StringBuffer();
		char[] buf = new char[64];
		int count = 0;
		try {
			while ((count = isr.read(buf)) != -1) {
				sb.append(buf, 0, count);
			}
 
		} finally {
			isr.close();
		}
		System.out.println(sb.toString());
	}
```

# 在内存操作中的编码

String类提供了转换到字节的方法，也支持将字节转换为字符串的构造函数。 
```Java
	public static void main(String[] args) throws UnsupportedEncodingException {
		String s="这是一段中文字符串";
		byte[] bytes=s.getBytes("UTF-8");
		String string=new String(bytes,"UTF-8");
	}
```
Charset类提供encode()与decode()，分别对应char[]到byte[]的编码和byte[]到char[]的解码。
```Java
	public static void main(String[] args) {
		Charset cs = Charset.forName("UTF-8");
		ByteBuffer byteBuffer = cs.encode("这是要编码的字符串");
		CharBuffer charBuffer = cs.decode(byteBuffer);
	}
```
ByteBuffer提供一种char和byte之间的软转换，它们之间转换不需要编码和解码，只是把一个16bit的char拆分为2个8bit的byte表示，它们的实际值并没有被修改，仅仅是数据的类型做了转换。

	public static void main(String[] args) {
		ByteBuffer heapByteBuffer = ByteBuffer.allocate(1024);
		ByteBuffer buffer = heapByteBuffer.putChar('中');
		System.out.print(Integer.toBinaryString(buffer.get(0)) + " ");
		System.out.print(Integer.toBinaryString(buffer.get(1)));
	}

打印结果：
```
1001110 101101
```
编码问题（char-encoding-problem）典型示例：
```Java
	public class EncodeTest {
		static String toHexString(byte[] bytes) {
			StringBuilder sb = new StringBuilder("");
			if (bytes == null || bytes.length == 0) {
				return null;
			}
			for (int i = 0; i < bytes.length; i++) {
				int v = bytes[i] & 0xFF;
	 
				String hv = Integer.toHexString(v);
				sb.append(hv + " ");
			}
			return sb.toString();
		}
	 
		static String toHexString(char[] chars) {
			StringBuilder sb = new StringBuilder("");
			if (chars == null || chars.length == 0) {
				return null;
			}
			for (int i = 0; i < chars.length; i++) {
				String hv = Integer.toHexString((int) chars[i]);
				sb.append(hv + " ");
			}
			return sb.toString();
		}
	 
		public static void main(String[] args) {
			String string = "I am 李";
			// Unicode十进制数值为：    73       32       97       109      32       26446
			// Unicode十六进制字符串：  49       20       61       6d       20       674e
			// Unicode二进字符串：      01001001 00100000 01100001 01101101 00100000 0110011101001110
			try {
				byte[] iso8859 = string.getBytes("ISO-8859-1");
				byte[] gb2312 = string.getBytes("GB2312");
				byte[] gbk = string.getBytes("GBK");
				byte[] utf16 = string.getBytes("UTF-16");
				byte[] utf8 = string.getBytes("UTF-8");
	 
				System.out.println(toHexString(string.toCharArray()));
				// 输出结果：49 20 61 6d 20 674e
				
				/**
				 * ISO-8859-1编码会将不支持的字符编码为3f，即"?"字符。
				 */
				System.out.println(toHexString(iso8859));
				// 输出结果：49 20 61 6d 20 3f
				
				/**
				 * GB2312字符集有一个从char到byte的码表，不同的字符编码就是从这个码表找到与每个字符对应的字节，然后拼装成byte数组。
				 */
				System.out.println(toHexString(gb2312));
				// 输出结果：49 20 61 6d 20 c0 ee
				
				/**
				 * GBK编码兼容GB2312编码，且GBK包含的汉字字符更多。
				 */
				System.out.println(toHexString(gbk));
				// 输出结果：49 20 61 6d 20 c0 ee
				
				/**
				 * UTF-16仅将字符的高位与低位进行拆分变成两个字节，特点是编码效率非常高，规则很简单
				 * 前面用两个字节来保存BYTE_ORDER_MARK值，用来区分是高位字节在前，或者低位字节在前。
				 */
				System.out.println(toHexString(utf16));
				// 输出结果：fe ff 0 49 0 20 0 61 0 6d 0 20 67 4e 
				
				/**
				 * UTF-8编码也不用查表，效率很高，变长存储节省空间。
				 */
				System.out.println(toHexString(utf8));
				// 输出结果：49 20 61 6d 20 e6 9d 8e 
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
```

# 在Java Web中涉及的编解码

## URL的编解码
用户提交一个URL，在这个URL中可能存在中文，因此需要编码。

**Apache Tomcat**对URL的URI部分进行解码的字符集是在**Connector**的`<Connector URIEncoding="UTF-8"/>`中定义的，如果没有定义，那么将以`默认编码ISO-8859-1`解析。所以有中文URL时最好把URIEncoding设置成UTF-8编码。

`CATALINA_HOME\conf\server.xml`中修改`Connector` 配置如下：
```Xml
	<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8483"  URIEncoding="UTF-8"/>
```
`URL`中以`Get`方式请求的`QueryString`的解码是在`request.getParameter()`方法第一次被调用时进行的，解码字符集要么是`Header`中`ContentType`定义的`Charset`，要么是默认的`ISO-8859-1`，要使用`ContentType`中定义的编码，就要将`Connector`的`<Connector  URIEncoding="UTF-8" useBodyEncodingForURI="true"/>`中的`useBodyEncodingForURI`设置为true，对`QueryString`使用`BodyEncoding`解码。

`CATALINA_HOME\conf\server.xml`中修改`Connector` 配置如下：  
```Xml
	<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8483"  URIEncoding="UTF-8" useBodyEncodingForURI="true"/>
```

## HTTP Header的编解码
当客户端发起一个HTTP请求时，在Header中可能会传递其他参数，如Cookie、redirectPath等，对于这些参数`Tomcat默认使用ISO-8859-1解码`，并且不能设置成其他的解码格式。因此，如果你设置的Header中含有非ASCII字符，解码中肯定会有乱码。一个简单的解决办法：可以先将这些字符用`org.apache.catalina.util.URLEncoder`编码，再添加到Header中，使用这些项时再按照相应的字符集解码即可。

## POST表单的编解码
POST表单提交的参数的解码也是在`request.getParameter()`方法第一次被调用时发生的，POST表单的参数传递方法与QueryString不同，它是通过HTTP的BODY传递到服务端的。当我们在页面上单击提交按钮时浏览器首先将根据ContentType的Charset编码格式对在表单中填入的参数进行编码，然后提交到服务器端，在服务器端同样也是用ContentType中的字符集进行解码的。所以通过POST表单提交的参数一般不会出现问题，而且这个字符集编码是我们自己设置的，可以通过`request.setCharacterEncoding(charset)`来设置。

> <font color=red>注意</font>：要在第一次调用`request.getParameter()`方法之前就设置`request.setCharacterEncoding(charset)`，否则POST表单提交上来的数据可能出现乱码。

## HTTP BODY的编解码
当用户请求的资源已经成功获取后，这些内容将通过Response返回给客户端浏览器，这个过程要先经过编码，再到浏览器进行解码。编解码字符集可以通过`response.setCharacterEncoding`来设置，它将会覆盖`request.getCharacterencoding`的值，并且通过Header的Content-Type返回客户端，浏览器接收到返回的Socket流时将通过Content-Type的charset来解码。如果返回的HTTP Header中Content-Type没有设置charset，那么浏览器将根据HTML的`<meta HTTP-equiv="Content-Type" content="text/html; charset=GBK" />`中的charset来解码。如果也没有定义，那么浏览器将使用默认的编码来解码。

访问数据库都是通过客户端JDBC驱动来完成的，用JDBC来存取数据时要和数据的内置编码保持一致，可以通过设置JDBC URL来指定，如**MYSQL**：`url=“jdbc:mysql://localhost:3306/DB?useUnicode=true&characterEncoding=GBK”`。

# 在JS中的编码问题

## 外部引入JS文件 
在一个单独的JS文件中包含字符串输入 的情况，如：
```Html
	<html>
		<head>
			<meta http-equiv="content-type" content="text/html;charset=utf-8"/>
			<script src="script.js" charset="gbk"></script>
		</head>
	</html>
```
如果引入的**script.js**脚本中有如下代码：
```
	document.write("这是一段中文");
```
这时如果script没有设置cahrset，浏览器就会以当前这个页面的默认字符集解析这个JS文件。当**script.js**文件与当前页面的编码格式不一致时，就会出现乱码。

## JS的URL编码
在JS中处理URL编码的函数有三个：`escape()`、`encodeURI()`和`encodeURIComponent()`。 

### escape()
这个函数是将ASCII字母、数字、标点符号（* + - . / @ _）之外的其他字符转化成Unicode编码值，并且在编码值前加上“%u”，通过unescape()函数解码，如图所示。  
<div align=center>

![字符编码示意图](http://pengjunlee.3vzhuji.net/static/javacore/19.png "字符编码示意图")
<div align=left>

> <font color=red>注意</font>：escape()和unescape()已经从ECMAScript V3 标准中删除了，URL的编码可以用encodeURI和encodeURIComponent来代替。

### encodeURI()
与escap()相比，encodeURI()是真正的JS用来对URL编码的函数，它可以将整个URL中的字符（一些特殊字符除外，如：`！ # $ & ' ( ) * + , -  . / : ; = ? @ _ ~ 0-9 a-z A-Z）`进行UTF-8编码，在每个码值前加上`“%”`，解码通过decodeURI()函数，如图所示。 

<div align=center>

![字符编码示意图](http://pengjunlee.3vzhuji.net/static/javacore/20.png "字符编码示意图")
<div align=left>

### encodeURIComponent()
encodeURIComponent()这个函数比encodeURI()编码还要彻底，它除了对 ! ' ( ) * - . _ ~ 0-9 a-z A-Z这几个字符不编码，对其他所有字符都编码。这个函数通常用于将一个URL当作一个参数放在另一个URL中，解码通过decodeURIComponent()函数，如图所示。 

<div align=center>

![字符编码示意图](http://pengjunlee.3vzhuji.net/static/javacore/21.png "字符编码示意图")
<div align=left>

### Java与JS编解码问题
在Java端处理URL编解码有两个类，分别是 `java.net.URLEncoder`和`java.net.URLDecoder`。这两个类可以将所有`“%”`加UTF-8码值用UFT-8解码，从而得到原始字符。Java端的`URLEncoder`和`URLDecoder`与前端JS对应的是`encodeURIComponent`和`decodeURIComponent`。

> <font color=red>注意</font>：可以用`encodeURIComponent`两次编码，如`encodeURIComponent(encodeURIComponent(str))`，这样在Java端通过`request.getParameter()`用GBK解码后取得的就是UTF-8编码的字符串，如果Java端需要使用这个字符串，则再用UTF-8解码一次；如果是将这个结果直接通过JS输出到前端，那么这个UTF-8字符串可以直接在前端正常显示。  

# 其他需要编码的地方

XML文件可以通过设置头来指定编码格式：  

`<?xml version="1.0" encoding="UTF-8"?>`  

**Velocity**模板设置编码的格式如下：  

`services.VelocityService.input.encoding=UTF-8`

JSP设置编码的格式如下：  

`<%@page contentType="text/html;charset=UTF-8"%>`