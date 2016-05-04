---
layout: post
title: msmtp 发送邮件
date: 2014-02-10 17:11
author: charles
comments: true
tags:
 - linux
 - email
 - tech
---
<strong>安装:</strong>
[code lang="shell"]
sudo apt-get install msmtp
[/code]

<strong>在home目录新建.msmtprc文件</strong>
以163邮箱为例:
[code lang="shell"]
account default
host smtp.163.com
from 账号@163.com
auth login
user 账号
password 密码
logfile ~/.msmtp.log
[/code]

<strong>发封邮件试试:</strong>
[code lang="shell"]
echo "hello" | msmtp -a default QQ号码@qq.com
[/code]

登录qq邮箱看看,应该收到了一封无主题的邮件.


<strong>Gmail的配置:</strong>
[code lang="shell"]
account default
tls on
tls_starttls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
host smtp.gmail.com
port 587
from 账号@gmail.com
auth login
user 账号@gmail.com
password 密码
logfile ~/.msmtp.log
[/code]

QQ邮件比较恶心,目前还不知道怎么配置,网上能搜索到的方法都是不行的.


<strong>和mutt结合</strong>
[code lang="shell"]
//安装 mutt
sudo apt-get install mutt

//在home目录增加.muttrc,内容如下(具体路径用which mutt查看):
set sendmail="/usr/bin/msmtp"

//发送测试邮件

echo "hello,mutt" |mutt -s "my_title" QQ号码@qq.com
[/code]


