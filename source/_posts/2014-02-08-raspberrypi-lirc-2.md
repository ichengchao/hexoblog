---
layout: post
title: 树莓派的红外发射
date: 2014-02-08 17:49
tags:
  - raspberrypi
  - lirc

---
接着上篇,这篇说说红外发射

```sh
//上篇中的绑定命令是这样的,默认发射用的是GPIO17:
sudo modprobe lirc_rpi

//(默认是这样的,相当于sudo modprobe lirc_rpi)
sudo modprobe lirc_rpi gpio_in_pin=18 gpio_out_pin=17  
可以互换:
sudo modprobe lirc_rpi gpio_in_pin=17 gpio_out_pin=18

//查看绑定情况:
mount -t debugfs debugfs /sys/kernel/debug
cat /sys/kernel/debug/gpio 
```


完成后测试一下:

```sh
//看看有什么命令
irsend LIST tv ""

//挑一个试试
irsend SEND_ONCE tv KEY_1
```

总结:
试了几样家里的电器,发现华数的机顶盒,电风扇是可以正常工作的.乐视电视不行
一边发送,一边用irw是不行的.(不知道是什么原因)
我买了好几个红外模块.发现这家的工作正常:<a href="http://ywrobot.taobao.com/" title="淘宝地址" target="_blank">淘宝地址</a>
不是做广告,只是防止你们买到不能用的模块


资料地址:
<a href="http://aron.ws/projects/lirc_rpi/" title="http://aron.ws/projects/lirc_rpi/" target="_blank">http://aron.ws/projects/lirc_rpi/</a>
