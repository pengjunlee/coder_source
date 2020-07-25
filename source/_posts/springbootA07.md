---
title: SpringBoot框架整合之--使用BootstrapTable进行分页展示
date: 2020-07-24 14:07:00
updated: 2020-07-24 14:07:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何使用BootstrapTable进行分页展示?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
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
# 前端HTML页面
```Html
	<!DOCTYPE html>
	<html>
	<head>
	<meta name="viewport" content="width=device-width" />
	<title>BootstrapTable分页</title>
	 
	<link rel="stylesheet" href="/css/bootstrap.min.css" />
	<link rel="stylesheet" href="/css/bootstrap-table.min.css" />
	<script type="text/javascript" src="/js/jquery-1.11.1.min.js"></script>
	</head>
	<body>
	 
		<div class="bootstrap-table">
			<table id="exampleTable" data-mobile-responsive="true">
			</table>
		</div>
	 
		<script type="text/javascript" src="/js/bootstrap-table.min.js"></script>
		<script type="text/javascript" src="/js/bootstrap.min.js"></script>
		<script type="text/javascript" src="/js/demo.js"></script>
	</body>
	</html>
```

# demo.js 内容
```Javascript
	$(function() {
		load();
	});
	 
	function load() {
		$('#exampleTable').bootstrapTable({
			url : "/demo/load/data", // 请求的后台URL（*）
			method : 'get', // 请求方式：get/post（*）
			showRefresh : false, // 是否显示刷新按钮
			showToggle : false, // 是否显示详细视图和列表视图的切换按钮
			showColumns : false, // 是否显示列操作按钮
			detailView : false, // 是否显示详细视图
			striped : true, // 设置为true会有隔行变色效果
			dataType : "json", // 服务器返回的数据类型
			pagination : true, // 设置为true会在底部显示分页条
			// queryParamsType : "limit",
			// 设置为limit则会发送符合RESTFull格式的参数
			singleSelect : true, // 设置为true将禁止多选
			clickToSelect : true, // 是否启用点击选中行
	 
			// contentType : "application/x-www-form-urlencoded",
			// 发送到服务器的数据编码类型
			pageSize : 10, // 如果设置了分页，每页数据条数
			pageNumber : 1, // 如果设置了分布，首页页码
			search : false, // 是否显示搜索框
	 
			sidePagination : "server", // 设置在哪里进行分页，可选值为"client" 或者 "server"
			queryParams : function(params) {
				return {
					// 说明：传入后台的参数包括offset开始索引，limit步长，sort排序列，order：desc或者,以及所有列的键值对
					limit : params.limit,
					pageSize : 10,
					offset : params.offset,
					search : params.search,
					sort : "age",
					order : "DESC",
					name : "user"
				};
			},
			// 请求服务器数据时，你可以通过重写参数的方式添加一些额外的参数，例如 toolbar 中的参数 如果
			// queryParamsType = 'limit' ,返回参数必须包含
			// limit, offset, search, sort, order 否则, 需要包含:
			// pageSize, pageNumber, searchText, sortName,
			// sortOrder.
			// 返回false将会终止请求
			columns : [ {
				title : '序号',
				field : 'id',
				align : 'left',
				valign : 'center',
				width : '10%',
				formatter : function(value, row, index) {
					return index + 1;
				}
	 
			}, {
				title : '用户ID',
				field : 'id',
				align : 'left',
				valign : 'center',
				width : '20%'
	 
			}, {
				title : '用户姓名',
				field : 'name',
				align : 'left',
				valign : 'center',
				width : '50%',
				cellStyle : function(value, row, index) {
					return {
						css : {
							"word-wrap" : "break-word",
							"word-break" : "normal"
						}
					};
				}
	 
			}, {
				title : '用户年龄',
				field : 'age',
				align : 'left',
				valign : 'center',
				width : '20%'
	 
			}]
		});
	}
```

# 后台Java辅助代码

在此我们分别对 MongoTemplate 和 Mybatis 的分页方法进行举例： 
```Java
	import java.util.List;
	import java.util.Map;
	 
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Controller;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.ResponseBody;
	 
	import com.pengjunlee.common.utils.PageUtils;
	import com.pengjunlee.common.utils.Query;
	import com.pengjunlee.dao.MongoDao;
	import com.pengjunlee.domain.UserEntity;
	import com.pengjunlee.service.MysqlService;
	 
	@Controller
	@RequestMapping("/demo")
	public class DemoController {
		@Autowired
		MongoDao mongoDao;
	 
		@Autowired
		MysqlService mysqlService;
	 
		@GetMapping("/index")
		public String index() {
			return "index";
		}
	 
	    // 两个 /load/data 二选一
		@ResponseBody
		@GetMapping("/load/data") 
		public PageUtils loadFromMongo(@RequestParam Map<String, Object> params) {
			for (Map.Entry<String, Object> entry : params.entrySet()) {
				System.out.println(entry.getKey() + " : " + entry.getValue());
			}
			PageUtils pageUtils = mongoDao.pageUserByCond(params);
			return pageUtils;
		}
	    
	    // 两个 /load/data 二选一
		@GetMapping("/load/data")
		@ResponseBody
		PageUtils loadFromMysql(@RequestParam Map<String, Object> params) {
			Query query = new Query(params);
			List<UserEntity> userList = mysqlService.list(query);
			int total = mysqlService.count(query);
			PageUtils pageUtil = new PageUtils(userList, total);
			return pageUtil;
		}
	 
	}
```
PageUtils 是一个分页查询结果的封装类： 
```Java
	import java.io.Serializable;
	import java.util.List;
	 
	public class PageUtils implements Serializable {
		private static final long serialVersionUID = 1L;
		private int total;
		private List<?> rows;
	 
		public PageUtils(List<?> list, int total) {
			this.rows = list;
			this.total = total;
		}
	 
		public int getTotal() {
			return total;
		}
	 
		public void setTotal(int total) {
			this.total = total;
		}
	 
		public List<?> getRows() {
			return rows;
		}
	 
		public void setRows(List<?> rows) {
			this.rows = rows;
		}
	 
	}
```
Query 是一个分页查询条件的封装类： 

```Java
	import java.util.LinkedHashMap;
	import java.util.Map;
	 
	public class Query extends LinkedHashMap<String, Object> {
		private static final long serialVersionUID = 1L;
		// 偏移页数
		private int offset;
		// 每页条数
		private int limit;
	 
		public Query(Map<String, Object> params) {
			this.putAll(params);
			// 分页参数
			this.offset = Integer.parseInt(params.get("offset").toString());
			this.limit = Integer.parseInt(params.get("limit").toString());
			this.put("offset", offset);
			this.put("page", offset / limit + 1);
			this.put("limit", limit);
		}
	 
		public int getOffset() {
			return offset;
		}
	 
		public void setOffset(int offset) {
			this.put("offset", offset);
		}
	 
		public int getLimit() {
			return limit;
		}
	 
		public void setLimit(int limit) {
			this.limit = limit;
		}
	}
```
定义的用户实体类 UserEntity 代码如下： 
```Java
	import java.io.Serializable;
	 
	public class UserEntity implements Serializable {
	 
		private static final long serialVersionUID = 1L;
	 
		private String id;
		private String name;
	 
		private Integer age;
	 
		public String getId() {
			return id;
		}
	 
		public void setId(String id) {
			this.id = id;
		}
	 
		public String getName() {
			return name;
		}
	 
		public void setName(String name) {
			this.name = name;
		}
	 
		public Integer getAge() {
			return age;
		}
	 
		public void setAge(Integer age) {
			this.age = age;
		}
	 
	}
```

# Mongodb分页实现
```Java
	import java.util.Map;
	 
	import com.pengjunlee.common.utils.PageUtils;
	import com.pengjunlee.domain.UserEntity;
	 
	public interface MongoDao {
	 
		void upsertUser(UserEntity user);
	 
		PageUtils pageUserByCond(Map<String, Object> params);
	}
	import java.util.ArrayList;
	import java.util.List;
	import java.util.Map;
	 
	import org.apache.commons.lang3.StringUtils;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.domain.Sort;
	import org.springframework.data.domain.Sort.Direction;
	import org.springframework.data.mongodb.core.MongoTemplate;
	import org.springframework.data.mongodb.core.query.Criteria;
	import org.springframework.data.mongodb.core.query.Query;
	import org.springframework.data.mongodb.core.query.Update;
	import org.springframework.stereotype.Component;
	 
	import com.pengjunlee.common.utils.PageUtils;
	import com.pengjunlee.dao.MongoDao;
	import com.pengjunlee.domain.UserEntity;
	 
	@Component
	public class MongoDaoImpl implements MongoDao {
	 
		@Autowired
		private MongoTemplate mongoTemplate;
	 
		@Override
		public void upsertUser(UserEntity user) {
			Query query = new Query(Criteria.where("id").is(user.getId()));
			Update update = Update.update("id", user.getId());
			if (StringUtils.isNotBlank(user.getName())) {
				update.set("name", user.getName());
			}
	 
			if (null != user.getAge()) {
				update.set("age", user.getAge());
			}
			mongoTemplate.upsert(query, update, UserEntity.class);
	 
		}
	 
		@Override
		public PageUtils pageUserByCond(Map<String, Object> params) {
			int offset = 0;
			int limit = 10;
			if (null != params) {
	 
				if (StringUtils.isNotBlank(params.get("offset").toString())) {
					offset = Integer.parseInt(params.get("offset").toString());
				}
				if (StringUtils.isNotBlank(params.get("limit").toString())) {
					limit = Integer.parseInt(params.get("limit").toString());
				}
			}
	 
			Query query = null;
			if (StringUtils.isNotBlank(params.get("name").toString())) {
				query = new Query(Criteria.where("name").is(params.get("name").toString()));
			} else {
				query = new Query(new Criteria());
			}
			int count = mongoTemplate.find(query, UserEntity.class).size();
			if (StringUtils.isNotBlank(params.get("sort").toString())
					&& StringUtils.isNotBlank(params.get("order").toString())) {
				query.with(new Sort(Direction.valueOf(params.get("order").toString().toUpperCase()),
						params.get("sort").toString()));
			}
			query.skip(offset);
			query.limit(limit);
	 
			List<UserEntity> users = mongoTemplate.find(query, UserEntity.class);
			PageUtils pageUtils = new PageUtils(new ArrayList<>(), 0);
			pageUtils.setTotal(count);
			pageUtils.setRows(users);
			return pageUtils;
		}
	 
	}
```

# Mybatis分页实现
```Java
	import java.util.List;
	import java.util.Map;
	 
	import com.pengjunlee.domain.UserEntity;
	 
	public interface MysqlService {
	 
		List<UserEntity> list(Map<String, Object> params);
	 
		int count(Map<String, Object> params);
	 
	}
```
<br/>
```Java
	import java.util.List;
	import java.util.Map;
	 
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;
	 
	import com.pengjunlee.dao.MysqlDao;
	import com.pengjunlee.domain.UserEntity;
	import com.pengjunlee.service.MysqlService;
	 
	@Service
	public class MysqlServiceImpl implements MysqlService {
	 
		@Autowired
		private MysqlDao mysqlDao;
	 
		@Override
		public List<UserEntity> list(Map<String, Object> params) {
			return mysqlDao.list(params);
		}
	 
		@Override
		public int count(Map<String, Object> params) {
			return mysqlDao.count(params);
		}
	 
	}
```
<br/>
```Java
	import java.util.List;
	import java.util.Map;
	 
	import org.apache.ibatis.annotations.Mapper;
	 
	import com.pengjunlee.domain.UserEntity;
	 
	@Mapper
	public interface MysqlDao {
	 
		List<UserEntity> list(Map<String, Object> params);
	 
		int count(Map<String, Object> params);
	}
```
<br/>
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	 
	<mapper namespace="com.pengjunlee.dao.MysqlDao">
		<select id="list" resultType="com.pengjunlee.domain.UserEntity">
			select
			`id`,`name`,`age`
			from
			tbl_user
			<where>
				<if test="id != null"> and id = #{id} </if>
				<if test="name != null and name != ''"> and name = #{name} </if>
				<if test="age != null"> and age = #{age} </if>
			</where>
			<choose>
				<when test="sort != null and sort.trim() != ''">
					order by ${sort} ${order}
				</when>
				<otherwise>
					order by id desc
				</otherwise>
			</choose>
			<if test="offset != null and limit != null">
				limit #{offset}, #{limit}
			</if>
		</select>
	 
		<select id="count" resultType="int">
			select count(*) from tbl_user
			<where>
				<if test="id != null"> and id = #{id} </if>
				<if test="name != null and name != ''"> and name = #{name} </if>
				<if test="age != null"> and age = #{age} </if>
			</where>
		</select>
	</mapper>
```

# 分页页面展示

<div align=center>

![分页效果示意图](http://pengjunlee.3vzhuji.net/static/springboot/12.png "分页效果示意图")
<div align=left>