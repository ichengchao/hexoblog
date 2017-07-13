---
title: 命令行发送测试邮件
date: 2017-07-13 21:17:18
tags:
  - tech
  - mail

---

### 动机
之前帮别的团队写了一个邮件转发的[小工具](http://blog.chengchao.name/2013/10/10/java-email-server/),最近需要加点小功能,发现测试起来不是很方便.

### 方案
找寻了一番,发现有个非常方便的方法.直接telnet就能搞定.启动邮件服务,端口是25000,括号里面的是注解.


无认证模式

```shell
telnet localhost 25 
HELO sending-host
MAIL FROM: a@a.com
RCPT TO: b@b.com
DATA
(enter one blank line after DATA)
Subject: test
To: to-user
From: from-user
(enter one blank line after From:)
test text for email
. (enter a single period by itself on the last line)
QUIT
```

简易认证模式,服务端也需要改造

```shell
telnet localhost 25 
HELO sending-host
AUTH PLAIN (Base64-encoded ID/password)
MAIL FROM: a@a.com
RCPT TO: b@b.com
```

很简单,很实用,关键是测试起来非常方便,再也不需要装个客户端测试了.

