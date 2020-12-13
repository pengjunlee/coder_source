---
title: Java知识点系列之--获取302响应中的Location头信息
date: 2020-07-20 13:24:00
updated: 2020-07-20 13:24:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: Java中如何通过HttpClient获取302响应中的Location头信息?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img24.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img24.jpg
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
# HttpClient获取302响应中的Location头信息
```Java
	public static String getLocationUrl(String url) {
		RequestConfig config = RequestConfig.custom().setConnectTimeout(50000).setConnectionRequestTimeout(10000).setSocketTimeout(50000)
                .setRedirectsEnabled(false).build();//不允许重定向   
		CloseableHttpClient httpClient = HttpClients.custom().setDefaultRequestConfig(config).build(); 
		String location = null;
		int responseCode = 0;
 
		HttpResponse response;
		try {
			response = httpClient.execute(new HttpGet(url));
			responseCode = response.getStatusLine().getStatusCode();
			if (responseCode == 302) {
				Header locationHeader = response.getFirstHeader("Location");
				location = locationHeader.getValue();
			}
		} catch (Exception e) {
			
			e.printStackTrace();
		}
		
		return location;
	}
```