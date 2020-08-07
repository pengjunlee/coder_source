---
title: Docker虚拟化之--网络基础
date: 2020-08-01 13:04:00
updated: 2020-08-01 13:04:00
tags: Docker
categories: Docker
keywords: Docker, Linux
type: 
description: Docker网络基础。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
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
Docker守护进程在启动时会自动创建一个docker0网卡（Linux虚拟网桥），用来为各个Docker容器的网络连接提供支持。

用户每启动一个Docker容器都会在运行Docker守护进程的宿主机上创建一个名称以veth开头的网络接口，Docker容器正是通过这个这个网络接口来实现与docker0之间的网络连接。

<div align=center>

![Docker](http://pengjunlee.3vzhuji.net/static/docker/14.png "Docker示意图")
<div align=left>

以一个启动了两个Docker容器的宿主机为例，查看其网络配置类似下面所示：
```Bash
	# 如果报错：ifconfig command not found，执行以下命令
	[root@localhost ~]# yum -y install net-tools
	# 查看宿主机网络配置
	[root@localhost ~]# ifconfig
	docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
	        inet6 fe80::42:2dff:fee6:86c9  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:2d:e6:86:c9  txqueuelen 0  (Ethernet)
	        RX packets 57473  bytes 3468100 (3.3 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 58782  bytes 176093357 (167.9 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 172.16.250.239  netmask 255.255.255.0  broadcast 172.16.250.255
	        inet6 fe80::8aa:2a37:21f1:f511  prefixlen 64  scopeid 0x20<link>
	        ether 00:50:56:84:18:99  txqueuelen 1000  (Ethernet)
	        RX packets 39686464  bytes 4636752774 (4.3 GiB)
	        RX errors 0  dropped 2248  overruns 0  frame 0
	        TX packets 822537  bytes 2607546264 (2.4 GiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 454  bytes 39532 (38.6 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 454  bytes 39532 (38.6 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	veth0a1f0b1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet6 fe80::4042:29ff:fe71:2060  prefixlen 64  scopeid 0x20<link>
	        ether 42:42:29:71:20:60  txqueuelen 0  (Ethernet)
	        RX packets 21  bytes 2910 (2.8 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 25  bytes 2387 (2.3 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	veth5d9f0b5: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet6 fe80::d4b8:bfff:feea:48d7  prefixlen 64  scopeid 0x20<link>
	        ether d6:b8:bf:ea:48:d7  txqueuelen 0  (Ethernet)
	        RX packets 30189  bytes 1892972 (1.8 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 30584  bytes 79838742 (76.1 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	# 如果报错：brctl command not found，执行以下命令
	[root@localhost ~]# yum -y install bridge-utils
	# 查看网桥
	[root@localhost ~]# brctl show
	bridge name	bridge id		STP enabled	interfaces
	docker0		8000.02422de686c9	no		veth0a1f0b1
											veth5d9f0b5
```
任一Docker容器的网络配置类似如下：
```Bash
	[root@46c6dc5d78f2 nginx-1.16.1]# ifconfig
	eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 172.17.0.4  netmask 255.255.0.0  broadcast 0.0.0.0
	        inet6 fe80::42:acff:fe11:4  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:ac:11:00:04  txqueuelen 0  (Ethernet)
	        RX packets 30584  bytes 79838742 (76.1 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 30189  bytes 1892972 (1.8 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 0  bytes 0 (0.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 0  bytes 0 (0.0 B)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
# 修改docker0地址
编辑 `/etc/docker/daemon.json` 文件，增加 "bip": "ip/netmask" 配置，切勿与宿主机同网段：
```
	{
	    "bip":"192.168.100.1/24"
	}
```
修改完成后，保存文件并重启docker服务：
```Bash
	# 重启Docker服务
	[root@localhost ~]# systemctl restart docker
	# 查看docker0配置是否生效
	[root@localhost ~]# ifconfig
	docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
	        inet 192.168.100.1  netmask 255.255.255.0  broadcast 0.0.0.0
	        inet6 fe80::42:2dff:fee6:86c9  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:2d:e6:86:c9  txqueuelen 0  (Ethernet)
	        RX packets 64521  bytes 3905211 (3.7 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 66459  bytes 201315812 (191.9 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 172.16.250.239  netmask 255.255.255.0  broadcast 172.16.250.255
	        inet6 fe80::8aa:2a37:21f1:f511  prefixlen 64  scopeid 0x20<link>
	        ether 00:50:56:84:18:99  txqueuelen 1000  (Ethernet)
	        RX packets 39909674  bytes 4867970075 (4.5 GiB)
	        RX errors 0  dropped 2248  overruns 0  frame 0
	        TX packets 889739  bytes 2612736260 (2.4 GiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 470  bytes 40924 (39.9 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 470  bytes 40924 (39.9 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	# 启动一个容器
	[root@localhost ~]# docker start 5c302540af47
	5c302540af47
	[root@localhost ~]# docker attach 5c302540af47
	# 查看容器的 eth0 网卡配置
	[root@5c302540af47 /]# ifconfig
	eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 192.168.100.2  netmask 255.255.255.0  broadcast 0.0.0.0
	        inet6 fe80::42:c0ff:fea8:6402  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:c0:a8:64:02  txqueuelen 0  (Ethernet)
	        RX packets 7  bytes 578 (578.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 7  bytes 578 (578.0 B)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 0  bytes 0 (0.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 0  bytes 0 (0.0 B)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

# 自定义虚拟网桥
自定义网桥使用 docker network create --driver | -d bridge <bridge_name> 命令。
```Bash
	# 创建网桥
	[root@localhost ~]# docker network create --driver bridge --subnet 192.168.200.0/24 --ip-range 192.168.200.0/28  br01 
	69a232376b3b595a7132c9c20f8bb3d6fe2bfc1fb90851411b835c33749b07d8
	# 查看网桥
	[root@localhost ~]# docker network ls --no-trunc
	NETWORK ID                                                         NAME                DRIVER              SCOPE
	69a232376b3b595a7132c9c20f8bb3d6fe2bfc1fb90851411b835c33749b07d8   br01                bridge              local
	9299092e4ff2573bd2998568288fa063bb5ed87a6e0acbaa3401277b15c09b56   bridge              bridge              local
	c316a20108eaf909a1a64bac9ada69abe7a99ceda2d1a3733632dca6619648f0   host                host                local
	59bfb31a4394ab364a8dc97e209ee6b83ae5e1edf21e89d14307b245e5fa2b25   none                null                local
	# 查看宿主机网络配置，多了一个 br-* 网桥
	root@localhost ~]# ifconfig
	br-69a232376b3b: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 192.168.200.1  netmask 255.255.255.0  broadcast 0.0.0.0
	        inet6 fe80::42:e5ff:fe06:6f14  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:e5:06:6f:14  txqueuelen 0  (Ethernet)
	        RX packets 470  bytes 40924 (39.9 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 470  bytes 40924 (39.9 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
	        inet 192.168.100.1  netmask 255.255.255.0  broadcast 0.0.0.0
	        inet6 fe80::42:2dff:fee6:86c9  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:2d:e6:86:c9  txqueuelen 0  (Ethernet)
	        RX packets 64529  bytes 3905747 (3.7 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 66459  bytes 201315812 (191.9 MiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
启动容器时指定使用刚刚创建的网桥：

```Bash
	# 启动一个 centos-tomcat 容器
	[root@localhost ~]# docker run -it --name tomcat_server --net br01 ea67b3a90969
	# 如果报错：ifconfig command not found，执行以下命令
	[root@396d8a6d395c /]# yum -y install net-tools
	# 查看的IP地址，是否与自定义网桥一致
	[root@396d8a6d395c /]# ifconfig
	eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 192.168.200.2  netmask 255.255.255.0  broadcast 0.0.0.0
	        inet6 fe80::42:c0ff:fea8:c802  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:c0:a8:c8:02  txqueuelen 0  (Ethernet)
	        RX packets 3526  bytes 12593380 (12.0 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 3160  bytes 234855 (229.3 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 84  bytes 10642 (10.3 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 84  bytes 10642 (10.3 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
若在启动容器时不指定所使用的网桥，默认会使用 docker0：
```Bash
	[root@localhost ~]# docker run -it --name tomcat_server2 ea67b3a90969
	[root@5b7d28b3b53b /]# ifconfig
	eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 192.168.100.2  netmask 255.255.255.0  broadcast 0.0.0.0
	        inet6 fe80::42:c0ff:fea8:6402  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:c0:a8:64:02  txqueuelen 0  (Ethernet)
	        RX packets 3739  bytes 12615775 (12.0 MiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 3595  bytes 295118 (288.2 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 0  bytes 0 (0.0 B)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 0  bytes 0 (0.0 B)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

# 更换网桥
由于  tomcat_server 容器启动时使用的是自定义网桥，现先将其与网桥连接断开。
```Bash
	[root@localhost ~]# docker ps -a
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
	5b7d28b3b53b        ea67b3a90969        "/bin/bash"         4 minutes ago       Up 4 minutes                            tomcat_server2
	396d8a6d395c        ea67b3a90969        "/bin/bash"         16 minutes ago      Up 16 minutes                           tomcat_server
	[root@localhost ~]# docker network disconnect br01 396d8a6d395c
	[root@localhost ~]# docker attach 396d8a6d395c
	[root@396d8a6d395c /]# ifconfig
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 84  bytes 10642 (10.3 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 84  bytes 10642 (10.3 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
当断开 tomcat_server 容器与网桥之间的连接后，eth0 网卡项不见了。
```Bash
	# 再新建一个网桥
	[root@localhost ~]# docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 br02
	d9206dac157d8837ba19d758b0f5ad4bca472f4805e758c10a4053d3a06f758b
	# tomcat_server连接新创建的网桥
	[root@localhost ~]# docker network connect br02 396d8a6d395c
	[root@localhost ~]# docker attach 396d8a6d395c
	# 查看容器网络配置
	[root@396d8a6d395c /]# ifconfig                                           
	eth2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 172.20.240.1  netmask 255.255.0.0  broadcast 0.0.0.0
	        inet6 fe80::42:acff:fe14:f001  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:ac:14:f0:01  txqueuelen 0  (Ethernet)
	        RX packets 16  bytes 1296 (1.2 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 8  bytes 648 (648.0 B)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	 
	lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
	        inet 127.0.0.1  netmask 255.0.0.0
	        inet6 ::1  prefixlen 128  scopeid 0x10<host>
	        loop  txqueuelen 1000  (Local Loopback)
	        RX packets 84  bytes 10642 (10.3 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 84  bytes 10642 (10.3 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
当 tomcat_server 容器连接上新网桥后，又重新出现了一个网卡项，IP地址与新网桥一致。