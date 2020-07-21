---
title: Java知识点系列之--执行bat脚本和shell脚本并传入参数
date: 2020-07-20 13:27:00
updated: 2020-07-20 13:27:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: Java中如何执行bat脚本和shell脚本并传入参数?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
之前想着在windows下和linux下调用一些python Scrapy的接口，发现路径问题，传参数问题都挺麻烦，遂改为在bat文件和shell中具体写方法，然后执行他们就好了

# 1.执行bat脚本
## (1)传入参数
bat处理文件中可引用的参数为`%0~%9`，`%0`是指批处理文件的本身，也可以说是一个外部命令；`%1~%9`是批处理参数，也称形参，例如：新建一个文件`test_argv.bat`，文件内容如下： 
```Bash
	@echo off
	 
	echo param[0] = %0
	echo param[1] = %1
	echo param[2] = %2
	echo param[3] = %3
	echo param[4] = %4
	echo param[5] = %5
	echo ...
	pause
```
调用时只需要在执行bat文件后加上参数即可，记得参数间有空格
```
test_argv.bat 1 game test what
```
此时输出：
```
	param[0] = test_argv.bat 
	param[1] = 1 
	param[2] = game 
	param[3] = test 
	param[4] = what 
	param[5] = 
	… 
	请按任意键继续…
```

## (2)调用
最简单的调用
```
	Runtime.getRuntime().exec("D:\\aaa\\remoteDesktop\\remoteConnection.bat");
```
这样调用是不会有回显的，如果你需要看到返回结果，就需要这样：
```Java
	try {
        // 执行ping命令
        Process process = Runtime.getRuntime().exec("cmd /c e:&dir");
        BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream(), Charset.forName("GBK")));
        String line = null;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
		}
	} catch (IOException e) {
	    e.printStackTrace();
	}
```
注意到了吗，这次的exec函数不是简单的选择bat文件的路径，多了“`cmd /c`” 这个前缀有以下使用方式：
```
	cmd /c dir 是执行完dir命令后关闭命令窗口。
	cmd /k dir 是执行完dir命令后不关闭命令窗口。
	cmd /c start dir 会打开一个新窗口后执行dir指令，原窗口会关闭。
	cmd /k start dir 会打开一个新窗口后执行dir指令，原窗口不会关闭。
```
> <font color=red>注意</font>：第二个调用方式在bat的输出内容过长时，会卡死！！！T_T

网上的解释说“`因为本地进程输入输出缓存有限，你不快点读取的话Process就挂在那了。`” 所以需要开一个进程去不断的取数据，具体实现方式见 ：<https://blog.csdn.net/aerchi/article/details/7669215> 。这个我没有检测，因为我发现，如果bat输出的内容过长时，使用第一种方式，不会卡死，若是想看到输出 可以加前缀`cmd /k start`，若是想把输出存起来 就加上后缀` >> 1.txt`，就好了，毕竟还要开线程太繁琐。

# 2.执行shell文件
## (1)传入参数
shell脚本传入参数与bat基本一致，只不过形参变成了`$1,,$2,$3`…..

例如，脚本`test.sh`的内容如下：
```
	name=$1
	echo "the ${name} are great man!"
```
执行`./test.sh Xiao Ming`命令，可以看到自己编写脚本的结果
```
the Xiao Ming are great man!
```

## (2)调用
linux环境果然友好得多，封装好了以下代码，传入shell文件的路径就好了
```Java
	public static String linuxShellexec(String shellPath) {
	    String result="";
	    try {
	        Process ps = Runtime.getRuntime().exec(shellPath);
	        ps.waitFor();
	        BufferedReader br = new BufferedReader(new InputStreamReader(ps.getInputStream()));
	        StringBuffer sb = new StringBuffer();
	        String line;
	        while ((line = br.readLine()) != null) {
	            sb.append(line).append("\n");
	        }
	        result = sb.toString();
	    }
	    catch (Exception e) {
	        e.printStackTrace();
	        result="linux下运行完毕";
	    }
	    return result;
	}
```

# 3. 参考文章

<https://www.cnblogs.com/happyPawpaw/p/3740903.html>

<https://blog.csdn.net/zyf_balance/article/details/51692065>

<https://blog.csdn.net/aerchi/article/details/7669215> 

<https://blog.csdn.net/a1010256340/article/details/76187353>  

<https://www.cnblogs.com/abel-hefei/p/7284256.html> 