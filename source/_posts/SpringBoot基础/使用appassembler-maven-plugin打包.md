---
title: SpringBoot基础之--使用appassembler-maven-plugin打包
date: 2020-07-25 14:20:00
updated: 2020-07-25 14:20:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何使用appassembler-maven-plugin打包？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img20.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img20.jpg
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
`appassembler-maven-plugin`是一个用来为Java应用打包并生成启动脚本的Maven插件，使用起来非常简单，只需要在项目的pom.xml中加入插件的相关配置即可。它在脚本打包过程中，能够将项目的所有依赖自动复制到指定的打包目录中，并将这些依赖添加到classpath中。

官网地址：<http://www.mojohaus.org/appassembler/appassembler-maven-plugin/>  

# 生成可执行的启动脚本

打包命令：`mvn clean package appassembler:assemble`
```Xml
	<build>
		<plugins>
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>appassembler-maven-plugin</artifactId>
				<version>2.0.0</version>
				<configuration>
					<!-- 生成linux, windows两种平台的执行脚本 -->
					<platforms>
						<platform>unix</platform>
						<platform>windows</platform>
					</platforms>
					<!-- 包存放的根目录 -->
					<assembleDirectory>${project.build.directory}/${project.name}</assembleDirectory>
					<!-- 打包的jar，以及maven依赖的jar存放目录 -->
					<repositoryName>lib</repositoryName>
					<!-- lib目录中jar的存放规则，默认是${groupId}/${artifactId}的目录格式，flat表示直接把jar放到lib目录 -->
					<!-- 可执行脚本的存放目录 -->
					<binFolder>bin</binFolder>
					<!-- 配置文件的存放目录 -->
					<configurationDirectory>conf</configurationDirectory>
					<!-- 拷贝配置文件到上面的目录中 -->
					<copyConfigurationDirectory>true</copyConfigurationDirectory>
					<!-- 从哪里拷贝配置文件 (默认src/main/config) -->
					<configurationSourceDirectory>src/main/resources</configurationSourceDirectory>
					<repositoryLayout>flat</repositoryLayout>
					<encoding>UTF-8</encoding>
					<logsDirectory>logs</logsDirectory>
					<tempDirectory>tmp</tempDirectory>
					<programs>
						<program>
							<!-- 启动类 -->
							<mainClass>com.pengjunlee.MyApplication</mainClass>
							<jvmSettings>
								<extraArguments>
									<extraArgument>-server</extraArgument>
									<extraArgument>-Xmx1G</extraArgument>
									<extraArgument>-Xms1G</extraArgument>
								</extraArguments>
							</jvmSettings>
						</program>
					</programs>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

# 生成后台服务程序

打包命令：`mvn clean package appassembler:generate-daemons`

Usage: `{ console | start | stop | restart | status | dump } `
```Xml
	<build>
		<plugins>
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>appassembler-maven-plugin</artifactId>
				<version>2.0.0</version>
				<configuration>
					<!-- 根目录 -->
					<assembleDirectory>${project.build.directory}/${project.name}</assembleDirectory>
					<!-- 打包的jar，以及maven依赖的jar存放目录 -->
					<repositoryName>lib</repositoryName>
					<!-- 可执行脚本的存放目录 -->
					<binFolder>bin</binFolder>
					<!-- 配置文件的存放目录 -->
					<configurationDirectory>conf</configurationDirectory>
					<!-- 拷贝配置文件到上面的目录中 -->
					<copyConfigurationDirectory>true</copyConfigurationDirectory>
					<!-- 从哪里拷贝配置文件 (默认src/main/config) -->
					<configurationSourceDirectory>src/main/resources</configurationSourceDirectory>
					<!-- lib目录中jar的存放规则，默认是${groupId}/${artifactId}的目录格式，flat表示直接把jar放到lib目录 -->
					<repositoryLayout>flat</repositoryLayout>
					<encoding>UTF-8</encoding>
					<logsDirectory>logs</logsDirectory>
					<tempDirectory>tmp</tempDirectory>
					<daemons>
						<daemon>
							<mainClass>com.pengjunlee.MyApplication</mainClass>
							<platforms>
								<platform>jsw</platform>
							</platforms>
							<generatorConfigurations>
								<generatorConfiguration>
									<generator>jsw</generator>
									<includes>
										<include>linux-x86-32</include>
										<include>linux-x86-64</include>
										<include>windows-x86-32</include>
										<include>windows-x86-64</include>
									</includes>
									<configuration>
										<property>
											<name>configuration.directory.in.classpath.first</name>
											<value>conf</value>
										</property>
										<property>
											<name>wrapper.ping.timeout</name>
											<value>120</value>
										</property>
										<property>
											<name>set.default.REPO_DIR</name>
											<value>lib</value>
										</property>
										<property>
											<name>wrapper.logfile</name>
											<value>logs/wrapper.log</value>
										</property>
									</configuration>
								</generatorConfiguration>
							</generatorConfigurations>
							<jvmSettings>
								<!-- jvm参数 -->
								<systemProperties>
									<systemProperty>com.sun.management.jmxremote</systemProperty>
									<systemProperty>com.sun.management.jmxremote.port=1984</systemProperty>
									<systemProperty>com.sun.management.jmxremote.authenticate=false</systemProperty>
									<systemProperty>com.sun.management.jmxremote.ssl=false</systemProperty>
								</systemProperties>
							</jvmSettings>
						</daemon>
					</daemons>
				</configuration>
			</plugin>
		</plugins>
	</build>
```