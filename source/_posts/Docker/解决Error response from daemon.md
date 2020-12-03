---
title: Docker容器系列之--解决Error response from daemon
date: 2020-08-01 13:10:00
updated: 2020-08-01 13:10:00
tags: Docker
categories: Docker
keywords: Docker, Linux
type: 
description: 解决Error response from daemon 错误。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img10.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img10.jpg
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
> 原文传送门：<https://www.clxz.top/2019/03/31/111040/>

启动 docker 映射到宿主机时出现如下错误时：

<font color=red>/usr/bin/docker-current: Error response from daemon: driver failed programming external connectivity on endpoint sc_mysql (1bc03030afe9f722ae1e6b46166172a70cf87bcc3f02f0acdac0be2a7f0f0036): (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 3306 -j DNAT --to-destination 172.17.0.2:3306 ! -i docker0: iptables: No chain/target/match by that name.</font>

这是由于来自守护进程的错误响应，而致使外部连接失败。解决的办法就是将其 docker 进程 kill 掉，然后再 清空掉iptables 下 nat 表下的所有链（规则） 。最后，将 docker 的网桥删除，并重启 docker 服务
```Bash
	[root@seichung ] pkill docker                        # 终止进程
	[root@seichung ] iptables -t nat -F                  # 清空 nat 表的所有链
	[root@seichung ] ifconfig docker0 down               # 停止 docker 默认网桥
	[root@seichung ] yum install bridge-utils -y         # 部分机器是无法使用 brctl，所以需要提前安装
	[root@seichung ] brctl delbr docker0                 # 删除网桥  
	[root@seichung ] systemctl restart docker            # 重启docker
```
docker 镜像成功映射后，会在 iptables 上添加所属的链，如图：

<div align=center>

![Docker](http://pengjunlee.3vzhuji.net/static/docker/21.png "Docker示意图")
<div align=left>

 