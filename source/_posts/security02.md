---
title: 安全框架系列之--Kerberos身份验证流程
date: 2020-08-01 14:02:00
updated: 2020-08-01 14:02:00
tags: Kerberos
categories: 安全框架
keywords: Kerberos, 安全
type: 
description: 一篇博文带你了解Kerberos身份验证流程。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
> 转载自：<https://www.cnblogs.com/zpchcbd/p/11707302.html>

# 介绍
Kerberos 是一种由 MIT（麻省理工大学）提出的一种网络身份验证协议。它旨在通过使用密钥加密技术为客户端/服务器应用程序提供强身份验证。

在 Kerberos 认证中，最主要的问题是如何证明「`你是你`」的问题，如当一个 Client 去访问 Server 服务器上的某服务时，Server 如何判断 Client 是否有权限来访问自己主机上的服务，同时保证在这个过程中的通讯内容即使被拦截或篡改也不影响通讯的安全性，这正是 Kerberos 解决的问题。在域渗透过程中 Kerberos 协议的攻防也是很重要的存在。

Kerberos 主要是用在域环境下的身份认证协议。

# Kerberos协议框架
在 Kerberos 协议中主要是有三个角色的存在：

1. 访问服务的 Client；
2. 提供服务的 Server；
3. KDC（Key Distribution Center）密钥分发中心

<div align=center>

![Kerberos图](http://pengjunlee.3vzhuji.net/static/security/50.jpg "Kerberos示意图")
<div align=left>

> <font color=red>注意</font>：

1. KDC 服务默认会安装在一个域的域控中
2. 从物理层面看，AD与KDC均为域控制器(Domain Controller)
3. AD其实是一个类似于本机SAM的一个数据库，全称叫account database，存储所有client的白名单，只有存在于白名单的client才能顺利申请到TGT
4. KDC 服务框架中包含一个 KRBTGT 账户，它是在创建域时系统自动创建的一个账号，你可以暂时理解为他就是一个无法登陆的账号，在发放票据时会使用到它的密码 HASH 值。

<div align=center>

![Kerberos图](http://pengjunlee.3vzhuji.net/static/security/51.jpg "Kerberos示意图")
<div align=left>

上文提到的TGT，KDC,client，server又是什么东西呢，下文来介绍下：

# Kerberos粗略的验证流程
先来举个例子：如果把 Kerberos 中的票据类比为一张火车票，那么 Client 端就是乘客，Server 端就是火车，而 KDC 就是就是车站的认证系统。如果Client端的票据是合法的（由你本人身份证购买并由你本人持有）同时有访问 Server 端服务的权限（车票对应车次正确）那么你才能上车。当然和火车票不一样的是 Kerberos 中有存在两张票，而火车票从头到尾只有一张。

在KDC中又分为两个部分：`Authentication Service`和`Ticket Granting Service`

- Authentication Service：AS 的作用就是验证 Client 端的身份（确定你是身份证上的本人），验证通过就会给一张 TGT（Ticket Granting Ticket）票给 Client。
- Ticket Granting Service：TGS 的作用是通过 AS 发送给 Client 的票（TGT）换取访问 Server 端的票（上车的票 ST）。ST（ServiceTicket）也被称之为 TGS Ticket，为了和 TGS 区分，在这里就用 ST 来说明，所以TGS Ticket和ST的意思是一样的。

这就说明了为什么在Kerberos中存有两种票，其实就是更加细分了其中的验证，简单的描述就是首先你拿身份证验证头像是不是一样，是的话就返回一张TGT，然后在通过验证车票获得上车的资格，这里就有对TGT的验证，通过的话再返回一张ST/TGS Ticket的票，如果都可以那么就验证成功了。

# Kerberos详解认证流程
当 Client 想要访问 Server 上的某个服务时，需要先向 AS 证明自己的身份，然后通过 AS 发放的 TGT 向 Server 发起认证请求，这个过程分为三块：

- The Authentication Service Exchange：Client 与 AS 的交互，
- The Ticket-Granting Service (TGS) Exchange：Client 与 TGS 的交互，
- The Client/Server Authentication Exchange：Client 与 Server 的交互。

<div align=center>

![Kerberos图](http://pengjunlee.3vzhuji.net/static/security/52.jpg "Kerberos示意图")
<div align=left>

## TheAuthentication Service Exchange

**KRB_AS_REQ(请求)**：

Client->AS：Client 先向 KDC 的 AS 发送 Authenticator1(内容为Client hash加密的时间戳）

**KRB_AS_REP(应答)**：

AS-> Client：AS根据用户名在AD中寻找是否在白名单中，然后根据用户名提取到对应的NTLM Hash，然后会生成一个随机数session key，然后返回给Client由Client的ntlm hash加密的session key-as作为AS数据，再返回TGT（使用KDC中krbtgt的NTLM Hash加密session key和客户端的信息，客户端的信息里面包含了时间戳等信息）

**AS验证的简述**：在 KDC(AD) 中存储了域中所有用户的密码 HASH，当 AS 接收到 Client 的请求之后会根据 KDC 中存储的密码来解密，解密成功并且验证信息。
验证成功后返回给 Client两个东西，一个是由 Client 密码 HASH 加密的 session key-as 和 TGT（由 KRBTGT HASH 加密的 session key 和 TimeStamp 等信息）。

<div align=center>

![Kerberos图](http://pengjunlee.3vzhuji.net/static/security/53.jpg "Kerberos示意图")
<div align=left>

## TheTicket-Granting Service (TGS) Exchange

**KRB_TGS_REQ(请求)**：

Client 接收到了加密后的session key-as 和 TGT 之后，先用自身密码 HASH解密得到session key ，TGT 是由 KDC 中KRBTGT的HASH加密，所以Client 无法解密。这时 Client 再用session key加密的TimeStamp，然后再和TGT 一起发送给 KDC 中的 TGS（TicketGranting Server）票据授权服务器换取能够访问 Server 的票据。

**KRB_TGS_REP(应答)**：

TGS 收到 Client 发送过来的 TGT 和 Session key 加密的 TimeStamp 之后，首先会检查自身是否存在 Client 所请求的服务。如果服务存在，则用 KRBTGT的HASH解密 TGT。

一般情况下 TGS 会检查 TGT 中的时间戳查看 TGT 是否过期，且原始地址是否和 TGT 中保存的地址相同。

验证成功之后将返回Client两个东西，一个是用 session key 加密的 session key-tgs，然后另一个是 Client要访问的Server的密码 HASH 加密的 session key-tgs（前面和session key加密生成的）生成就是ST（TGS）

<div align=center>

![Kerberos图](http://pengjunlee.3vzhuji.net/static/security/54.jpg "Kerberos示意图")
<div align=left>

## TheClient/Server Authentication Exchange

**KRB_AP_REQ(请求)**：

Client -> Server 发送 Authenticator3(session key-tgs 加密 TimeStamp) 和票据 ST(Server 密码 HASH 加密 session key-tgs)

Client 收到 session key 加密生成的 session key-tgs 和 Server 密码 HASH 加密 session key-tgs生成的TGS 之后，用 session key 解密得到 session key-tgs，然后把 sessionkey-tgs 加密的 TimeStamp 和 ST（也就是TGS）一起发送给 Server。

**KRB_AP_REP(应答)**：

Server-> Client

server 通过自己的密码解密 ST，得到 sessionkey-tgs, 再用 sessionkey-tgs 解密 Authenticator3 得到 TimeStamp，验证正确返回验证成功。

<div align=center>

![Kerberos图](http://pengjunlee.3vzhuji.net/static/security/55.jpg "Kerberos示意图")
<div align=left>

上文文字描述的也就是最开头的图片，这里再放上去，是不是会发现这张图理解了？？？

<div align=center>

![Kerberos图](http://pengjunlee.3vzhuji.net/static/security/50.jpg "Kerberos示意图")
<div align=left>

# 参考文章

<https://tools.ietf.org/html/rfc4120.html>

<https://www.freebuf.com/articles/system/196434.html>