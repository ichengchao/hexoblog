---
layout: post
title: Extjs(1):hello world
date: 2014-04-03 15:32
tags:
  - extjs
  - html
---


简介
---

- [百度百科](http://baike.baidu.com/view/1350145.htm)
- [维基百科](http://zh.wikipedia.org/zh/Extjs)


我个人的一些看法:  

- 初次学习成本有点高,但是理解一个组件后基本能融会贯通.
- 适合做后台管理系统,可以再没有UED支持的情况下也能做到功能丰富,跨浏览器而且不是很难看.
- 文档异常丰富,当然人家是靠这个卖钱的.
- 两种license.付费版和LGPL,想把产品拿来卖钱的需要关注一下,内部使用不收费.[许可选择](http://choosealicense.gitcafe.com/)


开始
---

在[官方网站](http://www.sencha.com/products/extjs/download/)下载  
安装一个nginx或者apache之类的web容器.把下载后的zip减压到html目录.这样就能在本地查看文档和示例了.  
在你的web root目录增加extjs的资源文件.如下图,只加入必要的资源文件  

新建一个index.html和index.js  
index.html

```html
<html>
<head>
	<link rel="stylesheet" type="text/css"	href="extjslibv4/resources/css/ext-all.css" />
	<script type="text/javascript" src="extjslibv4/ext-all.js"></script>
	<script type="text/javascript" src="index.js"></script>
</head>
<body>
</body>
</html>
```

index.js

```js
Ext.onReady(function() {
			Ext.MessageBox.alert('Hello', 'extjs.');
		})
```

运行浏览器,输入地址 http://localhost/index.html  
至此,extjs的hello world就结束了.是不是还比较简单啊.
