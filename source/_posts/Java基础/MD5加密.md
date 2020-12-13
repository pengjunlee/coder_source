---
title: Java知识点系列之--MD5加密
date: 2020-07-20 13:12:00
updated: 2020-07-20 13:12:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 一起来了解一下Java中的MD5加密。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img12.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img12.jpg
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
# MD5简介

MD5即`Message-Digest Algorithm 5`（信息-摘要算法第五版），是一种在计算机安全领域被广泛使用的散列函数（又译杂凑算法、摘要算法、哈希算法），用于确保所加密的数据的信息完整性和一致性。将数据（如文本、压缩包等）运算为另一固定长度值，是杂凑算法的基础原理，MD5的前身有MD2、MD3和MD4。

MD5的作用是让大容量信息在用数字签名软件签署私人密钥前被"压缩"成一种保密的格式（就是把一个任意长度的字节串变换成一定长的十六进制数字串），它常被用来做一致性检验、数字签名、安全性认证等。除了MD5以外，比较有名的同类算法还有sha-1、RIPEMD以及Haval等。

大家都知道，地球上任何人都有自己独一无二的指纹，这常常成为司法机关鉴别罪犯身份最值得信赖的方法；与之类似，MD5可以为任何文件（不管其大小、格式、数量）产生一个同样独一无二的“数字指纹”，一旦有人对该文件做了任何改动，其MD5值也就是对应的“数字指纹”都会发生变化。

我们常常在某些软件下载站点的某软件信息中看到其MD5值，它的作用就在于我们可以在下载该软件后，对下载回来的文件用专门的软件（如Windows MD5 Check等）做一次MD5校验，以确保我们获得的文件与该站点提供的文件为同一文件。

MD5加密算法可以分为16位加密和32加密的，其实所谓的16位的加密算法只是在32位的加密算法中截取了第8位到第24位字符串，总共16位的字符串，故而叫做是16位的MD5加密算法。 

MD5算法具有以下特点： 

- 压缩性：任意长度的数据，算出的MD5值长度都是固定的。 
- 容易计算：从原数据计算出MD5值很容易。 
- 抗修改性：对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。 
- 强抗碰撞：已知原数据和其MD5值，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。
- 不可解密：根据加密后的md5值，无法逆向得到其原始数据。  

# 在java中使用MD5
对字节数组和字符串的MD5加密实现代码如下：  
```Java
    // 利用MD5对字节数组进行加密，得到128位的字节数组
    public static byte[] encryptByMD5(byte[] input)
    {
        byte[] output = null;
        try
        {
            MessageDigest md5 = MessageDigest.getInstance("MD5");
            output = md5.digest(input);
        }
        catch (NoSuchAlgorithmException e)
        {
            e.printStackTrace();
        }
 
        return output;
    }
 
    // 利用MD5对字符串进行加密，得到128位的字节数组
    public static byte[] encryptByMD5(String input)
    {
        byte[] output = null;
        try
        {
            MessageDigest md5 = MessageDigest.getInstance("MD5");
            output = md5.digest(input.getBytes("utf-8"));
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
 
        return output;
    }
```
在实际使用过程中，我们通常都会将MD5加密后的字节数组转换为字符串进行存储，其代码实现如下。  
```Java
    public static String encryptByMD5(String message)
    {
        String output = null;
        try
        {
            // 1 创建一个提供信息摘要算法的对象，初始化为md5算法对象
            MessageDigest md5 = MessageDigest.getInstance("MD5");
 
            // 2 将消息转换为byte数组
            byte[] input = message.getBytes("utf-8");
 
            // 3 计算后获得size为16（128位）的字节数组
            byte[] buff = md5.digest(input);
 
            // 4 把数组每一字节（一个字节占八位）转换成两个16进制字符并连接起来
            output = toHexString(buff);
 
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        return output;
    }
 
    public static String toHexString(byte[] bytes)
    {
        StringBuffer buffer = new StringBuffer();
        // 把数组每一字节转换成两个16进制的字符，并连接起来
        int digital;
        for (int i = 0; i < bytes.length; i++)
        {
            digital = bytes[i];
 
            if (digital < 0)
            {
                digital += 256;
            }
            if (digital < 16)
            {
                buffer.append("0");
            }
            buffer.append(Integer.toHexString(digital));
        }
        return buffer.toString().toUpperCase();
    }
```
`toHexString(byte[] bytes)`方法的作用是将128位（size=16*8）的字节数组转换为一个长度32的16进制字符串。如果对此方法的实现理解起来有困难，可以参照下面这段代码，两者实现效果相同。  
```Java
    /*
     * Converts a byte to hex digit and writes to the supplied buffer
     */
    private static void byte2hex(byte b, StringBuffer buf)
    {
        char[] hexChars =
        {
                '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'
        };
        int high = ((b & 0xf0) >> 4);
        int low = (b & 0x0f);
        buf.append(hexChars[high]);
        buf.append(hexChars[low]);
    }
 
    /*
     * Converts a byte array to hex string
     */
    private static String toHexString(byte[] block)
    {
        StringBuffer buf = new StringBuffer();
 
        int len = block.length;
 
        for (int i = 0; i < len; i++)
        {
            byte2hex(block[i], buf);
        }
        return buf.toString();
    }
```
出于安全考虑，一般情况下我们并不会直接使用MD5加密得到的结果，而是将MD5加密得到的结果使用BASE64再加密一次，得到相应的字符串。   
```Java
	public static String encryptByMD5(String message)
    {
        String output = null;
        try
        {
            // 1 创建一个提供信息摘要算法的对象，初始化为md5算法对象
            MessageDigest md5 = MessageDigest.getInstance("MD5");
 
            // 2 将消息转换为byte数组
            byte[] input = message.getBytes("utf-8");
 
            // 3 计算后获得size为16（128位）的字节数组
            byte[] buff = md5.digest(input);
 
            BASE64Encoder base64en = new BASE64Encoder();
 
            // 4 将MD5加密获得的字节数组使用BASE64进行第二次加密
            output = base64en.encode(buff);
 
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        return output;
    }
```