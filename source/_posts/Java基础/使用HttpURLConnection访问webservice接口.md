---
title: Java知识点系列之--使用HttpURLConnection访问webservice接口
date: 2020-07-20 13:02:00
updated: 2020-07-20 13:02:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 如何使用HttpURLConnection通过Post方式访问webservice接口？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
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
本文以获取天气预报**webservice**接口为例，使用·HttpURLConnection·通过发送**SOAP**消息格式内容来请求**webservice**接口。

天气预报接口地址：<http://www.webxml.com.cn/WebServices/WeatherWebService.asmx?wsdl> 
```Java
	import java.io.BufferedReader;
	import java.io.InputStreamReader;
	import java.io.OutputStream;
	import java.net.HttpURLConnection;
	import java.net.URL;
	 
	public class HttpUtil {
	 
		public static String sendSoapPost(String url, String xml,
				String contentType, String soapAction) {
			String urlString = url;
			HttpURLConnection httpConn = null;
			OutputStream out = null;
			String returnXml = "";
			try {
				httpConn = (HttpURLConnection) new URL(urlString).openConnection();
				httpConn.setRequestProperty("Content-Type", contentType);
				if (null != soapAction) {
					httpConn.setRequestProperty("SOAPAction", soapAction);
				}
				httpConn.setRequestMethod("POST");
				httpConn.setDoOutput(true);
				httpConn.setDoInput(true);
				httpConn.connect();
				out = httpConn.getOutputStream(); // 获取输出流对象
				httpConn.getOutputStream().write(xml.getBytes("UTF-8")); // 将要提交服务器的SOAP请求字符流写入输出流
				out.flush();
				out.close();
				int code = httpConn.getResponseCode(); // 用来获取服务器响应状态
				String tempString = null;
				StringBuffer sb1 = new StringBuffer();
				if (code == HttpURLConnection.HTTP_OK) {
					BufferedReader reader = new BufferedReader(
							new InputStreamReader(httpConn.getInputStream(),
									"UTF-8"));
					while ((tempString = reader.readLine()) != null) {
						sb1.append(tempString);
					}
					if (null != reader) {
						reader.close();
					}
				} else {
					BufferedReader reader = new BufferedReader(
							new InputStreamReader(httpConn.getErrorStream(),
									"UTF-8"));
					// 一次读入一行，直到读入null为文件结束
					while ((tempString = reader.readLine()) != null) {
						sb1.append(tempString);
					}
					if (null != reader) {
						reader.close();
					}
				}
				// 响应报文
				returnXml = sb1.toString();
			} catch (Exception e) {
				e.printStackTrace();
			}
			return returnXml;
		}
	 
		public static void main(String[] args) {
			String url = "http://www.webxml.com.cn/WebServices/WeatherWebService.asmx";
	 
			StringBuilder sb = new StringBuilder("");
			sb.append(
					"<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" ")
					.append("xmlns:web=\"http://WebXml.com.cn/\"><soapenv:Header/><soapenv:Body>")
					.append("<web:getSupportProvince/></soapenv:Body></soapenv:Envelope>");
			/*
			 * sb.append(
			 * "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" "
			 * ) .append(
			 * "xmlns:web=\"http://WebXml.com.cn/\"><soapenv:Header/><soapenv:Body><web:getSupportCity>"
			 * ) .append(
			 * "<web:byProvinceName>河北</web:byProvinceName></web:getSupportCity></soapenv:Body></soapenv:Envelope>"
			 * );
			 */
			String dataXml = sb.toString();
			String contentType = "text/xml; charset=utf-8";
			String soapAction = "http://WebXml.com.cn/getSupportProvince";
			// String soapAction =
			// "\"document/http://pengjunnlee.com/CustomUI:GetWeatherById\"";
			String resultXml = HttpUtil.sendSoapPost(url, dataXml, contentType,
					soapAction);
			System.out.println(resultXml);
		}
	}
```
执行程序，控制台打印结果。

```Xml
<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><soap:Body><getSupportProvinceResponse xmlns="http://WebXml.com.cn/"><getSupportProvinceResult><string>直辖市</string><string>特别行政区</string><string>黑龙江</string><string>吉林</string><string>辽宁</string><string>内蒙古</string><string>河北</string><string>河南</string><string>山东</string><string>山西</string><string>江苏</string><string>安徽</string><string>陕西</string><string>宁夏</string><string>甘肃</string><string>青海</string><string>湖北</string><string>湖南</string><string>浙江</string><string>江西</string><string>福建</string><string>贵州</string><string>四川</string><string>广东</string><string>广西</string><string>云南</string><string>海南</string><string>新疆</string><string>西藏</string><string>台湾</string><string>亚洲</string><string>欧洲</string><string>非洲</string><string>北美洲</string><string>南美洲</string><string>大洋洲</string></getSupportProvinceResult></getSupportProvinceResponse></soap:Body></soap:Envelope>
```