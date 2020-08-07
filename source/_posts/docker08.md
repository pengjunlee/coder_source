---
title: Docker虚拟化之--SELinux引起的docker启动失败
date: 2020-08-01 13:08:00
updated: 2020-08-01 13:08:00
tags: Docker
categories: Docker
keywords: Docker, Linux
type: 
description: SELinux引起的docker启动失败解决办法。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
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
> 转载自：<https://blog.51cto.com/10950710/2131803>

# 问题描述
有一台使用中的docker突然发生了故障，然后启动docker失败。

机器的系统版本：`CentOS Linux release 7.3.1611 (Core)`

最后将这台机器的docker卸载后重装，但是docker还是起不来，启动docker报“<font color=red>Error starting daemon: SELinux is not supported with the overlay2 graph driver on this kernel.</font>”的错误。具体的报错信息如下：
```Bash
	[root@registry lib]# systemctl start docker
	Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
	[root@registry lib]# systemctl status docker.service
	● docker.service - Docker Application Container Engine
	   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
	   Active: failed (Result: exit-code) since Fri 2018-06-22 15:22:45 CST; 10s ago
	     Docs: http://docs.docker.com
	  Process: 6374 ExecStart=/usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userland-proxy-path=/usr/libexec/docker/docker-proxy-current --init-path=/usr/libexec/docker/docker-init-current --seccomp-profile=/etc/docker/seccomp.json $OPTIONS $DOCKER_STORAGE_OPTIONS $DOCKER_NETWORK_OPTIONS $ADD_REGISTRY $BLOCK_REGISTRY $INSECURE_REGISTRY $REGISTRIES (code=exited, status=1/FAILURE)
	 Main PID: 6374 (code=exited, status=1/FAILURE)
	
	Jun 22 15:22:42 registry.sefon.com systemd[1]: Starting Docker Application Container Engine...
	Jun 22 15:22:42 registry.sefon.com dockerd-current[6374]: time="2018-06-22T15:22:42.987932115+08:00" level=info msg="libcontainerd: new containerd process, pid: 6381"
	Jun 22 15:22:45 registry.sefon.com dockerd-current[6374]: Error starting daemon: SELinux is not supported with the overlay2 graph driver on this kernel. Either boot into a newer kernel or disabl...nabled=false)         #关键报错信息
	Jun 22 15:22:45 registry.sefon.com systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
	Jun 22 15:22:45 registry.sefon.com systemd[1]: Failed to start Docker Application Container Engine.
	Jun 22 15:22:45 registry.sefon.com systemd[1]: Unit docker.service entered failed state.
	Jun 22 15:22:45 registry.sefon.com systemd[1]: docker.service failed.
	Hint: Some lines were ellipsized, use -l to show in full.
```

# 原因分析
根据报错信息“<font color=red>Error starting daemon: SELinux is not supported with the overlay2 graph driver on this kernel. Either boot into a newer kernel or disabl...nabled=false)</font>”的提示，这台机器的linux的内核中的SELinux不支持 overlay2 graph driver 。
解决方法有两个，要么启动一个新内核，要么就在docker配置文件里面里禁用selinux，--selinux-enabled=false

# 解决方法
没有启动新的内核，修改的docker配置文件。将配置文件的“`--selinux-enabled`”改成“`--selinux-enabled=false`”，然后再重启docker。
```Bash
	[root@registry lib]# cat /etc/sysconfig/docker
	# /etc/sysconfig/docker
	
	# Modify these options if you want to change the way the docker daemon runs
	#OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
	
	OPTIONS='--selinux-enabled=false --log-driver=journald --signature-verification=false --registry-mirror=https://fzhifedh.mirror.aliyuncs.com --insecure-registry=registry.sese.com'    #修改这里的"--selinux-enabled"，改成"--selinux-enabled=false"
	if [ -z "${DOCKER_CERT_PATH}" ]; then
	    DOCKER_CERT_PATH=/etc/docker
	fi
	
	......   #配置文件后面的内容省略
	[root@registry lib]# 
```
然后重新启动docker，就正常启动了：
```Bash
	[root@registry lib]# systemctl start docker
	[root@registry lib]# docker ps -a
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAME
	
	[root@registry lib]# 
```
解决方法参考文档：<https://www.cnblogs.com/weifeng1463/p/9040892.html>