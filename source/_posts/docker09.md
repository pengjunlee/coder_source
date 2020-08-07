---
title: Docker虚拟化之--解决oci runtime error
date: 2020-08-01 13:09:00
updated: 2020-08-01 13:09:00
tags: Docker
categories: Docker
keywords: Docker, Linux
type: 
description: docker run 解决oci runtime error。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
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
> 转载自：<http://www.weitip.com/news/22.html>

在部署新服务器运行docker镜像的时候遇到了报错，记录下解决方法。

docker 启动容器报错：<font color=red>Error response from daemon: oci runtime error: container_linux.go:247: starting container process caused "process_linux.go:258: applying cgroup configuration for process caused \"Cannot set property TasksAccounting</font>

docker 是通过 `yum install docker` 安装的，搜了一把，原来是因为linux与docker版本的兼容性问题。那就卸载旧版本安装最新版试试。

**0.通过uname -r命令查看你当前的内核版本**
```Bash
	uname -r
```
**1.使用 root 权限登录 Centos。确保 yum 包更新到最新。**
```Bash
	sudo yum update
```
**2.卸载旧版本(如果安装过旧版本的话)**
```Bash
	sudo yum remove docker  docker-common docker-selinux docker-engine
```
**3.安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的**
```Bash
	sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
**4.设置yum源**
```Bash
	sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
**5.可以查看所有仓库中所有docker版本，并选择特定版本安装**
```Bash
	yum list docker-ce --showduplicates | sort -r
```
**6.安装docker**
```Bash
	sudo yum install docker-ce
```
**7.启动并加入开机启动**
```Bash
	sudo systemctl start docker
	sudo systemctl enable docker
```
**8.验证安装是否成功(有client和service两部分表示docker安装启动都成功了)**
```Bash
	docker version
```
经过以上一通操作，pull 一下镜像再执行docker run命令，问题解决。