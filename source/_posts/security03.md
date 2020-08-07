---
title: 安全框架系列之--一个例子理解OAuth2.0
date: 2020-08-01 14:03:00
updated: 2020-08-01 14:03:00
tags: OAuth2.0
categories: 安全框架
keywords: 安全, OAuth2.0
type: 
description: 一个简单例子带你轻松理解OAuth2.0。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
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
> 转载自：<http://www.ruanyifeng.com/blog/2019/04/oauth_design.html>

OAuth 2.0 是目前最流行的授权机制，用来授权第三方应用，获取用户数据。

这个标准比较抽象，使用了很多术语，初学者不容易理解。其实说起来并不复杂，下面我就通过一个简单的类比，帮助大家轻松理解，OAuth 2.0 到底是什么。

# 一、快递员问题
我住在一个大型的居民小区。

<div align=center>

![OAuth 2.0图](http://pengjunlee.3vzhuji.net/static/security/29.jpg "OAuth 2.0示意图")
<div align=left>

小区有门禁系统。

<div align=center>

![OAuth 2.0图](http://pengjunlee.3vzhuji.net/static/security/30.jpg "OAuth 2.0示意图")
<div align=left>

进入的时候需要输入密码。

<div align=center>

![OAuth 2.0图](http://pengjunlee.3vzhuji.net/static/security/31.jpg "OAuth 2.0示意图")
<div align=left>

我经常网购和外卖，每天都有快递员来送货。我必须找到一个办法，让快递员通过门禁系统，进入小区。

<div align=center>

![OAuth 2.0图](http://pengjunlee.3vzhuji.net/static/security/32.jpg "OAuth 2.0示意图")
<div align=left>

如果我把自己的密码，告诉快递员，他们就拥有了与我同样的权限，这样好像不太合适。万一我想取消某一个快递员进入小区的权力，也很麻烦，我自己的密码也得跟着改了，还得通知其他的快递员。

有没有一种办法，让快递员能够自由进入小区，又不必知道小区居民的密码，而且他的唯一权限就是送货，其他需要密码的场合，他都没有权限？

# 二、授权机制的设计
于是，我设计了一套授权机制。

- 第一步，门禁系统的密码输入器下面，增加一个按钮，叫做"获取授权"。快递员需要首先按这个按钮，去申请授权。
- 第二步，他按下按钮以后，屋主（也就是我）的手机就会跳出对话框：有人正在要求授权。系统还会显示该快递员的姓名、工号和所属的快递公司。我确认请求属实，就点击按钮，告诉门禁系统，我同意给予他进入小区的授权。
- 第三步，门禁系统得到我的确认以后，向快递员显示一个进入小区的令牌（access token）。令牌就是类似密码的一串数字，只在短期内（比如七天）有效。
- 第四步，快递员向门禁系统输入令牌，进入小区。

有人可能会问，为什么不是远程为快递员开门，而要为他单独生成一个令牌？这是因为快递员可能每天都会来送货，第二天他还可以复用这个令牌。另外，有的小区有多重门禁，快递员可以使用同一个令牌通过它们。

# 三、互联网场景
我们把上面的例子搬到互联网，就是 OAuth 的设计了。

- 首先，居民小区就是储存用户数据的网络服务。比如，微信储存了我的好友信息，获取这些信息，就必须经过微信的"门禁系统"。
- 其次，快递员（或者说快递公司）就是第三方应用，想要穿过门禁系统，进入小区。
- 最后，我就是用户本人，同意授权第三方应用进入小区，获取我的数据。

简单说，OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

# 四、令牌与密码
令牌（token）与密码（password）的作用是一样的，都可以进入系统，但是有三点差异。

- 令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。
- 令牌可以被数据所有者撤销，会立即失效。以上例而言，屋主可以随时取消快递员的令牌。密码一般不允许被他人撤销。
- 令牌有权限范围（scope），比如只能进小区的二号门。对于网络服务来说，只读令牌就比读写令牌更安全。密码一般是完整权限。

上面这些设计，保证了令牌既可以让第三方应用获得权限，同时又随时可控，不会危及系统安全。这就是 OAuth 2.0 的优点。

> 注意：只要知道了令牌，就能进入系统。系统一般不会再次确认身份，所以令牌必须保密，泄漏令牌与泄漏密码的后果是一样的。这也是为什么令牌的有效期，一般都设置得很短的原因。

OAuth 2.0 对于如何颁发令牌的细节，规定得非常详细。具体来说，一共分成四种授权类型（authorization grant），即四种颁发令牌的方式，适用于不同的互联网场景。下一篇文章，我就来介绍这四种类型，并给出代码实例。