---
layout: post
title: iptable的使用
date: 2013-09-05 14:02
author: charles
comments: true

---
//check status 查看当前设置  

`iptables -L -n`

//stop by ip use root(需要用root执行,屏蔽单个ip)  

`iptables -I INPUT -s 192.168.1.1 -j DROP`  

//清除原来的防火墙规则  

`iptables -F`  
`iptables -t nat -F`  
`iptables -t mangle -F`  

//端口转发(有些时候由于启动程序不是用root执行的,但是需要用特殊端口,可以通过端口转发的方式实现)
Linux iptables

This is the magic command that should send all port 25 traffic to 2500. Note that your network interface, eth0, may be different. Also note that you will need to save your iptables state so that it is restored on startup. This varies with Linux flavors, but the Ubuntu instructions are helpful.

`iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 25 -j REDIRECT --to-ports 2500`

