---
title: 喜好杂藏--用HTML画一个二阶魔方
date: 2020-08-08 14:03:00
updated: 2020-08-08 14:03:00
tags: 魔方
categories: 喜好杂藏
keywords: 魔方, 二阶,HTML
type: 
description: 教你如何用HTML画一个二阶魔方。
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
aplayer: false
highlight_shrink: false
top: false
---
教你如何用HTML画一个二阶魔方，效果如下。
<div align=center>

![魔方](./imgs/12.png "魔方示意图")
<div align=left>

源代码如下：

```Html
	<!doctype html>
	<html>
	<head>
	<style type="text/css">
		body{
		background-color:gray;
		}
	
		.stage{
		-webkit-transform-perspective:1000px;
		margin:200px auto;
		}
	
		@-webkit-keyframes y-rot{
		0%  {-webkit-transform:rotateY(0deg);}
		50%  {-webkit-transform:rotateY(180deg);}
		100%  {-webkit-transform:rotateY(360deg);}
		}
	
		@-webkit-keyframes x-rot{
		0%  {-webkit-transform:rotateX(-30deg);}
		50%  {-webkit-transform:rotateX(30deg);}
		100%  {-webkit-transform:rotateX(-30deg);}
		}
	
		.xdiv{
		-webkit-transform-style:preserve-3d;
		-webkit-animation-name:x-rot;
		-webkit-animation-duration:6s;
		-webkit-animation-iteration-count:infinite;
		-webkit-animation-timing-function:ease;
		}
	
		.container{
		-webkit-transform-style:preserve-3d;
		-webkit-animation-name:y-rot;
		-webkit-animation-duration:10s;
		-webkit-animation-iteration-count:infinite;
		-webkit-animation-timing-function:linear;
		}
	
		.container>div{
		height:200px;
		width:200px;
		opacity:1;
		position:absolute;
		}
	
		.layer{
		display: flex;
		flex-wrap:wrap;
		}
	
		.layer div{
		box-sizing:border-box;
		height:100px;
		width:100px;
		border:1px solid black;
		}
	
		.container .layer:nth-child(1){
		-webkit-transform:rotateY(0deg) translateZ(100px);
		}
	
		.container .layer:nth-child(2){
		-webkit-transform:rotateY(180deg) translateZ(100px);
		}
	
		.container .layer:nth-child(3){
		-webkit-transform:rotateY(90deg) translateZ(100px);
		}
	
		.container .layer:nth-child(4){
		-webkit-transform:rotateY(270deg) translateZ(100px);
		}
	
		.container .layer:nth-child(5){
		-webkit-transform:rotateX(90deg) translateZ(100px);
		}
	
		.container .layer:nth-child(6){
		-webkit-transform:rotateX(270deg) translateZ(100px);
		}
	</style>
	</head>
	<body>
	<div class="stage">
		<div class="xdiv">
			<div class="container">
				<div class="layer">
					<div style="background-color:blue;">
					</div>
					<div style="background-color:blue;">
					</div>
					<div style="background-color:blue;">
					</div>
					<div style="background-color:blue;">
					</div>
				</div>
				<div class="layer" >
					<div style="background-color:green;">
					</div>
					<div style="background-color:green;">
					</div>
					<div style="background-color:green;">
					</div>
					<div style="background-color:green;">
					</div>
				</div>
				<div class="layer">
					<div style="background-color:red;">
					</div>
					<div style="background-color:red;">
					</div>			
					<div style="background-color:red;">
					</div>
					<div style="background-color:red;">
					</div>
				</div>
				<div class="layer">
					<div style="background-color:Darkorange;">
					</div>
					<div style="background-color:Darkorange;">
					</div>
					<div style="background-color:Darkorange;">
					</div>
					<div style="background-color:Darkorange;">
					</div>
				</div>
				<div class="layer" >
					<div style="background-color:yellow;">
					</div>
					<div style="background-color:yellow;">
					</div>
					<div style="background-color:yellow;">
					</div>
					<div style="background-color:yellow;">
					</div>
				</div>
				<div class="layer" >
					<div style="background-color:white;">
					</div>
					<div style="background-color:white;">
					</div>
					<div style="background-color:white;">
					</div>
					<div style="background-color:white;">
					</div>
				</div>
			</div>
		</div>
	</div>
	</body>
	</html>
```