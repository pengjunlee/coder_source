---
title: Docker虚拟化之--容器跨主机访问
date: 2020-08-01 13:07:00
updated: 2020-08-01 13:07:00
tags: Docker
categories: Docker
keywords: Docker, Linux
type: 
description: Docker容器跨主机访问。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
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
在《Docker容器间互联》一文中，我们了解了如何实现同一宿主机下的Docker容器互联。本章将继续之前的话题，接着介绍当容器部署在不同的主机上时，容器之间如何互联。

# 使用Weave实现容器互联
## Weave是什么？
Weave，原义为编织。在这里喻指建立一个虚拟网络，用于将运行在不同主机的 Docker 容器连接起来。

<div align=center>

![Docker](http://pengjunlee.3vzhuji.net/static/docker/18.png "Docker示意图")
<div align=left>

官网：<https://www.weave.works>

Github：<https://github.com/weaveworks/weave>

Weave是Github上一个比较热门的Docker容器网络方案，具有非常良好的易用性且功能强大。Weave 的框架它包含了两大主要组件：

- Weave：用户态的shell脚本，用于安装Weave，将container连接到Weave虚拟网络。并为它们分配IP。
- Weaver：运行于container内，每个Weave网络内的主机都要运行，是一个Go语言实现的虚拟网络路由器。不同主机之间的网络通信依赖于Weaver路由。

Weave通过创建虚拟网络使Docker容器能够跨主机通信并能够自动相互发现。

通过weave网络，由多个容器构成的基于微服务架构的应用可以运行在任何地方：主机，多主机，云上或者数据中心。

应用程序使用网络就好像容器是插在同一个网络交换机上一样，不需要配置端口映射，连接等。

在weave网络中，使用应用容器提供的服务可以暴露给外部，而不用管它们运行在何处。类似地，现存的内部系统也可以接受来自于应用容器的请求，而不管容器运行于何处。

<div align=center>

![Docker](http://pengjunlee.3vzhuji.net/static/docker/19.png "Docker示意图")
<div align=left>

如上图所示，在每一个部署Docker的主机（可能是物理机也可能是虚拟机）上都部署有一个W（即weave router，它本身也可以以一个容器的形式部署）。

weave网络是由这些weave routers组成的对等端点（peer）构成，并且可以通过weave命令行定制网络拓扑。

每个部署了weave router的主机之间都会建立TCP和UDP两个连接，保证weave router之间控制面流量和数据面流量的通过。控制面由weave routers之间建立的TCP连接构成，通过它进行握手和拓扑关系信息的交换通信。控制面的通信可以被配置为加密通信。而数据面由weave routers之间建立的UDP连接构成，这些连接大部分都会加密。这些连接都是全双工的，并且可以穿越防火墙。

## 如何使用Weave？
实验环境：
```
	Linux：CentOS 7
	Docker：1.13.1
	Host1：172.16.250.234
	Host2：172.16.250.239
```
### 安装Weave
 在两台主机上分别执行如下命令，安装Weave。
```Bash
	# 1. 下载二进制文件
	[root@localhost ~]# wget -O /usr/local/bin/weave git.io/weave
	# 2. 为 weave 文件添加可执行权限
	[root@localhost ~]# chmod a+x /usr/local/bin/weave
	# 3. 验证安装版本
	[root@localhost ~]# weave version
	weave script 2.6.0
	weave git-f43f3fc2e835
```
### 启动Weave网络
启动Weave网络之前，请确保所以要互联的主机上都已经正确安装并且启动 Docker守护进程。

在Host1（172.16.250.234）上执行如下命令：
```Bash
	[root@localhost ~]# weave launch
	5815608f50371c4dc44ae0f58dcd80183678c9e32641d74c6bc21d1f8650672d
	[root@localhost ~]# eval $(weave env)
	 
	# 再次检查 weave 版本
	[root@localhost ~]# weave version
	weave script 2.6.0
	weave 2.6.0
```
在Host2（172.16.250.239）上执行如下命令：
```Bash
	[root@localhost ~]# weave launch 172.16.250.234
	8353ccee1680b2249d39d0857ca44a469e3a11b67296742f2dc17d375e27ac31
	[root@localhost ~]# eval $(weave env)
	 
	# 检查 weave 连接状态
	[root@localhost ~]# weave status connections
	-> 172.16.250.234:6783   established fastdp 86:c4:52:b3:cf:8b(localhost.localdomain) mtu=1376
	# 查看 weave 详细状态
	[root@localhost ~]# weave status
	 
	        Version: 2.6.0 (up to date; next check at 2019/12/09 15:38:23)
	 
	        Service: router
	       Protocol: weave 1..2
	           Name: d2:cf:f5:9d:9c:c7(localhost.localdomain)
	     Encryption: disabled
	  PeerDiscovery: enabled
	        Targets: 1
	    Connections: 1 (1 established)
	          Peers: 2 (with 2 established connections)
	 TrustedSubnets: none
	 
	        Service: ipam
	         Status: ready
	          Range: 10.32.0.0/12
	  DefaultSubnet: 10.32.0.0/12
	 
	        Service: dns
	         Domain: weave.local.
	       Upstream: 172.16.0.3
	            TTL: 1
	        Entries: 0
	 
	        Service: proxy
	        Address: unix:///var/run/weave/weave.sock
	 
	        Service: plugin (legacy)
	     DriverName: weave
	# 列出当前启动的 docker 容器
	[root@localhost ~]# docker ps -a
	CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS               NAMES
	8353ccee1680        weaveworks/weave:2.6.0       "/home/weave/weave..."   8 minutes ago       Up 8 minutes                            weave
	aa5b7eac1049        weaveworks/weaveexec:2.6.0   "data-only"              50 minutes ago      Created                                 weavevolumes-2.6.0
	b0129638b441        weaveworks/weavedb:latest    "data-only"              5 hours ago         Created                                 weavedb
```
<font color=red>执行 weave launch 命令常见错误</font>： 
```Bash
	# 若出现以下错误
	[root@localhost ~]# weave launch
	WARNING: existing iptables rule
	 
	    '-A FORWARD -j REJECT --reject-with icmp-host-prohibited'
	 
	will block name resolution via weaveDNS - please reconfigure your firewall.
	 
	# 解决办法：关闭防火墙
	[root@localhost ~]# systemctl stop firewalld.service # 停止firewall
	[root@localhost ~]# systemctl disable firewalld.service # 禁止firewall开机启动
	# 若出现以下错误
	[root@localhost ~]# weave launch
	cannot locate running docker daemon
	Warning: unable to detect proxy TLS configuration. To enable TLS, launch the proxy with 'weave launch' and supply TLS options. To suppress this warning, supply the '--no-detect-tls' option.
	The weave container has died. Consult the container logs for further details.
	 
	# 解决办法：启动时增加 --no-detect-tls 选项
	[root@localhost ~]# weave launch --no-detect-tls
```
<font color=red>host clock skew of -6124s exceeds 900s limit，时钟偏差超过限制的解决方案</font>：  
```Bash
	# 若出现以下错误
	[root@localhost ~]# weave status connections
	-> 172.16.250.234:6783   failed      host clock skew of -6124s exceeds 900s limit, retry: 2019-12-09 08:26:45.12313254 +0000 UTC m=+451.600743522
	 
	# 解决办法：重新设置时间和时区
	[root@localhost ~]# tzselect # 选择时区
	[root@localhost ~]# date -s "20191209 16:31:01" # 设置时间
```
### 容器连通性检查
#### 同一宿主机下容器的连通性

在同一太宿主机上的容器默认情况下就可以相互连接，所以在此需要先将 Docker的 icc 选项设置为 false，避免干扰。

在Host2（172.16.250.239）上创建两个容器，查看其连通性。
```Bash
	# 启动容器
	[root@localhost ~]# docker run -itd --name container1 centos:7.6.1810
	61cd0cf587da7b328ee552531bde1a4fd044d48dcfb2091293430c5414bceb23
	[root@localhost ~]# docker run -itd --name container2 centos:7.6.1810
	5ca25464cb372230eb0e551eeda54c1553f9e90bf1abac12fb9fcaa3015e4e09
	[root@localhost ~]# docker exec -it container1 /bin/bash
	# 附加到 container1
	[root@localhost ~]# docker exec -it container1 /bin/bash
	# 查看 container1 的ip地址
	[root@61cd0cf587da /]# ifconfig
	eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 172.17.0.2  netmask 255.255.0.0  broadcast 0.0.0.0
	        inet6 fe80::42:acff:fe11:2  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
	        RX packets 3947  bytes 13317093 (12.7 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 3801  bytes 301864 (294.7 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 0  bytes 0 (0.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 0  bytes 0 (0.0 B)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	# 附加到 container2
	[root@localhost ~]# docker exec -it container2 /bin/bash
	# 查看是否能够 ping 通 container1
	[root@5ca25464cb37 /]# ping 172.16.0.2
	PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
	64 bytes from 172.16.0.2: icmp_seq=1 ttl=126 time=0.811 ms
	64 bytes from 172.16.0.2: icmp_seq=2 ttl=126 time=0.777 ms
	64 bytes from 172.16.0.2: icmp_seq=3 ttl=126 time=0.836 ms
	^C
	--- 172.16.0.2 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2001ms
	rtt min/avg/max/mdev = 0.777/0.808/0.836/0.024 ms
```
#### 不同宿主机下容器的连通性 

在Host1（172.16.250.234）上创建一个容器，查看其是否能够ping 通在 Host2（172.16.250.239）中创建的 container1（172.17.0.2）。
```Bash
	[root@localhost ~]# docker run -itd --name container3 centos:7.6.1810
	5fa9f0dc34272c7dada392e5c0e74a527439f0ea71dada0b464c9dfe856ced72
	[root@localhost ~]# docker exec -it container3 /bin/bash
	# 附加到 container3
	[root@localhost ~]# docker exec -it container3 /bin/bash
	# 查看是否能够 ping 同 container1
	[root@5fa9f0dc3427 /]# ping 172.16.0.2
	PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
	64 bytes from 172.16.0.2: icmp_seq=1 ttl=126 time=0.634 ms
	64 bytes from 172.16.0.2: icmp_seq=2 ttl=126 time=0.735 ms
	64 bytes from 172.16.0.2: icmp_seq=3 ttl=126 time=0.749 ms
	64 bytes from 172.16.0.2: icmp_seq=4 ttl=126 time=0.730 ms
	^C
	--- 172.16.0.2 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3000ms
	rtt min/avg/max/mdev = 0.634/0.712/0.749/0.045 ms 
```
由此可见，不同宿主机下的各容器也能正常相互访问。

# Open-vSwitch是什么？
Open vSwitch 是一个高质量的、多层虚拟交换机，使用开源 Apache 2.0 许可协议，由 Nicira Networks 开发，主要实现代码为可移植的C代码。它的目的是让大规模网络自动化可以通过编程扩展，同时仍然支持标准的管理接口和协议（例如：NetFlow，sFlow，SPAN，RSPAN，CLI，LACP，802.lag）。

# 什么是GRE隧道？
GRE （Generic Routing Encapsulation，通用路由协议封装），是一种通过使用互联网络的基础设施在网络之间传递数据的方式。使用隧道传递的数据（或负载）可以是不同协议的数据帧或包。隧道协议将其他协议的数据帧或包重新封装然后通过隧道发送。新的帧头提供路由信息，以便通过互联网传递被封装的负载数据。

# 使用Open-vSwitch实现跨主机容器互联
使用 Open vSwitch 实现跨主机容器互联的网络拓扑图如下：

<div align=center>

![Docker](http://pengjunlee.3vzhuji.net/static/docker/20.png "Docker示意图")
<div align=left>

由于系统资源有限，使用Open-vSwitch实现跨主机容器互联的具体过程就不再具体示例了。