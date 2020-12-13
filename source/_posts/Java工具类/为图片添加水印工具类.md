---
title: Java工具类系列之--为图片添加水印工具类
date: 2020-07-20 12:05:00
updated: 2020-07-20 12:05:00
tags: Java工具类
categories: Java工具类
keywords: Java, 工具类
type: 
description: 一个非常好用可以给图片添加上水印的工具类。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
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
在实际项目开发过程中经常会需要给图片添加上水印，俗话说的好：“好记性不如烂笔头“，在此对其实现方法做下笔记。 
```Java
	package com.pengjunlee.watermark;
	 
	import java.awt.AlphaComposite;
	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics2D;
	import java.awt.Image;
	import java.awt.RenderingHints;
	import java.awt.image.BufferedImage;
	import java.io.File;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.OutputStream;
	import java.util.logging.Level;
	import java.util.logging.Logger;
	 
	import javax.imageio.ImageIO;
	 
	import com.sun.image.codec.jpeg.JPEGCodec;
	import com.sun.image.codec.jpeg.JPEGImageEncoder;
	 
	/**
	 * 为图片添加水印
	 * 
	 * @author pengjunlee
	 *
	 */
	public class WaterMarkUtils {
	 
		private static final Logger logger = Logger.getLogger("watermark");
	 
		// 水印透明度
		private static Float ALPHA_NONE = 0.5f;
		// 水印文字字体
		private static Font FONT_SONG = new Font("宋体", Font.BOLD, 28);
		// 水印文字颜色
		private static Color COLOR_BLACK = Color.BLACK;
	 
		/**
		 * 为图片添加文本水印
		 * 
		 * @param srcImgPath
		 *            源图片路径
		 * @param targetImgPath
		 *            目标图片路径
		 * @param text
		 *            水印文字
		 * @param font
		 *            水印文字字体
		 * @param color
		 *            水印文字颜色
		 * @param alpha
		 *            水印透明度
		 * @param degree
		 *            水印旋转角度
		 */
		public static void addTextWaterMark(String srcImgPath, String targetImgPath, String text, Font font, Color color,
				Float alpha, Integer degree) {
	 
			if (isEmptyStr(srcImgPath) || isEmptyStr(targetImgPath)) {
				logger.log(Level.WARNING, "invalid watermark file path parameters...");
				return;
			}
	 
			File srcImgFile = new File(srcImgPath);
			File targetImgFile = new File(targetImgPath);
			addTextWaterMark(srcImgFile, targetImgFile, text, font, color, alpha, degree);
		}
	 
		/**
		 * 为图片添加文本水印
		 * 
		 * @param srcImgFile
		 *            源图片文件
		 * @param targetImgFile
		 *            目标图片文件
		 * @param text
		 *            水印文字
		 * @param font
		 *            水印文字字体
		 * @param color
		 *            水印文字颜色
		 * @param alpha
		 *            水印透明度
		 * @param degree
		 *            水印旋转角度
		 */
		public static void addTextWaterMark(File srcImgFile, File targetImgFile, String text, Font font, Color color,
				Float alpha, Integer degree) {
	 
			if (!isFileReadable(srcImgFile) || null == targetImgFile) {
				logger.log(Level.WARNING, "invalid watermark file parameters...");
				return;
			}
	 
			File targetParentFile = targetImgFile.getParentFile();
			if (!targetParentFile.exists()) {
				targetParentFile.mkdirs();
			}
	 
			FileOutputStream outImgStream = null;
	 
			try {
	 
				Image srcImg = ImageIO.read(srcImgFile); // 读取图片
				int srcImgWidth = srcImg.getWidth(null); // 图片宽度
				int srcImgHeight = srcImg.getHeight(null); // 图片高度
	 
				BufferedImage bufferedImg = new BufferedImage(srcImgWidth, srcImgHeight, BufferedImage.TYPE_INT_RGB);
				Graphics2D g = bufferedImg.createGraphics();
				// 开启文本着色抗锯齿
				g.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
				// 控制显示文本的质量
				g.setRenderingHint(RenderingHints.KEY_FRACTIONALMETRICS, RenderingHints.VALUE_FRACTIONALMETRICS_OFF);
				// 控制着色技术
				g.setRenderingHint(RenderingHints.KEY_RENDERING, RenderingHints.VALUE_RENDER_SPEED);
	 
				g.drawImage(srcImg, 0, 0, srcImgWidth, srcImgHeight, null);
	 
				// 设置边白
				// g.setColor(Color.WHITE);
				// g.fillRect(0, srcImgHeight - 30, srcImgWidth, srcImgHeight);
	 
				// 设置水印旋转角度
				if (null != degree) {
					g.rotate(Math.toRadians(degree), (double) bufferedImg.getWidth() / 2,
							(double) bufferedImg.getHeight() / 2);
				}
	 
				// 设置水印文字颜色
				g.setColor(null == color ? COLOR_BLACK : color);
	 
				// 设置水印透明度
				g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP, null == alpha ? ALPHA_NONE : alpha));
	 
				// 设置水印文字字体
				if (null == font) {
					font = FONT_SONG;
				}
				g.setFont(font);
	 
				// 设置水印的坐标
				int x = srcImgWidth - getTextWidth(text, g) - 10;
				int y = srcImgHeight - 10;
	 
				// 绘制水印
				g.drawString(text, x, y);
				g.dispose();
	 
				outImgStream = new FileOutputStream(targetImgFile);
				// ImageIO.write(bufferedImg, "jpg", outImgStream);
				JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(outImgStream);
				encoder.encode(bufferedImg);
				outImgStream.flush();
				outImgStream.close();
	 
				logger.log(Level.INFO, "add text watermark success...");
	 
			} catch (IOException e) {
				logger.log(Level.SEVERE, e.getMessage());
			} finally {
				if (null != outImgStream) {
					try {
						outImgStream.close();
					} catch (IOException e) {
						logger.log(Level.SEVERE, e.getMessage());
					}
				}
			}
	 
		}
	 
		/**
		 * 为图片添加文本水印
		 * 
		 * @param srcImgStream
		 * @param targetImgStream
		 * @param text
		 *            水印文字
		 * @param font
		 *            水印文字字体
		 * @param color
		 *            水印文字颜色
		 * @param alpha
		 *            水印透明度
		 * @param degree
		 *            水印旋转角度
		 */
	 
		public static void addTextWaterMark(InputStream srcImgStream, OutputStream targetImgStream, String text, Font font,
				Color color, Float alpha, Integer degree) {
	 
			if (null == srcImgStream || null == targetImgStream) {
				logger.log(Level.WARNING, "invalid watermark file parameters...");
				return;
			}
	 
			try {
	 
				Image srcImg = ImageIO.read(srcImgStream); // 文件转化为图片
				int srcImgWidth = srcImg.getWidth(null); // 图片宽度
				int srcImgHeight = srcImg.getHeight(null); // 图片高度
	 
				BufferedImage bufferedImg = new BufferedImage(srcImgWidth, srcImgHeight, BufferedImage.TYPE_INT_RGB);
				Graphics2D g = bufferedImg.createGraphics();
				// 开启文本着色抗锯齿
				g.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
				// 控制显示文本的质量
				g.setRenderingHint(RenderingHints.KEY_FRACTIONALMETRICS, RenderingHints.VALUE_FRACTIONALMETRICS_OFF);
				// 控制着色技术
				g.setRenderingHint(RenderingHints.KEY_RENDERING, RenderingHints.VALUE_RENDER_SPEED);
	 
				g.drawImage(srcImg, 0, 0, srcImgWidth, srcImgHeight, null);
	 
				// 设置边白
				// g.setColor(Color.WHITE);
				// g.fillRect(0, srcImgHeight - 30, srcImgWidth, srcImgHeight);
	 
				// 设置水印旋转角度
				if (null != degree) {
					g.rotate(Math.toRadians(degree), (double) bufferedImg.getWidth() / 2,
							(double) bufferedImg.getHeight() / 2);
				}
	 
				// 设置水印文字颜色
				g.setColor(null == color ? COLOR_BLACK : color);
	 
				// 设置水印透明度
				g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP, null == alpha ? ALPHA_NONE : alpha));
	 
				// 设置水印文字字体
				if (null == font) {
					font = FONT_SONG;
				}
				g.setFont(font);
	 
				// 设置水印的坐标
				int x = srcImgWidth - getTextWidth(text, g) - 10;
				int y = srcImgHeight - 10;
	 
				// 绘制水印
				g.drawString(text, x, y);
				g.dispose();
	 
				ImageIO.write(bufferedImg, "png", targetImgStream);
				targetImgStream.flush();
				targetImgStream.close();
	 
				logger.log(Level.INFO, "add text watermark success...");
	 
			} catch (IOException e) {
				logger.log(Level.SEVERE, e.getMessage());
			} finally {
				if (null != targetImgStream) {
					try {
						targetImgStream.close();
					} catch (IOException e) {
						logger.log(Level.SEVERE, e.getMessage());
					}
				}
			}
	 
		}
	 
		/**
		 * 为图片添加图片水印
		 * 
		 * @param srcImgPath
		 *            源图片路径
		 * @param waterImgPath
		 *            水印图片路径
		 * @param targetImgPath
		 *            目标图片路径
		 * @param x
		 *            水印x坐标
		 * @param y
		 *            水印y坐标
		 * @param alpha
		 *            水印透明度
		 * @param degree
		 *            水印旋转角度
		 */
		public static void addImageWaterMark(String srcImgPath, String waterImgPath, String targetImgPath, int x, int y,
				Float alpha, Integer degree) {
	 
			if (isEmptyStr(srcImgPath) || isEmptyStr(waterImgPath) || isEmptyStr(targetImgPath)) {
				logger.log(Level.WARNING, "invalid watermark file path parameters...");
				return;
			}
	 
			File srcImgFile = new File(srcImgPath);
			File waterImgFile = new File(waterImgPath);
			File targetImgFile = new File(targetImgPath);
			addImageWaterMark(srcImgFile, waterImgFile, targetImgFile, x, y, alpha, degree);
		}
	 
		/**
		 * 为图片添加图片水印
		 * 
		 * @param srcImgFile
		 *            源图片文件
		 * @param waterImgFile
		 *            水印图片文件
		 * @param targetImgFile
		 *            目标图片文件
		 * @param x
		 *            水印x坐标
		 * @param y
		 *            水印y坐标
		 * @param alpha
		 *            水印透明度
		 * @param degree
		 *            水印旋转角度
		 */
		public static void addImageWaterMark(File srcImgFile, File waterImgFile, File targetImgFile, int x, int y,
				Float alpha, Integer degree) {
	 
			if (!isFileReadable(srcImgFile) || !isFileReadable(srcImgFile) || null == targetImgFile) {
				logger.log(Level.WARNING, "invalid watermark file parameters...");
				return;
			}
	 
			File targetParentFile = targetImgFile.getParentFile();
			if (!targetParentFile.exists()) {
				targetParentFile.mkdirs();
			}
	 
			FileOutputStream outImgStream = null;
	 
			try {
	 
				Image srcImg = ImageIO.read(srcImgFile); // 读取图片
				int srcImgWidth = srcImg.getWidth(null); // 图片宽度
				int srcImgHeight = srcImg.getHeight(null); // 图片高度
	 
				BufferedImage bufferedImg = new BufferedImage(srcImgWidth, srcImgHeight, BufferedImage.TYPE_INT_RGB);
				Graphics2D g = bufferedImg.createGraphics();
				// 开启图形着色抗锯齿
				g.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
				// 控制颜色着色的方式
				g.setRenderingHint(RenderingHints.KEY_COLOR_RENDERING, RenderingHints.VALUE_COLOR_RENDER_QUALITY);
				// 控制如何处理抖动
				g.setRenderingHint(RenderingHints.KEY_DITHERING, RenderingHints.VALUE_DITHER_ENABLE);
				// 控制内插方式
				g.setRenderingHint(RenderingHints.KEY_INTERPOLATION, RenderingHints.VALUE_INTERPOLATION_BICUBIC);
				// 控制着色技术
				g.setRenderingHint(RenderingHints.KEY_RENDERING, RenderingHints.VALUE_RENDER_SPEED);
	 
				// 绘制目标图片
				g.drawImage(srcImg, 0, 0, srcImgWidth, srcImgHeight, null);
	 
				// 设置水印旋转角度
				if (null != degree) {
					g.rotate(Math.toRadians(degree), (double) bufferedImg.getWidth() / 2,
							(double) bufferedImg.getHeight() / 2);
				}
	 
				// 绘制水印图片
				Image targetImg = ImageIO.read(waterImgFile);
				// SRC、DST、SRC_IN、DST_IN、SRC_OUT、DST_OUT（交集保留其一） / CLEAR（交集都不保留）
				// SRC_OVER、DST_OVER、SRC_ATOP、DST_ATOP（交集一个在上）/ XOR（差集）
				g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, null == alpha ? ALPHA_NONE : alpha));
				g.drawImage(targetImg, x, y, null);
				g.dispose();
	 
				outImgStream = new FileOutputStream(targetImgFile);
				// ImageIO.write(bufferedImg, "jpg", outImgStream);
				JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(outImgStream);
				encoder.encode(bufferedImg);
				outImgStream.flush();
				outImgStream.close();
	 
				logger.log(Level.INFO, "add text watermark success...");
	 
			} catch (IOException e) {
				logger.log(Level.SEVERE, e.getMessage());
			} finally {
				if (null != outImgStream) {
					try {
						outImgStream.close();
					} catch (IOException e) {
						logger.log(Level.SEVERE, e.getMessage());
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
		 * 判断文件是否可读
		 * 
		 * @param file
		 * @return
		 */
		private static boolean isFileReadable(File file) {
			return file != null && file.exists() && file.isFile() && file.canRead();
		}
	 
		/**
		 * 获取文本在图片当前字体下的宽度
		 * 
		 * @param text
		 * @param g
		 * @return
		 */
		public static int getTextWidth(String text, Graphics2D g) {
			return g.getFontMetrics(g.getFont()).charsWidth(text.toCharArray(), 0, text.length());
		}
	 
		public static void main(String[] args) {
			addTextWaterMark("D:/src/Yao1.jpg", "D:/src/target/Yao1.jpg", "作者：pengjunlee",
					new Font("微软雅黑", Font.BOLD, 25), Color.white, null, null);
			addImageWaterMark("D:/src/Yao2.jpg", "D:/src/watermark.png", "D:/src/target/Yao2.jpg", 300, 60, 1f,
					null);
		}
	}
```
程序执行后的结果，左侧为原图，右侧为添加水印后的图片。 

<div align=center>

![给图片添加上水印示意图](http://pengjunlee.3vzhuji.net/static/javacore/6.png "给图片添加上水印示意图")
<div align=left>