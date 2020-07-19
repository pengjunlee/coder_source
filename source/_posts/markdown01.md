---
title: 10分钟掌握Markdown
date: 2020-07-18 12:00:00
updated: 2020-07-18 12:00:00
tags: 
indexing: true
categories: 博客搭建
keywords: Markdown, 语法
type: 
description: 一篇博客带你轻松搞定Markdown基础语法。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: true
top: true
---
# 前言
&emsp;&emsp;写过博客或者github上面的文档的，应该知道Markdown语法的重要性，不知道的朋友们也别着急，一篇博客带你轻松搞定Markdown基础语法。

# 快捷键
|<font face="黑体" color=fuchsia size=5>功能</font>|<font face="黑体" color=fuchsia size=5>快捷键</font>|
|:-|:-|
|加粗|Ctrl + B|
|斜体|Ctrl + I|
|引用|Ctrl + Q|
|插入链接|Ctrl + L|
|插入代码|Ctrl + K|
|插入图片|Ctrl + G|
|提升标题|Ctrl + H|
|有序列表|Ctrl + O|
|无序列表|Ctrl + U|
|横线|Ctrl + R|
|撤销|Ctrl + Z|
|重做|Ctrl + Y|

# 常用语法
## 文本
|<font face="黑体" color=fuchsia size=5>示例</font>|<font face="黑体" color=fuchsia size=5>效果</font>|
|:-|:-|
|`正常文字`|正常文字|
|`*斜体文字*`|*斜体文字*|
|`_斜体文字_`|_斜体文字_|
|`**粗体文字**`|**粗体文字**|
|`***斜体加粗***`|***斜体加粗***|

## 标题
&emsp;&emsp;一级标题：`# 一级标题`  
&emsp;&emsp;二级标题：`## 二级标题`  
&emsp;&emsp;三级标题：`### 三级标题`  
&emsp;&emsp;四级标题：`#### 四级标题`  
&emsp;&emsp;五级标题：`##### 五级标题`  
&emsp;&emsp;六级标题：`###### 六级标题`  

## 链接
|<font face="黑体" color=fuchsia size=5>示例</font>|<font face="黑体" color=fuchsia size=5>效果</font>|
|:-|:-|
|`[博客地址](https://blog.mtracy.club/ "李朋军的个人博客")`|[博客地址](https://blog.mtracy.club/ "李朋军的个人博客")|
|`[博客地址][blog-addr]`<br>`[blog-addr]:https://blog.mtracy.club/ "李朋军的个人博客"`|[博客地址](https://blog.mtracy.club/ "李朋军的个人博客")|
|`<https://blog.mtracy.club/>`|<https://blog.mtracy.club/>|

## 图片
&emsp;&emsp;***示例1***:  
`![博客地址头像](http://pengjunlee.3vzhuji.net/static/img/avatar.png "我的头像")`
&emsp;&emsp;或者：
`![博客地址头像][blog-avatar]`  
`[blog-avatar]:http://pengjunlee.3vzhuji.net/static/img/avatar.png "我的头像"`
&emsp;&emsp;<font face="黑体" color=blue><b>效果：</b></font>

![博客地址头像](http://pengjunlee.3vzhuji.net/static/img/avatar.png "我的头像")


&emsp;&emsp;***示例2：调整大小***
`<img src="http://pengjunlee.3vzhuji.net/static/img/avatar.png" width="100px" height="100px" />`  
<img src="http://pengjunlee.3vzhuji.net/static/img/avatar.png" width="100px" height="100px" />  

&emsp;&emsp;***示例3：居中***
`<div align=center>![博客地址头像](http://pengjunlee.3vzhuji.net/static/img/avatar.png "我的头像")`
`<div align=left>`

<div align=center>

![博客地址头像](http://pengjunlee.3vzhuji.net/static/img/avatar.png "我的头像")
<div align=left>

&emsp;&emsp;***示例4：引用本地图片***  
`![博客地址头像](./imgs/csdn.jpg "我的头像")`

## 分割线
&emsp;&emsp;在一行中使用 `***` 或者 `---` 或者 `___` 都可以插入一个分割线，行内不允许有其他内容。

## 代码块
&emsp;&emsp;行内代码块（使用ESC键上的英文字符 \` 包裹代码）：  

	`var foo = 'bar';`
	
&emsp;&emsp;代码块（缩进四个空格或是一个制表符）：
```
    <table>
        <tr>
            <th>姓名</th>
        </tr>	
        <tr>
            <td>小明</td>
        </tr>	
    </table>
```

&emsp;&emsp;多行代码块（使用连续3个ESC键上的英文字符 \` 包裹代码）：  

	```
		<table>
			<tr>
			<th>姓名</th>
			</tr>	
			<tr>
			<td>小明</td>
			</tr>	
		</table>
	```

## 引用
&emsp;&emsp;**单层引用：**  
&emsp;&emsp;在被引用的文本前加上 `>` 符号，例如：`> 语出《论语》`
> 语出《论语》  

&emsp;&emsp;**嵌套引用：**
```  

> #引用1
>> ##引用2

```

## 列表
&emsp;&emsp;**无序列表：**（使用 `*`，`+`，`-` 表示）:  
```
+ item1
- item2
* item3
```
&emsp;&emsp;<font face="黑体" color=blue><b>效果：</b></font>

+ item1
- item2
* item3


&emsp;&emsp;**有序列表：**（使用数字紧跟一个英文句点表示）：
```
1. item1
2. item2
3. item3
```

1. item1
2. item2
3. item3

> 注意：<font face="黑体" color=red>无序列表和有序列表的列表序号和列表内容之间必须有一个空格。 </font> 

## 表格

	| Column 1 | Column 2      |Column 2      | 
	|:--------:| -------------:|:-------------|
	| centered 文本居中 | right-aligned 文本居右 | left-aligned 文本居左 |

&emsp;&emsp;示例：
	
	|项目     | Value   |
	|:--------|:--------|
	|电脑     | $1600   |
	|手机     | $12     |
	|导管     | $1      |

&emsp;&emsp;<font face="黑体" color=blue><b>效果：</b></font>

|项目     | Value   |
|:--------|:--------|
|电脑     | $1600   |
|手机     | $12     |
|导管     | $1      |

# 常用技巧

## 换行

+ `连续两个以上空格+回车`
+ `使用html语言换行标签：<br/>`

## 缩进
+ `&nbsp;`：缩进1/4中文字符  
+ `&ensp;`：缩进1/2中文字符,1英文字符  
+ `&emsp;`：缩进1中文字符,2英文字符  