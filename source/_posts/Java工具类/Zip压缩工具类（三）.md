---
title: Java工具类系列之--Zip压缩工具类（三）
date: 2020-07-20 12:04:00
updated: 2020-07-20 12:04:00
tags: Java工具类
categories: Java工具类
keywords: Java, 工具类
type: 
description: 一个可用于Zip压缩与解压缩的工具类。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
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
在**Java**项目中需要对文件夹的内容进行**Zip**压缩，参考了网上的代码并修复了里面的一些问题，例如：中文乱码、媒体文件解压后损坏。 

所使用的**Jar**包：`ant-*.*.*.jar` 和 `log4j-*.*.*.jar` ，示例工程的CSDN下载地址：<http://download.csdn.net/download/pengjunlee/10124740>。  

# ZipUtil工具类源码
```
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.InputStream;
	import java.util.Enumeration;
	 
	import org.apache.log4j.Logger;
	import org.apache.tools.zip.ZipEntry;
	import org.apache.tools.zip.ZipFile;
	import org.apache.tools.zip.ZipOutputStream;
	 
	public class ZipUtil {
	 
		private static final Logger logger = Logger.getLogger(ZipUtil.class);
	 
		/**
		 * 由于压缩方法是静态方法，无须创建工具类实例，故将构造方法声明为私有
		 */
		private ZipUtil() {
	 
		}
	 
		/**
		 * 压缩文件或目录
		 * 
		 * @param srcPath
		 *            要压缩的源文件或目录
		 * @param destPath
		 *            生成的压缩文件全名，这里是绝对路径
		 * @param encoding
		 *            压缩时采用的编码格式
		 */
		public static void zip(String srcPath, String destPath, String encoding){
			if (isEmptyStr(srcPath) || isEmptyStr(destPath)) {
				logger.error("invalid zip parameters...");
				return;
			}
			if (isEmptyStr(encoding)) {
				encoding = "UTF-8";
			}
			zip(new File(srcPath), new File(destPath), encoding);
		}
	 
		/**
		 * 压缩文件或目录
		 * 
		 * @param srcFile
		 *            目录或者单个文件
		 * @param destFile
		 *            压缩后的ZIP文件
		 * @param encoding
		 *            压缩时采用的编码格式
		 */
		public static void zip(File srcFile, File destFile, String encoding) {
			File parentFile = destFile.getParentFile();
			if (!parentFile.exists()) {
				parentFile.mkdirs();
			}
			ZipOutputStream out = null;
			try {
				out = new ZipOutputStream(new FileOutputStream(destFile));
				out.setEncoding(encoding);
				zip(srcFile, out);
			} catch (Exception e) {
				logger.error(e.getMessage());
			} finally {
				if (null != out) {
					try {
						out.close();// 记得关闭资源
					} catch (IOException e) {
						logger.error(e.getMessage());
					}
				}
			}
		}
	 
		/**
		 * 压缩文件或目录
		 * 
		 * @param srcFile
		 *            目录或者单个文件
		 * @param out
		 *            压缩文件输出流
		 */
		public static void zip(File srcFile, ZipOutputStream out) {
			zip(srcFile, out, "");
		}
	 
		/**
		 * 压缩文件或目录
		 * 
		 * @param srcFile
		 *            目录或者单个文件
		 * @param out
		 *            压缩文件输出流
		 * @param curPaht
		 *            正在压缩的当前文件与srcPath的相对路径
		 */
		public static void zip(File srcFile, ZipOutputStream out, String curPaht) {
			if (srcFile.isDirectory()) {
				File[] files = srcFile.listFiles();
				if (files != null && files.length > 0) {
					for (File file : files) {
						String name = srcFile.getName();
						if (!"".equals(curPaht)) {
							name = curPaht + File.separator + name;
						}
						ZipUtil.zip(file, out, name);
					}
				}
			} else {
				doZip(srcFile, out, curPaht);
			}
		}
	 
		/**
		 * 压缩文件
		 * 
		 * @param file
		 *            要压缩的文件
		 * @param out
		 *            压缩文件输出流
		 * @param curPath
		 *            正在压缩的当前文件与srcPath的相对路径
		 */
		public static void doZip(File file, ZipOutputStream out, String curPath) {
			String entryName = null;
			if (!"".equals(curPath)) {
				entryName = curPath + File.separator + file.getName();
			} else {
				entryName = file.getName();
			}
			ZipEntry entry = new ZipEntry(entryName);
			FileInputStream fis = null;
			try {
				out.putNextEntry(entry);
				fis = new FileInputStream(file);
				int len = 0;
				byte[] buffer = new byte[1024];
				while ((len = fis.read(buffer)) > 0) {
					out.write(buffer, 0, len);
					out.flush();
				}
				out.closeEntry();
			} catch (IOException e) {
				logger.error(e.getMessage());
			} finally {
				if (null != fis) {
					try {
						fis.close();
					} catch (IOException e) {
						logger.error(e.getMessage());
					}
				}
			}
	 
		}
	 
		/**
		 * 判断是否为空字符串
		 * 
		 * @param str
		 * @return
		 */
		private static boolean isEmptyStr(String str) {
			return str == null || str.length() == 0;
		}
	 
		/**
		 * 解压缩
		 * 
		 * @param zipFilePath
		 *            压缩文件路径
		 * @param outputDirectory
		 *            解压缩后文件存放的目录
		 * @param encoding
		 *            解压缩时采用的编码格式
		 */
		public static void unzip(String zipFilePath, String outputDirectory,String encoding){
			if (isEmptyStr(zipFilePath) || isEmptyStr(outputDirectory)) {
				logger.error("invalid unzip parameters...");
				return;
			}
			if (isEmptyStr(encoding)) {
				encoding = "UTF-8";
			}
			ZipFile zipFile=null;
			try {
				zipFile = new ZipFile(zipFilePath, encoding);
			} catch (IOException e) {
				logger.error(e.getMessage());
			}
			unzip(zipFile, outputDirectory,encoding);
		}
	 
		/**
		 * 解压缩
		 * 
		 * @param zipFile
		 *            源压缩文件
		 * @param outputDirectory
		 *            解压缩后文件存放目录
		 * @param encoding
		 *            解压缩时采用的编码格式
		 */
		@SuppressWarnings("rawtypes")
		private static void unzip(ZipFile zipFile, String outputDirectory,String encoding) {
			File outputFile=new File(outputDirectory);
			if (!outputFile.exists()) {
				outputFile.mkdirs();
			}
			try {
				Enumeration zipEntries = zipFile.getEntries();
				ZipEntry zipEntry = null;
	 
				while (zipEntries.hasMoreElements()) {
					zipEntry = (ZipEntry) zipEntries.nextElement();
	 
					String entryName = new String(zipEntry.getName().getBytes(
							encoding), encoding);
	 
					InputStream in = null;
					FileOutputStream out = null;
	 
					try {
						if (zipEntry.isDirectory()) {
							String name = zipEntry.getName();
							name = name.substring(0, name.length() - 1);
	 
							File f = new File(outputDirectory + File.separator
									+ name);
							f.mkdirs();
						} else {
							int index = entryName.lastIndexOf("\\");
							if (index != -1) {
								File df = new File(outputDirectory + File.separator
										+ entryName.substring(0, index));
								df.mkdirs();
							}
							index = entryName.lastIndexOf("/");
							if (index != -1) {
								File df = new File(outputDirectory + File.separator
										+ entryName.substring(0, index));
								df.mkdirs();
							}
	 
							File f = new File(outputDirectory + File.separator
									+ zipEntry.getName());
							// f.createNewFile();
							in = zipFile.getInputStream(zipEntry);
							out = new FileOutputStream(f);
	 
							int len;
							byte[] buffer = new byte[1024 * 1024];
	 
							while ((len = in.read(buffer)) != -1) {
								out.write(buffer, 0, len);
							}
							out.flush();
						}
					} catch (IOException e) {
						logger.error("解压失败...");
						logger.error(e.getMessage());
					} finally {
						if (null != in) {
							try {
								in.close();
							} catch (IOException e) {
								logger.error(e.getMessage());
							}
						}
						if (null != out) {
							try {
								out.close();
							} catch (IOException e) {
								logger.error(e.getMessage());
							}
						}
					}
				}
	 
			} catch (IOException e) {
				logger.error("解压失败...");
				logger.error(e.getMessage());
			} finally {
				if (null != zipFile) {
					try {
						zipFile.close();
					} catch (IOException e) {
						logger.error(e.getMessage());
					}
				}
			}
			
		}
		
		public static void main(String[] args) {
			ZipUtil.zip("D:\\sourceFolder", "D:\\targetFolder\\test.zip", "GBK");
			ZipUtil.unzip("D:\\targetFolder\\test.zip", "D:\\test", "GBK");
	 
		}
	 
	}
```