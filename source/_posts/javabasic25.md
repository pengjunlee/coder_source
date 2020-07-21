---
title: Java知识点系列之--使用net.sf.json遍历Json数组
date: 2020-07-20 13:25:00
updated: 2020-07-20 13:25:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: Java中如何使用net.sf.json遍历Json数组?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img25.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img25.jpg
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
# 使用net.sf.json遍历Json数组
```Java
	import org.junit.Test;
	import java.util.Iterator;
	import net.sf.json.JSONArray;
	import net.sf.json.JSONObject;
	 
	public class JsonArrayTest {
	 
		@SuppressWarnings("unchecked")
		@Test
		public void test1() {
			String arrStr = "[{key:'a',value:'1'},{key:'b',value:'2'},{key:'c',value:'3'}]";
			JSONArray jsonArray = JSONArray.fromObject(arrStr);
			for (int i = 0; i < jsonArray.size(); i++) {
				JSONObject jsonObj = jsonArray.getJSONObject(i);
				Iterator<String> iterator = jsonObj.keys();
				while (iterator.hasNext()) {
					String key = iterator.next();
					System.out.println(key + ":" + jsonObj.getString(key) + " ");
				}
			}
		}
	 
	}
```