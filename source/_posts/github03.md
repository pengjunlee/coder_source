---
title: GitHub系列之--Windows下使用Git管理本地代码
date: 2020-08-01 16:03:00
updated: 2020-08-01 16:03:00
tags: GitHub
categories: GitHub
keywords: 版本, GitHub
type: 
description: Windows下使用Git管理本地代码。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
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
# 一、什么是Git
Git是一款免费的、开源的、分布式的版本控制系统。旨在快速高效地处理无论规模大小的任何软件工程。
每一个 Git克隆都是一个完整的文件库，含有全部历史记录和修订追踪能力，不依赖于网络连接或中心服务器。其最大特色就是“分支”及“合并”操作非常快速、简便。

#二、Git和Svn的区别
SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而干活的时候，用的都是自己的电脑，所以首先要从中央服务器哪里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，如果在局域网还可以，带宽够大，速度够快，如果在互联网下，如果网速慢的话，就纳闷了。
Git是分布式版本控制系统，那么它就没有中央服务器的，每个人的电脑就是一个完整的版本库，这样，工作的时候就不需要联网了，因为版本都是在自己的电脑上。既然每个人的电脑都有一个完整的版本库，那多个人如何协作呢？比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

# 三、在Windows上安装Git
msysGit 发行了exe格式的Git安装文件，可以通过以下网站进行下载。

官网下载地址：

<https://git-for-windows.github.io>  

<https://git-scm.com/download/win>

国内下载地址：<https://github.com/waylau/git-for-win>

下载的Git安装文件是exe格式的可执行文件，直接打开，按照默认的配置一路点击Next即可完成安装。

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/01.png "Github示意图")
<div align=left>

# 四、创建本地仓库
## 1) 配置用户身份
在Git Bash中，输入如下指令 ：

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/02.png "Github示意图")
<div align=left>

此操作在多人协作时非常有用，可以用来标识更新代码的用户的身份。
## 2) 切换到需要创建仓库的文件目录 

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/03.png "Github示意图")
<div align=left>

## 3) 初始化本地仓库
在仓库目录下，输入指令“git init”来初始化git的本地仓库，该操作会在仓库目录下生成一个.git的隐藏文件夹，用来记录用户的git操作。若要删除本地仓库，直接删除仓库下的这个隐藏文件夹即可。 

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/04.png "Github示意图")
<div align=left>

输入指令“git status” 来查看当前仓库中的文件状态。 

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/05.png "Github示意图")
<div align=left>

## 4) 提交代码到本地仓库
使用“git add”命令来添加要提交的文件。 
语法：`git add .（表示添加所有文件）|目录名|文件名`

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/06.png "Github示意图")
<div align=left>

添加文件后，输入指令“git status” 来查看当前仓库中的文件状态。 

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/07.png "Github示意图")
<div align=left>

使用“git commit”命令来提交文件。 
语法：`git commit -m “提交描述信息”`

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/08.png "Github示意图")
<div align=left>

提交完成后，输入指令“git status” 再次查看当前仓库中的文件状态。 

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/09.png "Github示意图")
<div align=left>

# 五、查看文件更新状态
**1)“git status”命令用来查看本地文件和当前版本的文件有哪些不同 。**

当有新文件添加进来时：

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/10.png "Github示意图")
<div align=left>

当有文件被修改时：

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/11.png "Github示意图")
<div align=left>

当有文件被删除时：

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/12.png "Github示意图")
<div align=left>

**2) “git diff”命令用来查看文件发生修改的具体内容，减号表示减少的部分，加号表示增加的部分。** 

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/13.png "Github示意图")
<div align=left>

# 六、撤销操作
## 1)文件修改的撤销
使用“git checkout”命令可以将发生修改的文件恢复到当前版本未修改时的状态，相当于svn中的“revert”操作。
语法：`git checkout -- <file>`

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/14.png "Github示意图")
<div align=left>

## 2)新增文件的撤销
使用“git reset HEAD”命令可以撤销未提交的“git add”操作。
语法：`git reset HEAD <file>`

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/15.png "Github示意图")
<div align=left>

# 七、日志查看
使用“git log”命令查看提交记录日志。

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/16.png "Github示意图")
<div align=left>

使用“git log id -p”命令查看当次提交具体的修改内容。

<div align=center>

![Github](http://pengjunlee.3vzhuji.net/static/github/17.png "Github示意图")
<div align=left>

按“q”键退出日志查看。
