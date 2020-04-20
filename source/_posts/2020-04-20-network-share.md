---
title: 网络分享
date: 2020-04-20 23:51:23
tags:

 - tech
 - tools
 - network
---

最近有网络分享的需求,具体有两个场景

- 手机的网络分享Mac,PC或者电视盒子
- Mac的网络分享给PC,手机或者电视盒子

### 手机分享

##### iphone

Quantumult自带网络分享

<img src="http://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/quantumult_wifi_share.jpeg" style="zoom:50%;" />

##### Android

- Proxy Server
- Easy Proxy

### Mac分享

在介绍之前先插一下[Homebrew](https://brew.sh/)

> **The Missing Package Manager for macOS** 

安装完成后,在/usr/local/就能看到了etc和var就是对应到linux的etc和var,基本使用方法可以网上查一下,如果速度太慢也可以更换一下默认的git remote.然后就可以安装Squid或者Tinyproxy这种代理工具,拿Tinyproxy举个例子

##### Tinyproxy

```bash
brew install tinyproxy
# 安装后修改一下默认的配置文件 /usr/local/etc/tinyproxy/tinyproxy.conf
# 开启LogFile
# 注释掉 # Allow 127.0.0.1
# 因为没有任何规则的时候就是默认Allow
# 启动Tinyproxy
brew serivces start tinyproxy
brew serivces list
# 看一下默认的8888端口是否已经监听
netstat -na|grep LISTEN
# 找一台机器用curl测试一下,1.1.1.1替换成你自己的局域网IP
telnet 1.1.1.1 8888
curl -x 1.1.1.1:8888 www.baidu.com
```