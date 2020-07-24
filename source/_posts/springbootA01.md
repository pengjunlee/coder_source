---
title: SpringBoot框架整合之--集成FastDFS（上）
date: 2020-07-24 14:01:00
updated: 2020-07-24 14:01:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot项目中如何集成FastDFS?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# 一、Maven依赖
```Xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <java.version>1.8</java.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
 
        <!-- 引入fastdfs客户端依赖 -->
        <dependency>
            <groupId>org.csource</groupId>
            <artifactId>fastdfs-client-java</artifactId>
            <version>1.27-SNAPSHOT</version>
        </dependency>
    </dependencies>
```
遇到 `fastdfs-client-java` 依赖无法引入的异常，可参考 [《解决 Maven 无法下载 fastdfs-client-java 依赖》](https://blog.csdn.net/pengjunlee/article/details/100929599 "《解决 Maven 无法下载 fastdfs-client-java 依赖》")。

# 二、FastDFS客户端配置

在 `src\main\resources` 新建一个FastDFS客户端配置文件（fdfs_client.conf），内容如下：
```Properties
	connect_timeout = 5000
	network_timeout = 30000
	charset = UTF-8
	 
	# Tracker配置文件 /etc/fdfs/tracker.conf 中配置的http端口
	http.tracker_http_port = 8080
	http.anti_steal_token = false
	http.secret_key = FastDFS1234567890
	 
	# Tracker服务器地址，可以写多个
	tracker_server = 172.16.250.238:22122
	#tracker_server = 172.16.250.240:22122
```

# 三、封装FdfsUtil工具类
```Java
	package com.pengjunlee.utils;
	 
	 
	import org.csource.fastdfs.*;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.core.io.ClassPathResource;
	 
	import java.io.*;
	 
	/**
	 * Fast DFS系统文件操作上传、下载客户端类
	 *
	 * @author pengjunlee
	 * @create 2019-09-17 10:25
	 */
	 
	public class FdfsUtil {
	 
	    private static Logger logger = LoggerFactory.getLogger(FdfsUtil.class);
	 
	    private static StorageClient1 client;
	 
	    static {
	        try {
	            String filePath = new ClassPathResource("fdfs_client.conf").getFile().getAbsolutePath();
	            ClientGlobal.init(filePath);
	            TrackerClient trackerClient = new TrackerClient();
	            TrackerServer trackerServer = trackerClient.getConnection();
	            StorageServer storageServer = trackerClient.getStoreStorage(trackerServer);
	            client = new StorageClient1(trackerServer, storageServer);
	        } catch (Exception e) {
	            logger.error("FastDFS init failure!");
	        }
	    }
	 
	    /**
	     * 上传一个本地文件到分布式中
	     *
	     * @param file
	     * @return
	     * @throws Exception
	     */
	    public static String upload(File file) throws Exception {
	        return upload(new FileInputStream(file), file.getName(), file.length());
	    }
	 
	    /**
	     * 上传一个流，调用后in会自动关闭
	     *
	     * @param in
	     * @param fileName
	     * @param fileSize
	     * @return
	     * @throws Exception
	     */
	    public static String upload(InputStream in, String fileName, long fileSize) throws Exception {
	        try {
	            String extName = fileExt(fileName);
	            String ret = client.upload_appender_file1(null, fileSize, new UploadStream(in, fileSize), extName, null);
	            if (ret == null) {
	                throw new IOException("upload fileName:" + fileName + " error");
	            }
	            return ret;
	        } finally {
	            in.close();
	        }
	    }
	 
	    /**
	     * 把本地文件追加到分布式文件系统中
	     *
	     * @param file
	     * @param dfsPath
	     * @return
	     * @throws Exception
	     */
	    public static long append(File file, String dfsPath) throws Exception {
	        return append(new FileInputStream(file), dfsPath, file.length());
	    }
	 
	    /**
	     * 追加文件，in会被关闭
	     *
	     * @param in
	     * @param dfsPath
	     * @param appendSize
	     * @return
	     * @throws Exception
	     */
	    public static long append(InputStream in, String dfsPath, long appendSize) throws Exception {
	        try {
	            int ret = client.append_file1(dfsPath, appendSize, new UploadStream(in, appendSize));
	            if (ret != 0) {
	                throw new IOException("uploadAppend dfsPath:" + dfsPath + " error");
	            }
	            return appendSize;
	        } finally {
	            in.close();
	        }
	    }
	 
	    /**
	     * 下载文件,方法调用后，out会被强制关闭
	     *
	     * @param out
	     * @param dfsPath
	     * @return
	     * @throws Exception
	     */
	    public static long download(OutputStream out, String dfsPath) throws Exception {
	        try {
	            DownloadStream downloadStream = new DownloadStream(out);
	            int ret = client.download_file1(dfsPath, downloadStream);
	            if (ret != 0) {
	                throw new IOException("download dfsPath:" + dfsPath + " error");
	            }
	            return downloadStream.downloadSize;
	        } finally {
	            out.close();
	        }
	    }
	 
	    /**
	     * 下载文件,方法调用后，out会被强制关闭
	     *
	     * @param out
	     * @param dfsPath
	     * @return
	     * @throws Exception
	     */
	    public static long download(OutputStream out, String dfsPath, long offset, long bytes) throws Exception {
	        try {
	            DownloadStream downloadStream = new DownloadStream(out);
	            int ret = client.download_file1(dfsPath, offset, bytes, downloadStream);
	            if (ret != 0) {
	                throw new IOException("download dfsPath:" + dfsPath + " error");
	            }
	            return downloadStream.downloadSize;
	        } finally {
	            out.close();
	        }
	    }
	 
	    /**
	     * 删除指定位置文件
	     *
	     * @param dfsPath
	     * @throws Exception
	     */
	    public static void delete(String dfsPath) throws Exception {
	        int ret = client.delete_file1(dfsPath);
	        if (ret != 0)
	            throw new IOException("delete dfsPath:" + dfsPath + " error");
	    }
	 
	    /**
	     * 获取文件大小；返回负数表示文件不存在，其中-1表示正常情况文件不存在，-2表示出异常了
	     *
	     * @param dfsPath
	     * @return
	     * @throws Exception
	     */
	    public static long length(String dfsPath) {
	        try {
	            FileInfo fileInfo = client.get_file_info1(dfsPath);
	            if (fileInfo == null) {
	                return -1;
	            }
	            return fileInfo.getFileSize();
	        } catch (Throwable ex) {
	            return -2;
	        }
	    }
	 
	    /**
	     * 判断Fast是否存在指定路径的文件信息
	     *
	     * @param dfsPath
	     * @return
	     * @throws Exception
	     */
	    public static boolean exists(String dfsPath) throws Exception {
	        if (dfsPath == null || dfsPath.trim().length() == 0) {
	            return false;
	        }
	 
	        try {
	            FileInfo fileInfo = client.get_file_info1(dfsPath);
	            return fileInfo != null;
	        } catch (Exception e) {
	            return false;
	        }
	    }
	 
	    /**
	     * 从路径中提取文件后缀名，不带句号
	     *
	     * @param fileName
	     * @return
	     */
	    private static String fileExt(String fileName) {
	        if (fileName == null)
	            return "";
	 
	        int dot = fileName.lastIndexOf(".");
	        if (dot == -1)
	            return "";
	 
	        String ret = fileName.substring(dot + 1);
	        for (int i = 0; i < ret.length(); i++) {
	            char c = ret.charAt(i);
	            boolean isAlpha = ('0' <= c && c <= '9') || ('a' <= c && c <= 'z') || ('A' <= c && c <= 'Z');
	            if (!isAlpha) {
	                return "";
	            }
	        }
	        return ret;
	    }
	 
	    private static class DownloadStream implements DownloadCallback {
	 
	        private long downloadSize = 0L;
	 
	        private OutputStream out;
	 
	        public DownloadStream(OutputStream out) {
	            this.out = out;
	        }
	 
	        public int recv(long fileSize, byte[] data, int bytes) {
	            try {
	                this.out.write(data, 0, bytes);
	            } catch (IOException ex) {
	                throw new RuntimeException(ex);
	            }
	            this.downloadSize += bytes;
	            return 0;
	        }
	    }
	}
```

# 四、Junit测试
```Java
	import com.pengjunlee.utils.FdfsUtil;
	import org.junit.Test;
	 
	import java.io.File;
	import java.io.FileNotFoundException;
	import java.io.FileOutputStream;
	 
	public class FdfsTests {
	 
	    /**
	     * 文件上传测试
	     */
	    @Test
	    public void testUpload() {
	        File file = new File("C:\\1.jpg");
	        try {
	            String fdfsPath = FdfsUtil.upload(file);
	            System.out.println("文件上传成功，文件ID： " + fdfsPath); // 文件ID： group1/M00/00/00/rBD67l2BM0mEP2WXAAAAABISex8406.jpg
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	 
	    }
	 
	    /**
	     * 文件下载测试
	     */
	    @Test
	    public void testDownload() throws FileNotFoundException {
	        try {
	            long downloadSize = FdfsUtil.download(new FileOutputStream(new File("C:\\1_bak.jpg")), "group1/M00/00/00/rBD67l2BM0mEP2WXAAAAABISex8406.jpg");
	            System.out.println("文件下载完成，文件大小（单位 byte）： " + downloadSize);
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	 
	    /**
	     * 文件删除测试
	     */
	    @Test
	    public void testDelete() {
	        try {
	            FdfsUtil.delete("group1/M00/00/00/rBD67l2BM0mEP2WXAAAAABISex8406.jpg");
	            boolean exists = FdfsUtil.exists("group1/M00/00/00/rBD67l2BM0mEP2WXAAAAABISex8406.jpg");
	            System.out.println("文件是否还存在： " + exists);
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	}
```