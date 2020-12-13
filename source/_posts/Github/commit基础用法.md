---
title: GitHub系列之--Git commit基础用法
date: 2020-08-01 16:01:00
updated: 2020-08-01 16:01:00
tags: GitHub
categories: GitHub
keywords: 版本, GitHub
type: 
description: Git commit基础用法。
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
highlight_shrink: false
top: false
---
> 原文传送门：<https://www.cnblogs.com/qianqiannian/p/6005628.html>

git commit 主要是将暂存区里的改动给提交到本地的版本库。每次使用git commit 命令我们都会在本地版本库生成一个40位的哈希值，这个哈希值也叫commit-id，commit-id在版本回退的时候是非常有用的，它相当于一个快照,可以在未来的任何时候通过与git reset的组合命令回到这里。
```Bash
	git commit -m “message”
```
这种是比较常见的用法，-m 参数表示可以直接输入后面的“message”，如果不加 -m参数，那么是不能直接输入message的，而是会调用一个编辑器一般是vim来让你输入这个message。

message即是我们用来简要说明这次提交的语句。还有另外一种方法，当我们想要提交的message很长或者我们想描述的更清楚更简洁明了一点，我们可以使用这样的格式，如下：
```Bash
    git commit -m ‘
    message1
    message2
    message3
    ’
```
# git commit -a -m “massage”
其他功能如-m参数，加的-a参数可以将所有已跟踪文件中的执行修改或删除操作的文件都提交到本地仓库，即使它们没有经过git add添加到暂存区，注意，新加的文件（即没有被git系统管理的文件）是不能被提交到本地仓库的。建议一般不要使用-a参数，正常的提交还是使用git add先将要改动的文件添加到暂存区，再用git commit 提交到本地版本库。

# git commit --amend
如果我们不小心提交了一版我们不满意的代码，并且给它推送到服务器了，在代码没被merge之前我们希望再修改一版满意的，而如果我们不想在服务器上abondon，那么我们怎么做呢？
```Bash
	git commit --amend //也叫追加提交，它可以在不增加一个新的commit-id的情况下将新修改的代码追加到前一次的commit-id中
```

1. 假如现在版本库里最近的一版正是我们想要追加进去的那版，此时是最简单的，直接修改工作区代码，然后git add，之后就可以直接进行git push到服务器，中间不需要进行其他的操作如git pull等
2. 如果现在版本库里最近的一版不是我们想要追加进去的那版，那么此时我们需要将版本库里的版本回退到我们想要追加的那一版，想要将版本回退到我们想要的哪一版有好几种方法;

<br/>

- 第一种即是我们从服务器上选取我们需要的版本，直接进行挑拣，在服务器的提交管理页面上右上方一般会有一个Download按钮，点击会弹出一个下拉框，选择其中的cherry-pick，复制命令，之后在我们版本仓库对应的目录下运行这个命令，执行完后，使用git log -1 命令，可以查看到现在版本库里最近的一版变成了我们刚才挑拣的这版，此时再在工作区直接修改代码，改完之后进行git add，再执行本git commit --amend命令，之后git push。
- 使用gitk或其他的图形界面化工具，在终端输入 gitk，回车，会弹出gitk的图形界面，在界面的左侧部分陈列着版本库中的一条条commit-id，此时选中我们需要的那一版，右键点击之后会弹出一个选择菜单，如果是在master  分支上，那么其中会有一项是 Reset master branch to here，点击这项，会弹出一个名为confirm reset的确认box，选择reset type 中的hard项，再点击OK，关闭gitk图形界面，回到终端，运行git log -1命令，发现现在版本库里最近的一次提交已经是我们希望的那一版了，此时再在工作区直接修改代码，改完之后进行git add，再执行本git commit --amend命令，之后git push。
- 如果我们知道我们需要的版本与现在最近的版本中间隔着 n 个提交，那么我们可以直接使用git reset --hard HEAD～n命令，关于git reset 命令有详解，此时这个命令执行完后，运行git log -1 命令我们会发现现在版本库里最近的一版就是我们需要的那版，此时再在工作区直接修改代码，改完之后进行git add，再执行本git commit --amend命令，之后git push。
- 如果我们不知道我们需要的版本与现在最近的版本中间隔着 n 个提交，那么我们可以使用git log来查看版本库中的commit-id，找到我们需要的commit-id后，在终端中执行git reset --hard commit-id，时这个命令执行完后，运行git log -1 命令我们会发现现在版本库里最近的一版就是我们需要的那版，此时再在工作区直接修改代码，改完之后进行git add，再执行本git commit --amend命令，之后git push。

# git commit --help
查看帮助，还有许多参数有其他效果，一般来说了解上述三种即可满足我们工作中的日常开发了。