---
title: Java工具类系列之--二进制文件与十六进制字符串转化
date: 2020-07-20 12:06:00
updated: 2020-07-20 12:06:00
tags: Java工具类
categories: Java工具类
keywords: Java, 工具类
type: 
description: 一个非常好用的二进制文件与十六进制字符串转化工具类。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img6.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img6.jpg
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
本工具类主要用来将二进制文件读取并转换成十六进制字符串，并提供了将十六进制字符串还原为二进制文件的方法。 
```Java
	import java.io.ByteArrayOutputStream;
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.InputStream;
	import java.nio.ByteBuffer;
	import java.nio.channels.FileChannel;
	 
	import org.apache.log4j.Logger;
	 
	public class HexStringUtils {
		private static final Logger logger = Logger.getLogger(HexStringUtils.class);
	 
		private static final char[] hexChars = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E',
				'F' };
	 
		private static final String hexString = "0123456789ABCDEF";
	 
		public static String toHexString(String filePath) throws FileNotFoundException, IOException {
			if (isEmptyStr(filePath)) {
				logger.error("invalid filePath parameter...");
			}
	 
			return toHexString(new File(filePath));
		}
	 
		public static String toHexString(File file) throws FileNotFoundException, IOException {
			if (null == file || !file.exists() || file.isDirectory()) {
				logger.error("invalid file parameter...");
			}
			return toHexString(new FileInputStream(file));
		}
	 
		public static String toHexString(InputStream inputStream) throws IOException {
			byte[] bytes = inputStream2ByteArray(inputStream);
			return toHexString(bytes);
		}
	 
		private static byte[] inputStream2ByteArray(InputStream in) throws IOException {
	 
			ByteArrayOutputStream out = new ByteArrayOutputStream();
			byte[] buffer = new byte[1024 * 4];
			int n = 0;
			while ((n = in.read(buffer)) != -1) {
				out.write(buffer, 0, n);
			}
			return out.toByteArray();
		}
	 
		/*
		 * Converts a byte array to hex string
		 */
		private static String toHexString(byte[] bytes) {
			StringBuffer buf = new StringBuffer();
	 
			int len = bytes.length;
	 
			for (int i = 0; i < len; i++) {
				byte2hex(bytes[i], buf);
			}
			return buf.toString();
		}
	 
		/*
		 * Converts a byte to hex digit and writes to the supplied buffer
		 */
		private static void byte2hex(byte b, StringBuffer buf) {
			int high = ((b & 0xf0) >> 4);
			int low = (b & 0x0f);
			buf.append(hexChars[high]);
			buf.append(hexChars[low]);
		}
	 
		/*
		 * Converts a hex string to byte array
		 */
		private static byte[] convert2bytes(String stringOfHex) {
			if (isEmptyStr(stringOfHex)) {
				logger.error("invalid hex string parameter...");
				return null;
			}
			stringOfHex = stringOfHex.trim().toUpperCase();
			if (stringOfHex.length() % 2 == 1)
				return null;
	 
			int length = stringOfHex.length() / 2;
			byte[] bytes = new byte[length];
			char[] charArray = stringOfHex.toCharArray();
			for (int i = 0; i < length; i++) {
				int pos = i * 2;
				bytes[i] = (byte) (charToByte(charArray[pos]) << 4 | charToByte(charArray[pos + 1]));
			}
			return bytes;
		}
	 
		/*
		 * Converts a character to byte
		 */
		private static byte charToByte(char c) {
			return (byte) hexString.indexOf(c);
		}
	 
		/*
		 * Converts a byte array to file
		 */
		@SuppressWarnings("resource")
		public static void writeBytes2File(String filePath, byte[] bytes) throws IOException {
			File outputFile = new File(filePath);
			File parentFile = outputFile.getParentFile();
			if (!parentFile.exists()) {
				parentFile.mkdirs();
			}
			ByteBuffer bb = ByteBuffer.wrap(bytes);
	 
			FileChannel fc = new FileOutputStream(outputFile).getChannel();
			fc.write(bb);
			fc.close();
		}
	 
		/*
		 * Determine whether a string is empty
		 */
		private static boolean isEmptyStr(String str) {
			return str == null || str.length() == 0;
		}
	 
		public static void main(String[] args) throws FileNotFoundException, IOException {
			// 获取字体文件3D.TTF的16进制字符串
			String filePath = "D:/3D.TTF";
			long start = System.currentTimeMillis();
			String hexStr = toHexString(filePath);
			System.out.println(hexStr);
			long end = System.currentTimeMillis();
			System.out.println(end - start);
	 
			// 将3D.TTF的16进制字符串还原为二进制文件
			String newFilePath = "D:/3D副本.TTF";
			start = System.currentTimeMillis();
			byte[] bytes = convert2bytes(hexStr);
			writeBytes2File(newFilePath, bytes);
			end = System.currentTimeMillis();
			System.out.println(end - start);
	 
		}
	}
```