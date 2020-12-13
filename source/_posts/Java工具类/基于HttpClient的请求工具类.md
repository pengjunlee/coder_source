---
title: Java工具类系列之--基于HttpClient的请求工具类
date: 2020-07-20 12:01:00
updated: 2020-07-20 12:01:00
tags: Java工具类
categories: Java工具类
keywords: Java, 工具类
type: 
description: 如何使用HttpClient发送GET和POST请求？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
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
# 简介

**HTTP** 协议可能是现在 **Internet**上使用得最多、最重要的协议了，越来越多的 Java 应用程序需要直接通过 **HTTP** 协议来访问网络资源。

在 JDK 的 `java.net` 包中已经提供了访问 **HTTP** 协议的基本功能，我们可以使用该包中的 `URLConnection` 类来发送**GET**和**POST**请求，但是对于大部分应用程序来说，**JDK** 库本身提供的功能还不够丰富和灵活。

`HttpClient` 是 **Apache Jakarta Common** 下的子项目，用来提供高效的、最新的、功能丰富的支持 **HTTP** 协议的客户端编程工具包，并且它支持 **HTTP** 协议最新的版本和建议。

现在 `HttpClient` 最新版本为 **HttpClient 4.5 (GA) （2015-09-11）**，下载地址：<http://hc.apache.org/downloads.cgi> 。

`HttpClient` 提供了以下主要的功能：

- 实现了所有 HTTP 的方法（GET,POST,PUT,HEAD 等）
- 支持自动转向
- 支持 HTTPS 协议
- 支持代理服务器等  

# 使用方法

使用 HttpClient 发送一个HTTP请求一般需要以下 四 个步骤：

1. 创建 HttpClient 的实例。
2. 创建请求方法的实例，并指定请求URL，如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
3. 调用创建好的 HttpClient 实例的HttpClient.execute(HttpUriRequest request) 方法来执行请求方法，该方法会返回一个封装了服务器响应信息的HttpResponse对象。
4. 释放连接，无论执行方法是否成功，都必须释放连接。  

# 封装工具类
```Java
	import java.io.IOException;
	import java.io.UnsupportedEncodingException;
	import java.security.KeyManagementException;
	import java.security.KeyStoreException;
	import java.security.NoSuchAlgorithmException;
	import java.security.cert.CertificateException;
	import java.security.cert.X509Certificate;
	import java.util.ArrayList;
	import java.util.List;
	 
	import javax.net.ssl.SSLContext;
	 
	import org.apache.http.Header;
	import org.apache.http.HttpEntity;
	import org.apache.http.HttpResponse;
	import org.apache.http.HttpStatus;
	import org.apache.http.client.ClientProtocolException;
	import org.apache.http.client.config.RequestConfig;
	import org.apache.http.client.entity.UrlEncodedFormEntity;
	import org.apache.http.client.methods.CloseableHttpResponse;
	import org.apache.http.client.methods.HttpGet;
	import org.apache.http.client.methods.HttpPost;
	import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
	import org.apache.http.conn.ssl.TrustStrategy;
	import org.apache.http.entity.StringEntity;
	import org.apache.http.impl.client.CloseableHttpClient;
	import org.apache.http.impl.client.HttpClients;
	import org.apache.http.message.BasicHeader;
	import org.apache.http.message.BasicNameValuePair;
	import org.apache.http.ssl.SSLContextBuilder;
	import org.apache.http.util.EntityUtils;
	import org.apache.log4j.Logger;
	 
	/**
	 * Http请求工具类
	 * 
	 * @author pengjunlee
	 */
	public class HttpUtil
	{
	 
	    private static Logger log = Logger.getLogger(HttpUtil.class);
	 
	    /**
	     * 发送GET请求
	     * @param isHttps    是否https
	     * @param url        请求地址
	     * @return           响应结果
	     */
	    public static String get(boolean isHttps, String url)
	    {
	        CloseableHttpClient httpClient = null;
	        try
	        {
	            if (!isHttps)
	            {
	                httpClient = HttpClients.createDefault();
	            }
	            else
	            {
	                httpClient = createSSLInsecureClient();
	            }
	            HttpGet httpget = new HttpGet(url);
	            // HttpGet设置请求头的两种种方式
	            // httpget.addHeader(new BasicHeader("Connection", "Keep-Alive"));
	            // httpget.addHeader("Connection", "Keep-Alive");
	            Header[] heads = httpget.getAllHeaders();
	            for (int i = 0; i < heads.length; i++)
	            {
	                System.out.println(heads[i].getName() + "-->" + heads[i].getValue());
	            }
	            CloseableHttpResponse response = httpClient.execute(httpget);
	 
	            // 判断状态行
	            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK)
	            {
	                HttpEntity entity = response.getEntity();
	                if (entity != null)
	                {
	                    String out = EntityUtils.toString(entity, "UTF-8");
	                    return out;
	                }
	            }
	        }
	        catch (ClientProtocolException e)
	        {
	            e.printStackTrace();
	            return null;
	        }
	        catch (IOException e)
	        {
	            e.printStackTrace();
	            return null;
	        }
	        finally
	        {
	            try
	            {
	                if (null != httpClient)
	                {
	                    httpClient.close();
	                }
	            }
	            catch (IOException e)
	            {
	                log.error("httpClient.close()异常");
	            }
	        }
	        return null;
	    }
	 
	    /**
	     * 发送POST请求
	     * @param isHttps       是否https
	     * @param url           请求地址
	     * @param data          请求实体内容 
	     * @param contentType   请求实体内容的类型
	     * @return              响应结果
	     */
	    public static String post(boolean isHttps, String url, String data, String contentType)
	    {
	        CloseableHttpClient httpClient = null;
	        try
	        {
	            if (!isHttps)
	            {
	                httpClient = HttpClients.createDefault();
	            }
	            else
	            {
	                httpClient = createSSLInsecureClient();
	            }
	            HttpPost httpPost = new HttpPost(url);
	 
	            // HttpPost设置请求头的两种种方式
	            //httpPost.addHeader(new BasicHeader("Connection", "Keep-Alive"));
	            //httpPost.addHeader("Connection", "Keep-Alive");
	            // UrlEncodedFormEntity处理键值对格式请求参数
	            //List<BasicNameValuePair> list = new ArrayList<BasicNameValuePair>();
	            //new UrlEncodedFormEntity(list, "UTF-8");
	 
	            if (null != data)
	            {
	                // StringEntity处理任意格式字符串请求参数
	                StringEntity stringEntity = new StringEntity(data, "UTF-8");
	                stringEntity.setContentEncoding("UTF-8");
	                if (null != contentType)
	                {
	                    stringEntity.setContentType(contentType);
	                }
	                else
	                {
	                    stringEntity.setContentType("application/json");
	                }
	                httpPost.setEntity(stringEntity);
	            }
	 
	            // 设置请求和传输超时时间
	            RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(2000).setConnectTimeout(2000).build();
	            httpPost.setConfig(requestConfig);
	 
	            HttpResponse response = httpClient.execute(httpPost);
	            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK)
	            {
	                HttpEntity entity = response.getEntity();
	                if (entity != null)
	                {
	                    String out = EntityUtils.toString(entity, "UTF-8");
	                    return out;
	                }
	            }
	        }
	        catch (UnsupportedEncodingException e)
	        {
	            log.error(e);
	        }
	        catch (ClientProtocolException e)
	        {
	            e.printStackTrace();
	            log.error("连接超时：" + url);
	        }
	        catch (IOException e)
	        {
	            e.printStackTrace();
	            log.error("IO异常:" + url);
	        }
	        finally
	        {
	            try
	            {
	                if (null != httpClient)
	                {
	                    httpClient.close();
	                }
	            }
	            catch (IOException e)
	            {
	                log.error("httpClient.close()异常");
	            }
	        }
	        return null;
	    }
	 
	    /**
	     * Https请求对象，信任所有证书
	     * 
	     * @return CloseableHttpClient
	     */
	    public static CloseableHttpClient createSSLInsecureClient()
	    {
	        try
	        {
	            SSLContext sslContext = new SSLContextBuilder().loadTrustMaterial(null, new TrustStrategy()
	            {
	                // 信任所有
	                public boolean isTrusted(X509Certificate[] chain, String authType) throws CertificateException
	                {
	                    return true;
	                }
	            }).build();
	            SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext);
	            return HttpClients.custom().setSSLSocketFactory(sslsf).build();
	        }
	        catch (KeyManagementException e)
	        {
	            e.printStackTrace();
	        }
	        catch (NoSuchAlgorithmException e)
	        {
	            e.printStackTrace();
	        }
	        catch (KeyStoreException e)
	        {
	            e.printStackTrace();
	        }
	        return HttpClients.createDefault();
	    }
	 
	    public static void main(String[] args)
	    {
	        String restu = get(true,
	                "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=wx169c72abdb70cdab&secret=55a4b77eda664e183881e5ed9cce189a");
	        System.out.println(restu);
	    }
	}
```