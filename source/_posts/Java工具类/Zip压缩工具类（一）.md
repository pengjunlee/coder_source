---
title: Java工具类系列之--Zip压缩工具类（一）
date: 2020-07-20 12:02:00
updated: 2020-07-20 12:02:00
tags: Java工具类
categories: Java工具类
keywords: Java, 工具类
type: 
description: 一个可用于Zip压缩与解压缩的工具类。
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
在**Java**项目中需要对文件夹的内容进行**Zip**压缩，参考了网上的代码并修复了里面的一些问题，例如：中文乱码、媒体文件解压后损坏。

所使用的**Jar**包：`ant-*.*.*.jar` 和 `log4j-*.*.*.jar` ，示例工程的CSDN下载地址：<http://download.csdn.net/download/pengjunlee/10043076>。

# ZipUtil工具类源码
```Java
	import java.io.BufferedInputStream;
	import java.io.BufferedOutputStream;
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.OutputStream;
	import java.util.ArrayList;
	import java.util.Enumeration;
	import java.util.List;
	 
	import org.apache.log4j.Logger;
	import org.apache.tools.zip.ZipEntry;
	import org.apache.tools.zip.ZipFile;
	import org.apache.tools.zip.ZipOutputStream;
	 
	/* 
	 * ZIP压缩工具类 
	 */
	public class ZipUtil {
	 
		private static final Logger logger = Logger.getLogger(ZipUtil.class);
	 
		/**
		 * 打包指定目录下所有文件，包括子文件夹
		 * 
		 * @param sourceFolder
		 *            要打包的文件目录
		 * @param targetFolder
		 *            生成的压缩文件存放目录
		 * @param zipFileName
		 *            生成的压缩文件名
		 * @param encoding
		 *            压缩时采用的编码格式
		 * @throws IOException
		 */
		public static void zipFolder(String sourceFolder, String targetFolder,
				String zipFileName, String encoding) {
			if (isEmptyStr(sourceFolder) || isEmptyStr(targetFolder)
					|| isEmptyStr(zipFileName)) {
				logger.error("invalid compress parameters...");
				return;
			}
			sourceFolder = formatFilePath(sourceFolder);
			targetFolder = formatFilePath(targetFolder);
			List<String> fileList = generatefileList(sourceFolder);
			zipfileList(fileList, sourceFolder, targetFolder, zipFileName, encoding);
		}
	 
		/**
		 * 打包指定的文件。要打包的文件在文件列表中指定
		 * 
		 * @param fileList
		 *            要打包的文件列表，这些文件是相对路径
		 * @param sourceFolder
		 *            要打包的文件所在目录的绝对路径
		 * @param targetFolder
		 *            生成的压缩文件存放的目录
		 * @param zipFileName
		 *            生成的压缩文件名
		 * @param encoding
		 *            压缩时采用的编码格式
		 * @throws IOException
		 */
		public static void zipfileList(List<String> fileList, String sourceFolder,
				String targetFolder, String zipFileName, String encoding) {
			if (fileList == null || fileList.isEmpty()) {
				logger.error("no files to be zip. fileList is null or empty...");
				return;
			}
			if (isEmptyStr(targetFolder) || isEmptyStr(zipFileName)) {
				logger.error("targetFolder and zipFileName are unspecified...");
				return;
			}
	 
			sourceFolder = formatFilePath(sourceFolder);
			targetFolder = formatFilePath(targetFolder);
			if (isEmptyStr(encoding)) {
				encoding = "UTF-8";
			}
	 
			File outputDir = new File(targetFolder);
			if (!outputDir.exists()) {
				outputDir.mkdirs();
			}
	 
			byte[] buffer = new byte[1024 * 1024];
	 
			String outputFullFileName = (targetFolder + "/" + zipFileName);
			ZipOutputStream zos = null;
			try {
				zos = new ZipOutputStream(new FileOutputStream(outputFullFileName));
				zos.setEncoding(encoding);
				logger.info("Output to Zip : " + outputFullFileName);
				for (String file : fileList) {
					if (isEmptyStr(file)) {
						continue;
					}
					logger.info("File Added : " + file);
					ZipEntry ze = new ZipEntry(file); // 这里用的是相对路径
					zos.putNextEntry(ze);
	 
					FileInputStream in = new FileInputStream(sourceFolder + "/" + file); // 这里是绝对路径
	 
					int len;
					while ((len = in.read(buffer)) > 0) {
						zos.write(buffer, 0, len);
					}
					in.close();
				}
				zos.closeEntry();
				logger.info("Compress Done!!!");
			} catch (IOException e) {
				logger.error("Compress Wrong!!!");
			} finally {
				// remember close it
				if (null != zos) {
					try {
						zos.close();
					} catch (IOException e) {
						logger.error(e.getMessage());
					}
				}
			}
	 
		}
	 
		/**
		 * 获取指定目录下的文件列表，包含子文件夹下的文件，忽略空文件夹
		 * 
		 * @param fileFolder
		 *            文件或目录
		 * @return 目录下的文件列表
		 */
		public static List<String> generatefileList(String fileFolder) {
			List<String> fileList = null;
			if (!isEmptyStr(fileFolder)) {
				fileFolder = formatFilePath(fileFolder);
				fileList = new ArrayList<String>();
				File node = new File(fileFolder);
				generateFileListHelper(fileFolder, node, fileList);
			}
			return fileList;
		}
	 
		private static void generateFileListHelper(String sourceFolder, File node,
				List<String> fileList) {
	 
			// add file only
			if (node.isFile()) {
				String absoluteFile = node.getAbsoluteFile().toString();
				String filepath = generateZipEntry(sourceFolder, absoluteFile);
				fileList.add(filepath);
			}
	 
			if (node.isDirectory()) {
				String[] subNodes = node.list();
				for (String subNode : subNodes) {
					File subFile = new File(node, subNode);
					generateFileListHelper(sourceFolder, subFile, fileList);
				}
			}
	 
		}
	 
		/**
		 * 格式化被压缩文件的路径,删除文件的sourceFolder路径 例如： d:\ziptest\tmpty.txt --> tmpty.txt
		 * d:\ziptest\sub\t.xls --> sub\t.xls
		 * 
		 * @param file
		 *            被压缩文件，这里是绝对路径
		 * @return 格式化后的文件路径，这里是与sourceFolder的相对路径
		 */
		private static String generateZipEntry(String sourceFolder, String file) {
			logger.debug("sourceFolder=" + sourceFolder);
			logger.debug("file=" + file);
			String formattedPath = file.substring(sourceFolder.length() + 1);
			formattedPath = formatFilePath(formattedPath);
			logger.debug("formattedPath=" + formattedPath);
			return formattedPath;
		}
	 
		/**
		 * 将文件路径中的分隔符"\"转换成"/",并去掉最后的分隔符（如果有）
		 * 
		 * @param filePath
		 *            文件路径
		 * @return 格式化后的文件路径
		 */
		public static String formatFilePath(String filePath) {
			if (filePath != null && filePath.length() != 0) {
				filePath = filePath.replaceAll("\\\\", "/");
			}
			if (filePath.endsWith("/")) {
				filePath = filePath.substring(0, filePath.length() - 1);
			}
			return filePath;
		}
	 
		private static boolean isEmptyStr(String str) {
			return str == null || str.length() == 0;
		}
	 
		/**
		 * 解压zip文件，支持子文件夹和中文
		 * 
		 * @param zipFileFullName
		 *            要解压的zip文件，这里是绝对路径
		 * @param targetFolder
		 *            解压到指定目录。这里是绝对路径，为空则默认解压到压缩文件所在目录
		 * @param encoding
		 *            解压时采用的编码格式
		 */
		@SuppressWarnings("rawtypes")
		public static void unzip(String zipFileFullName, String targetFolder,
				String encoding) {
			if (zipFileFullName == null || zipFileFullName.length() == 0) {
				return;
			}
			if (!zipFileFullName.endsWith(".zip")) {
				logger.error(zipFileFullName + " is not a zip file...");
				return;
			}
			if (isEmptyStr(encoding)) {
				encoding = "UTF-8";
			}
			// 将文件路径中的分隔符"\"转换成"/"
			zipFileFullName = zipFileFullName.replaceAll("\\\\", "/");
			// 获取压缩文件所在目录
			String zipFolder = zipFileFullName.replaceAll("/[^/]+\\.zip", "");
			if (targetFolder == null || targetFolder.length() == 0) {
				targetFolder = zipFolder;
			}
			targetFolder = targetFolder.replaceAll("\\\\", "/");
	 
			File targetFolderFile = new File(targetFolder);
			if (!targetFolderFile.exists()) {
				targetFolderFile.mkdirs();
			}
			OutputStream outStream = null;
			InputStream inStream = null;
			try {
				ZipFile zip = new ZipFile(zipFileFullName, encoding);
				Enumeration zipFileEntries = zip.getEntries();
	 
				while (zipFileEntries.hasMoreElements()) {
					ZipEntry entry = (ZipEntry) zipFileEntries.nextElement();
					String entryName = entry.getName();
					logger.debug("Extracting,entryName=" + entryName);
	 
					int lastSlashPos = entryName.lastIndexOf("/");
					/*
					 * 用本程序中ZipUtil.zipFolder或者ZipUtil.zipfileList生成的zip文件，
					 * 如果有子文件夹，entry.getName()会直接得到子文件夹中的文件而略过子文件夹
					 * 所以这里需要先生成文件所在的各级子文件夹目录
					 */
					if (lastSlashPos != -1) {
						String folderStr = targetFolder + "/"
								+ entryName.substring(0, lastSlashPos);
						File folder = new File(folderStr);
						if (!folder.exists()) {
							folder.mkdirs();
						}
					}
					if (!entryName.endsWith("/")) {
						File outFile = new File(targetFolder + "/" + entryName);
						outStream = new BufferedOutputStream(new FileOutputStream(
								outFile));
						byte[] buffer = new byte[1024 * 1024];
						inStream = new BufferedInputStream(
								zip.getInputStream(entry));
						int len;
						while ((len = inStream.read(buffer)) > 0) {
							outStream.write(buffer, 0, len);
						}
						outStream.flush();
	 
					}
				}
	 
			} catch (Exception e) {
				logger.error("errors occur when decompressing...");
				e.printStackTrace();
			} finally {
				if (null != outStream) {
					try {
						outStream.close();
					} catch (IOException e) {
						logger.error(e.getMessage());
					}
				}
				if (null != inStream) {
					try {
						inStream.close();
					} catch (IOException e) {
						logger.error(e.getMessage());
					}
				}
	 
			}
		}
	 
		public static void main(String[] argv) {
	 
			String sourceFolder = "D:\\sourceFolder";// 要打包压缩的文件夹
			String targetFolder = "D:\\targetFolder";
			String outputFileName = "test.zip";
			zipFolder(sourceFolder, targetFolder, outputFileName, "GB2312");
	 
			String zipFileFullName = "D:\\targetFolder\\test.zip";
			unzip(zipFileFullName, "D:\\test", "GB2312");
	 
		}
	}
```