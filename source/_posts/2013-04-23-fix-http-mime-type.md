---
layout: post
title: 解决:mime类型不匹配 javascript不能加载
date: 2013-04-23 22:36
author: charles
comments: true
tags: 
  - apache
  - http
  - tech
  
---
前段时间有些用户反映A应用在IE9+中的浏览器打不开.  
F12看了一下控制台输出,大概意思是:  **脚本因 mime 类型不匹配而被阻止  
由于本来用IE的人就很少,就建议他换个firefox或者chrome.  
这几天又有人反映在chrome 27.* 的版本中也出现这个问题.  
发现问题有点严重了.搜索一番,官方资料如下:  
出现这个问题的解释:  
<http://msdn.microsoft.com/zh-cn/library/ie/hh180764(v=vs.85).aspx>  
其中SEC7112 说的就是这个问题.  

还有一个详细的说明:(microsoft的文档真牛逼)  
<http://msdn.microsoft.com/en-us/library/ie/gg622941(v=vs.85).aspx>  

看了后知道是因为MIME type设置有问题,不过应用中用到js文件是用velocity渲染出来,而且访问后缀是.htm,所以通过配置web.xml的方式行不通  
仔细看了看,这个问题的前提是设置了`X-Content-Type-Options: nosniff`
又查了一下,这个参数是为了防止html注入的.由于是内部应用所以决定先在response header中去除掉.  

知道解决方案就很简单了.
配置httpd.conf

```xml
<IfModule mod_headers.c>
#       Header set X-Content-Type-Options "nosniff"
		Header set X-XSS-protection "mode=block"
		RequestHeader unset Range
</IfModule>
```
搞定!
