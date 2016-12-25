---
layout: post
title: maven问题记录
date: 2014-03-07 13:24
author: charles
comments: true
tags:
 - java
 - maven
 - tech
 
---

1. 出现:  *** 是 Sun 的专用 API，可能会在未来版本中删除 的问题
解决:指定maven-compiler-plugin的版本2.3.2或者更新的版本

2. 命令行指定settings.xml
举例`mvn install -Dorg.apache.maven.user-settings="D:\my_settings.xml"`  
该参数在maven3中不能使用,目前的解决方式是下载一个2.2.1的版本

3. maven的国内镜像,oschina的已经关闭,用阿里云的代替

```xml
<mirror>
	<id>aliyun</id>
	<mirrorOf>central</mirrorOf>
	<name>aliyun mirror</name>
	<url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

4. 使用archetype:generate卡住的问题是因为请求中央库的时候卡住了

```
#调试,查看卡住的原因
mvn archetype:generate -X
#解决
mvn archetype:generate -DarchetypeCatalog=internal

```