---
layout: post
title: linux下禁用ping和开启ping的方法
date: 2014-08-28 00:42
author: charles
comments: true
tags:
 - linux
 - ping
 - tech
---

###Linux 禁ping和开启ping操作,必须root用户执行###

禁用:  
`# echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all`

开启：  
`# echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all`

