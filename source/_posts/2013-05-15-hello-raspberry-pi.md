---
layout: post
title: hello,raspberry pi
date: 2013-05-15 23:03
tags:
  - raspberrypi
  - tech
---

前几天买了块arduino玩,不是很过瘾.  
毕竟已经是互联网时代了,不能上网的设备就不是好设备.  
于是搞了块raspberry pi.  
购买地址:[ICkey,带wifi网卡的套餐](http://www.ickey.cn/)  
按照官方的[quick-start](http://www.raspberrypi.org/quick-start-guide)装好os.  
电源线需要用5V--1A的,发现iphone的充电器刚好  

启动后直接用pi登录,默认密码是:raspberry  
系统中的root默认没有密码,自己sudo passwd root设置一个新的就行.  
几个比较有用的步骤记录:(root)  

- agt-get update //更新源列表
- agt-get install vim
- 更改.bashrc 启动ll
- 配置wifi的静态ip  

下面是修改后的/etc/network/interface  
配置生效:service networking restart  

```sh
auto lo
auto wlan0 #优先使用wifi

iface lo inet loopback
iface eth0 inet static
        address 192.168.1.99
        netmask 255.255.255.0
        gateway 192.168.1.1

iface wlan0 inet static
//如果是dhcp,则把static换成dhcp,而且把adress,netmask,gateway注释
        address 192.168.1.98
        netmask 255.255.255.0
        gateway 192.168.1.1
        wpa-ssid "wifi名称"
        wpa-psk "wifi密码"
allow-hotplug wlan0 #表示可以热插拔
```