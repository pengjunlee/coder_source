---
title: 喜好杂藏--二阶魔方还原教程
date: 2020-08-08 14:01:00
updated: 2020-08-08 14:01:00
tags: 魔方
categories: 喜好杂藏
keywords: 魔方, 二阶, 还原
type: 
description: 科学计算证明，一个随意打乱的二阶魔方，即使是最复杂的状态，仍可以在14步之内（180°旋转的情况当作两步的前提下）就可以还原。
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
aplayer: false
highlight_shrink: false
top: false
---
一般常见的二阶魔方还原方法是根据三阶的还原方法再加以简化改良而成。因为二阶可以理解为三阶的8个角块，所以可以这样说：`只要会还原三阶，就一定会还原二阶`。

科学计算证明，一个随意打乱的二阶魔方，即使是最复杂的状态，仍可以在14步之内（180°旋转的情况当作两步的前提下）就可以还原。也就是说，如果我们知道每一种打乱状态的排列组合情形的最短路径，那么我们最多只需14步就可以还原一个二阶。

但是，我们人类不可能像电脑一样每次都寻找出最少步骤来还原魔方，所以我们需要一些比较容易观察且操作方便的方法来还原。二阶除了可以用三阶的方法还原，主流还原方法有色先法和面先法。这里介绍的是相对比较简单而且速度较快的面先法。

二阶快速法无论是初学者还是已经学过一段时间的朋友，都可以很轻松的上手，甚至可以在四天内Sub10秒。它的原理很简单，无非是先两面，然后直接调整上下两层的角块，完成。

> <font color=red>重点</font>：在完成底面和顶面的时候均不需要考虑是否要对齐颜色！只需要完成两面先！这就是这种解法快速的关键！

> <font color=red>过程</font>：第一步:**底面** ---> 第二步:**顶面(OLL)** ---> 第三步:**整体换角(XLL)**  

# 第一步 底面
这一步里十五秒的观察很重要,尽量在十五秒内把第一面的完成步聚想好,争取在在两秒内完成一面。

# 第二步 顶面(OLL)
二阶的OLL并不是很多有如下几种，公式都很顺手:
<div align=center>

![魔方还原](./imgs/01.png "魔方还原示意图")
<div align=left>

# 第三步 整体换角(XLL)
这一步里，观察的速度是最重要的，这里不象三阶，只需判断一层，需要同时判断上下两层，不过这一步的时候PLL只有三大种情况，非常好判断，主要是观察相邻的色块。

## 上下两层的对角换
> <font color=red>判断技巧</font>：个魔方分为上下两层。然后上下两层均无相同色在一起者。
<div align=center>

![魔方还原](./imgs/02.png "魔方还原示意图")
<div align=left>


## 上层前两邻角 下层对角
> <font color=red>判断技巧</font>：整个魔方分上下两层。只有上面一层有两个相同的色拼在一起 如果是下面一层拼在了一起就立刻把整个魔方翻个180度。
<div align=center>

![魔方还原](./imgs/03.png "魔方还原示意图")
<div align=left>

## 上下层均只换前面的两邻角
> <font color=red>判断技巧</font>：整个魔方分两层。两层都有两个相同色拼在一起。 
<div align=center>

![魔方还原](./imgs/04.png "魔方还原示意图")
<div align=left>
当然，也许你很运气，(或者是很没运气)在完成两面的时候已经有一面完成，那么还有两种PLL情况: 
<div align=center>

![魔方还原](./imgs/05.png "魔方还原示意图")
<div align=left>

好了，现在你手上的二阶已经还原了。多练习就能轻松进10。  