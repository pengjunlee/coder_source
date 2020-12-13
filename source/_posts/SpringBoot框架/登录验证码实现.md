---
title: SpringBoot框架整合之--登录验证码实现
date: 2020-07-24 14:03:00
updated: 2020-07-24 14:03:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot项目中如何实现登录验证码?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
> 转载自：https://blog.csdn.net/lp840312696/article/details/90635039

今天记录一下验证码的实现，希望能够帮助到大家！

首先我们看一下实现的效果：

<div align=center>

![验证码示意图](http://pengjunlee.3vzhuji.net/static/springboot/00.png "验证码示意图")
<div align=left>

此验证码的实现没有用到太多的插件，话不多说直接上代码，大家拿过去就可以用。

# 1.验证码类
```Java
	package com.youyou.login.util.validatecode;
	 
	import lombok.Data;
	 
	/**
	 * 验证码类
	 */
	@Data
	public class VerifyCode {
	 
	    private String code;
	 
	    private byte[] imgBytes;
	 
	    private long expireTime;
	 
	}
```

# 2.验证码生成接口
```Java
	package com.youyou.login.util.validatecode;
	 
	import java.io.IOException;
	import java.io.OutputStream;
	 
	/**
	 * 验证码生成接口
	 */
	public interface IVerifyCodeGen {
	 
	    /**
	     * 生成验证码并返回code，将图片写的os中
	     *
	     * @param width
	     * @param height
	     * @param os
	     * @return
	     * @throws IOException
	     */
	    String generate(int width, int height, OutputStream os) throws IOException;
	 
	    /**
	     * 生成验证码对象
	     *
	     * @param width
	     * @param height
	     * @return
	     * @throws IOException
	     */
	    VerifyCode generate(int width, int height) throws IOException;
	}
```

# 3.验证码生成实现类
```Java
	package com.youyou.login.util.validatecode;
	 
	import com.youyou.util.RandomUtils;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	 
	import javax.imageio.ImageIO;
	import java.awt.*;
	import java.awt.image.BufferedImage;
	import java.io.ByteArrayOutputStream;
	import java.io.IOException;
	import java.io.OutputStream;
	import java.util.Random;
	 
	/**
	 * 验证码实现类
	 */
	public class SimpleCharVerifyCodeGenImpl implements IVerifyCodeGen {
	 
	    private static final Logger logger = LoggerFactory.getLogger(SimpleCharVerifyCodeGenImpl.class);
	 
	    private static final String[] FONT_TYPES = { "\u5b8b\u4f53", "\u65b0\u5b8b\u4f53", "\u9ed1\u4f53", "\u6977\u4f53", "\u96b6\u4e66" };
	 
	    private static final int VALICATE_CODE_LENGTH = 4;
	 
	    /**
	     * 设置背景颜色及大小，干扰线
	     *
	     * @param graphics
	     * @param width
	     * @param height
	     */
	    private static void fillBackground(Graphics graphics, int width, int height) {
	        // 填充背景
	        graphics.setColor(Color.WHITE);
	        //设置矩形坐标x y 为0
	        graphics.fillRect(0, 0, width, height);
	 
	        // 加入干扰线条
	        for (int i = 0; i < 8; i++) {
	            //设置随机颜色算法参数
	            graphics.setColor(RandomUtils.randomColor(40, 150));
	            Random random = new Random();
	            int x = random.nextInt(width);
	            int y = random.nextInt(height);
	            int x1 = random.nextInt(width);
	            int y1 = random.nextInt(height);
	            graphics.drawLine(x, y, x1, y1);
	        }
	    }
	 
	    /**
	     * 生成随机字符
	     *
	     * @param width
	     * @param height
	     * @param os
	     * @return
	     * @throws IOException
	     */
	    @Override
	    public String generate(int width, int height, OutputStream os) throws IOException {
	        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
	        Graphics graphics = image.getGraphics();
	        fillBackground(graphics, width, height);
	        String randomStr = RandomUtils.randomString(VALICATE_CODE_LENGTH);
	        createCharacter(graphics, randomStr);
	        graphics.dispose();
	        //设置JPEG格式
	        ImageIO.write(image, "JPEG", os);
	        return randomStr;
	    }
	 
	    /**
	     * 验证码生成
	     *
	     * @param width
	     * @param height
	     * @return
	     */
	    @Override
	    public VerifyCode generate(int width, int height) {
	        VerifyCode verifyCode = null;
	        try (
	                //将流的初始化放到这里就不需要手动关闭流
	                ByteArrayOutputStream baos = new ByteArrayOutputStream();
	        ) {
	            String code = generate(width, height, baos);
	            verifyCode = new VerifyCode();
	            verifyCode.setCode(code);
	            verifyCode.setImgBytes(baos.toByteArray());
	        } catch (IOException e) {
	            logger.error(e.getMessage(), e);
	            verifyCode = null;
	        }
	        return verifyCode;
	    }
	 
	    /**
	     * 设置字符颜色大小
	     *
	     * @param g
	     * @param randomStr
	     */
	    private void createCharacter(Graphics g, String randomStr) {
	        char[] charArray = randomStr.toCharArray();
	        for (int i = 0; i < charArray.length; i++) {
	            //设置RGB颜色算法参数
	            g.setColor(new Color(50 + RandomUtils.nextInt(100),
	                    50 + RandomUtils.nextInt(100), 50 + RandomUtils.nextInt(100)));
	            //设置字体大小，类型
	            g.setFont(new Font(FONT_TYPES[RandomUtils.nextInt(FONT_TYPES.length)], Font.BOLD, 26));
	            //设置x y 坐标
	            g.drawString(String.valueOf(charArray[i]), 15 * i + 5, 19 + RandomUtils.nextInt(8));
	        }
	    }
	}
```

# 4.工具类
```Java
	package com.youyou.util;
	 
	import java.awt.*;
	import java.util.Random;
	 
	public class RandomUtils extends org.apache.commons.lang3.RandomUtils {
	 
	    private static final char[] CODE_SEQ = { 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'J',
	            'K', 'L', 'M', 'N', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W',
	            'X', 'Y', 'Z', '2', '3', '4', '5', '6', '7', '8', '9' };
	 
	    private static final char[] NUMBER_ARRAY = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' };
	 
	    private static Random random = new Random();
	 
	    public static String randomString(int length) {
	        StringBuilder sb = new StringBuilder();
	        for (int i = 0; i < length; i++) {
	            sb.append(String.valueOf(CODE_SEQ[random.nextInt(CODE_SEQ.length)]));
	        }
	        return sb.toString();
	    }
	 
	    public static String randomNumberString(int length) {
	        StringBuilder sb = new StringBuilder();
	        for (int i = 0; i < length; i++) {
	            sb.append(String.valueOf(NUMBER_ARRAY[random.nextInt(NUMBER_ARRAY.length)]));
	        }
	        return sb.toString();
	    }
	 
	    public static Color randomColor(int fc, int bc) {
	        int f = fc;
	        int b = bc;
	        Random random = new Random();
	        if (f > 255) {
	            f = 255;
	        }
	        if (b > 255) {
	            b = 255;
	        }
	        return new Color(f + random.nextInt(b - f), f + random.nextInt(b - f), f + random.nextInt(b - f));
	    }
	 
	    public static int nextInt(int bound) {
	        return random.nextInt(bound);
	    }
	}
```
经过以上代码，我们的验证码生成功能基本上已经实现了，现在还需要一个controller来调用它。
```Java
    @ApiOperation(value = "验证码")
    @GetMapping("/verifyCode")
    public void verifyCode(HttpServletRequest request, HttpServletResponse response) {
        IVerifyCodeGen iVerifyCodeGen = new SimpleCharVerifyCodeGenImpl();
        try {
            //设置长宽
            VerifyCode verifyCode = iVerifyCodeGen.generate(80, 28);
            String code = verifyCode.getCode();
            LOGGER.info(code);
            //将VerifyCode绑定session
            request.getSession().setAttribute("VerifyCode", code);
            //设置响应头
            response.setHeader("Pragma", "no-cache");
            //设置响应头
            response.setHeader("Cache-Control", "no-cache");
            //在代理服务器端防止缓冲
            response.setDateHeader("Expires", 0);
            //设置响应内容类型
            response.setContentType("image/jpeg");
            response.getOutputStream().write(verifyCode.getImgBytes());
            response.getOutputStream().flush();
        } catch (IOException e) {
            LOGGER.info("", e);
        }
    }
```
搞定！后台编写到此结束了。那么又会有博友说了：“说好的实现效果呢？”

好吧，那么我们继续前端的代码编写。

前端代码:
```Html
	<html>
		<body>
			<div>
	    		<input id="code" placeholder="验证码" type="text" class="" style="width:170px">
			    <!-- 验证码 显示 -->
			    <img οnclick="javascript:getvCode()" id="verifyimg" style="margin-left: 20px;"/>
			</div>
			 
			<script type="text/javascript">
			    getvCode();
			 
			    /**
			     * 获取验证码
			     * 将验证码写到login.html页面的id = verifyimg 的地方
			     */
			    function getvCode() {
			        document.getElementById("verifyimg").src = timestamp("http://127.0.0.1:81/verifyCode");
			    }
			    //为url添加时间戳
			     function timestamp(url) {
			        var getTimestamp = new Date().getTime();
			        if (url.indexOf("?") > -1) {
			            url = url + "&timestamp=" + getTimestamp
			        } else {
			            url = url + "?timestamp=" + getTimestamp
			        }
			        return url;
			    };
			</script>
		</body>
	</html>
```
可以实现点击图片更换验证码。

实现效果：

<div align=center>

![验证码示意图](http://pengjunlee.3vzhuji.net/static/springboot/01.png "验证码示意图")
<div align=left>

当然文章开头的截图是我系统中的截图，需要大家自己去根据自己的情况去开发前端了。