---
title: SpringBoot基础之--Controller处理Excel下载请求
date: 2020-07-25 14:08:00
updated: 2020-07-25 14:08:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot的application.properties中都包含哪些配置项?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
> 转载自：<https://www.cnblogs.com/gudongcheng/p/8268909.html>

在许多企业办公系统中，经常会有用户要求，需要对数据进行统计并且可以直接下载Excel文件，这样子的话，既然客户提出了要求，我们就应该去满足吖，毕竟客户是上帝嘛，那么我们如何去实现呢？且看我为你一一道来。

# POI简介
`Jakarta POI` 是一套用于访问微软格式文档的Java API。Jakarta POI有很多组件组成，其中有用于操作Excel格式文件的HSSF和用于操作Word的HWPF，在各种组件中目前只有用于操作Excel的HSSF相对成熟。

官方主页：<http://poi.apache.org/index.html>

API文档：<http://poi.apache.org/apidocs/index.html>

现在用的比较多的都是用POI技术来导出或者导入Excel，所以我们就用POI吧，用POI导出Excel我们首先要下载所需的jar包然后导入到我们的项目中，用maven的同学只需找到相关依赖加入到pom.xml里面即可。

# 使用POI

## 下载jar包
官方下载：<http://poi.apache.org/download.html> 这里可以下载到它的最新版本和文档，目前最新版本是3.7，这里使用比较稳定的3.6版。

百度网盘下载：<https://pan.baidu.com/s/1mjhoaWK>  密码：`pkur`

## 将jar包加入到项目中
将下载好的jar包加入到`WEB-INFO`目录下的`lib`文件夹中，Eclipse用户选中jar包然后右击选择Build Path选项， Idea用户选中jar包然后右击选择Add as Library选项即可。

如果是用maven的可自行到maven中央仓库搜索poi然后选择对应的版本即可，也可以直接将下面代码复制到pom.xml。
```Xml
	<!-- https://mvnrepository.com/artifact/org.apache.poi/poi -->
	<dependency>
	    <groupId>org.apache.poi</groupId>
	    <artifactId>poi</artifactId>
	    <version>3.6</version>
	</dependency>
```
> **小提示**：用maven引入依赖jar包的可能会遇到包引用不到的bug，但是maven依赖确实已经引入了，而且没有任何报错，但是只要一引用 `org.apache.poi.hssf.usermodel` 下面的类就会报错，报错内容为：<font color=red>Caused by: java.lang.NoClassDefFoundError: org/apache/poi/hssf/usermodel/HSSFWorkbook</font>。

## Jakarta POI HSSF API组件
HSSF（用于操作Excel的组件）提供给用户使用的对象在 `rg.apache.poi.hssf.usermodel` 包中,主要部分包括Excel对象，样式和格式，有以下几种常用的对象：

常用组件：
```
	- HSSFWorkbook     excel的文档对象
	- HSSFSheet        excel的表单
	- HSSFRow          excel的行
	- HSSFCell         excel的格子单元
	- HSSFFont         excel字体
```
样式：
```
	- HSSFCellStyle         cell样式
```
## 基本操作步骤
首先，我们应该要知道的是，一个Excel文件对应一个workbook，一个workbook中有多个sheet组成，一个sheet是由多个行(row)和列(cell)组成。那么我们用poi要导出一个Excel表格的正确顺序应该是：

1. 用HSSFWorkbook打开或者创建“Excel文件对象”
2. 用HSSFWorkbook对象返回或者创建Sheet对象
3. 用Sheet对象返回行对象，用行对象得到Cell对象
4. 对Cell对象读写
5. 将生成的HSSFWorkbook放入HttpServletResponse中响应到前端页面

## 导出Excel应用实例
**工具类代码**：
```Java
	package com.yq.util;
	 
	import org.apache.poi.hssf.usermodel.HSSFCell;
	import org.apache.poi.hssf.usermodel.HSSFCellStyle;
	import org.apache.poi.hssf.usermodel.HSSFRow;
	import org.apache.poi.hssf.usermodel.HSSFSheet;
	import org.apache.poi.hssf.usermodel.HSSFWorkbook;
	 
	public class ExcelUtil {
	 
	    /**
	     * 导出Excel
	     * @param sheetName sheet名称
	     * @param title 标题
	     * @param values 内容
	     * @param wb HSSFWorkbook对象
	     * @return
	     */
	    public static HSSFWorkbook getHSSFWorkbook(String sheetName,String []title,String [][]values, HSSFWorkbook wb){
	 
	        // 第一步，创建一个HSSFWorkbook，对应一个Excel文件
	        if(wb == null){
	            wb = new HSSFWorkbook();
	        }
	 
	        // 第二步，在workbook中添加一个sheet,对应Excel文件中的sheet
	        HSSFSheet sheet = wb.createSheet(sheetName);
	 
	        // 第三步，在sheet中添加表头第0行,注意老版本poi对Excel的行数列数有限制
	        HSSFRow row = sheet.createRow(0);
	 
	        // 第四步，创建单元格，并设置值表头 设置表头居中
	        HSSFCellStyle style = wb.createCellStyle();
	        style.setAlignment(HSSFCellStyle.ALIGN_CENTER); // 创建一个居中格式
	 
	        //声明列对象
	        HSSFCell cell = null;
	 
	        //创建标题
	        for(int i=0;i<title.length;i++){
	            cell = row.createCell(i);
	            cell.setCellValue(title[i]);
	            cell.setCellStyle(style);
	        }
	 
	        //创建内容
	        for(int i=0;i<values.length;i++){
	            row = sheet.createRow(i + 1);
	            for(int j=0;j<values[i].length;j++){
	                //将内容按顺序赋给对应的列对象
	                row.createCell(j).setCellValue(values[i][j]);
	            }
	        }
	        return wb;
	    }
	}
```
**控制器代码**：
```Java
	@Controller
	@RequestMapping(value = "/report")
	public class ReportFormController extends BaseController {
	 
	    @Resource(name = "reportService")
	    private ReportManager reportService;
	 
	    /**
	     * 导出报表
	     * @return
	     */
	    @RequestMapping(value = "/export")
	    @ResponseBody
	    public void export(HttpServletRequest request,HttpServletResponse response) throws Exception {
	           //获取数据
	           List<PageData> list = reportService.bookList(page);
	 
	           //excel标题
	        　　String[] title = {"名称","性别","年龄","学校","班级"};
	 
	         　//excel文件名
	        　 String fileName = "学生信息表"+System.currentTimeMillis()+".xls";
	 
	  　　　　　//sheet名
	        　 String sheetName = "学生信息表";
	 
	　　　　　　for (int i = 0; i < list.size(); i++) {
	            content[i] = new String[title.length];
	            PageData obj = list.get(i);
	            content[i][0] = obj.get("stuName").tostring();
	            content[i][1] = obj.get("stuSex").tostring();
	            content[i][2] = obj.get("stuAge").tostring();
	            content[i][3] = obj.get("stuSchoolName").tostring();
	            content[i][4] = obj.get("stuClassName").tostring();
	　　　　　　}
	 
	　　　　　　//创建HSSFWorkbook 
	　　　　　　HSSFWorkbook wb = ExcelUtil.getHSSFWorkbook(sheetName, title, content, null);
	 
	　　　　　　//响应到客户端
	　　　　　　try {
	　　　　　　　　this.setResponseHeader(response, fileName);
	       　　　　OutputStream os = response.getOutputStream();
	       　　　　wb.write(os);
	       　　　　os.flush();
	       　　　　os.close();
	 　　　　　　} catch (Exception e) {
	       　　　　e.printStackTrace();
	 　　　　　　}
	　　}
	 
	    //发送响应流方法
	    public void setResponseHeader(HttpServletResponse response, String fileName) {
	        try {
	            try {
	                fileName = new String(fileName.getBytes(),"ISO8859-1");
	            } catch (UnsupportedEncodingException e) {
	                // TODO Auto-generated catch block
	                e.printStackTrace();
	            }
	            response.setContentType("application/octet-stream;charset=ISO8859-1");
	            response.setHeader("Content-Disposition", "attachment;filename="+ fileName);
	            response.addHeader("Pargam", "no-cache");
	            response.addHeader("Cache-Control", "no-cache");
	        } catch (Exception ex) {
	            ex.printStackTrace();
	        }
	    }
	}
```
**前端页面代码**：
```Html
	<button id="js-export" type="button" class="btn btn-primary">导出Excel</button>
	$('#js-export').click(function(){
	     window.location.href="/report/exportBooksTable.do;
	});
```
好啦，到此大功告成，点击按钮文件就会自动下载了。