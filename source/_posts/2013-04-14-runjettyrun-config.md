---
layout: post
title: runjettyrun配置jetty.xml
date: 2013-04-14 23:37
author: charles
comments: true
categories: [eclipse, java, jetty, 技术]
---
首先推荐一下runjettyrun这个插件,非常好用
从jettylauncher就开始用,后来作者把这个项目改名叫runjettyrun.
这是项目现在的网址:<a href="https://code.google.com/p/run-jetty-run/">https://code.google.com/p/run-jetty-run/</a>

在1.3版本以前是不支持自定义的jetty.xml

我安装的1.3.3,所以已经支持自定义jetty.xml

配置如下:
需要注意的是最好选jetty7版本.要不然需要jetty.xml也必须是jetty6的.

还有其他的配置,例如jvm的一些参数:



