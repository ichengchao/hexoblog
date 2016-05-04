---
layout: post
title: linux下的端口映射
date: 2015-05-05 10:13
tags:
  - tech
  
---

之前项目中起了一个邮件服务器,需要监听25端口.  
由于应用要用admin启动,所以需要做一个端口映射.  
网上搜寻一番,发现rinted这个东西完全符合需求,而且配置异常简单.  
[http://www.boutell.com/rinetd/](http://www.boutell.com/rinetd/) 

```sh
wget  http://www.boutell.com/rinetd/http/rinetd.tar.gz
tar fxz rinetd.tar.gz
cd rinetd
make
make install

#配置文件
vi /etc/rinetd.conf 
0.0.0.0 25 0.0.0.0 25000

#启动
rinetd -c /etc/rinetd.conf

#检查,看看25000端口是否已经开启
netstat -nap|grep LISTEN
```