---
layout: post
title: linux开机运行命令
date: 2014-03-21 11:13
author: charles
comments: true
tags:
 - linux
 - tech
---

`sudo vi /etc/rc.local`


我的树莓派的rc.local看起来是这样的:  


```sh
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

#在这里加入自己的脚本：
printf "hello,my first run"

exit 0
```



