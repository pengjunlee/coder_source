---
title: SpringBoot框架整合之--集成FastDFS（下）
date: 2020-07-24 14:02:00
updated: 2020-07-24 14:02:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot项目中如何集成FastDFS?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
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
            <groupId>com.luhuiguo</groupId>
            <artifactId>fastdfs-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
    </dependencies>
```

# 二、application.yml配置

在核心配置文件中增加如下配置：
```Properties
	fdfs:
	  connect-timeout: 2000
	  so-timeout: 3000
	  tracker-list:
	    - 172.16.250.238:22122
```
    
# 三、Junit测试
```Java
	import com.luhuiguo.fastdfs.domain.FileInfo;
	import com.luhuiguo.fastdfs.domain.StorePath;
	import com.luhuiguo.fastdfs.service.FastFileStorageClient;
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.test.context.junit4.SpringRunner;
	 
	import javax.imageio.stream.FileImageOutputStream;
	import java.io.*;
	 
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class FdfsTests {
	 
	    @Autowired
	    private FastFileStorageClient storageClient;
	 
	    /**
	     * 文件上传测试
	     */
	    @Test
	    public void testUpload() {
	 
	        File file = new File("C:\\1.jpg");
	        try {
	            StorePath storePath = storageClient.uploadFile(new FileInputStream(file), file.length(), file.getName(), null);
	            System.out.println("文件上传成功，文件ID： " + storePath); // 文件ID： StorePath [group=group1, path=M00/00/00/rBD67l2BR5iABIjjAABB4RISex87.1.jpg]
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }
	 
	    /**
	     * 文件下载测试
	     */
	    @Test
	    public void testDownload() throws FileNotFoundException {
	        try {
	            byte[] bytes = storageClient.downloadFile("group1", "M00/00/00/rBD67l2BR5iABIjjAABB4RISex87.1.jpg");
	            FileImageOutputStream imageOutput = new FileImageOutputStream(new File("C:\\1_bak.jpg"));
	            imageOutput.write(bytes, 0, bytes.length);
	            imageOutput.close();
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
	            FileInfo fileInfo = storageClient.queryFileInfo("group1", "M00/00/00/rBD67l2BR5iABIjjAABB4RISex87.1.jpg");
	            System.out.println("文件大小（单位 byte）： " + fileInfo.getFileSize());
	            storageClient.deleteFile("group1", "M00/00/00/rBD67l2BR5iABIjjAABB4RISex87.1.jpg");
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	 
	}
```