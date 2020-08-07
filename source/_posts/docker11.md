---
title: Docker虚拟化之--管理容器小技巧
date: 2020-08-01 13:11:00
updated: 2020-08-01 13:11:00
tags: Docker
categories: Docker
keywords: Docker, Linux
type: 
description: Docker管理容器小技巧。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
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
> 转载自：<https://www.cnblogs.com/lonelyxmas/p/10336392.html>

# 查看运行容器
```Bash
	docker ps
```

# 查看所有容器
```Bash
	docker ps -a
```

# 进入容器
其中字符串为容器ID:
```Bash
	docker exec -it d27bd3008ad9 /bin/bash
```

# 停用全部运行中的容器
```Bash
	docker stop $(docker ps -q)
```

# 删除全部容器
```Bash
	docker rm $(docker ps -aq)
```

# 一条命令实现停用并删除容器
```Bash
	docker stop $(docker ps -q) & docker rm $(docker ps -aq)
```