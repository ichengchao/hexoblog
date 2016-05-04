---
layout: post
title: Java 获取资源路径
date: 2014-01-24 16:17
author: charles
comments: true
tags:
 - java
 - tech
---

在java中经常需要获取资源文件,常用的方式就是:

```java

//这是获取在Test.java所在目录的的a.txt
URL url = Test.class.getResource("a.txt");

//"/"表示classes的根目录在的a.txt
URL url = Test.class.getResource("/a.txt");

//获取jar中的文件,假设依赖了fastjson,就是查找jar包中的资源
//这个比较特殊,怎么理解这个呢,其实就是就是把所有依赖的jar(包括自己的源码)都解压缩到target/classes目录下
//按照各自的命名空间存放,万一有两个一样的文件都在根目录下会有什么效果呢?
//后面加载的会覆盖前面加载的.
//举个例子,在ibaits的jar的根目录下有一个license.txt,自己项目的resource目录下也有个license.txt,
//那用Test.class.getClassLoader().getResource("license.txt");
//加载的就是自己的那个.如果把自己的删除了,加载的就是ibatis中的license.txt
URL url = Test.class.getResource("/com/alibaba/fastjson/JSON.class");

//这个不需要"/"开头了.因为classloader的getResource就是从根目录开始的
URL url = Test.class.getClassLoader().getResource("com/alibaba/fastjson/JSON.class");

//在web应用中获取webapp目录中的文件
//获取webapp根目录
request.getSession().getServletContext().getRealPath("/");

//获取lib的目录的路径.
request.getSession().getServletContext().getRealPath("/WEB-INF/lib");

```

当然还有getResourceAsStream等方法,不过道理都是一样的.
例子的注释写得恶心了点,不过还是能看的.
理解了上面的例子后,基本在java项目中要找一个文件的话就不会手足无措了.


