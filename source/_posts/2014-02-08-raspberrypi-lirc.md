---
layout: post
title: 树莓派的红外接收
date: 2014-02-08 15:19
tags:
 - lirc
 - raspberrypi
 
---

这篇将介绍怎么来利用红外遥控器操作树莓派
需要准备一个红外接收器,淘宝上搜索:HS0038B
<strong>接收器连接:</strong>
data:GPIO1 (18)
gnd:gnd
vcc:3.3v
<strong>顺便说下发射器的连接:</strong>
data:GPIO0 (17)
gnd:gnd
vcc:5v
发送命令举例(与本文无关):

```sh
irsend SEND_ONCE tv KEY_1
irsend SEND_ONCE tv KEY_POWER


<strong>安装并调试:</strong>


#安装lirc
sudo apt-get install lirc

#完成后,开机会自动启动该服务,如果想关闭开机启动
禁用: sudo update-rc.d lirc remove
启用: sudo update-rc.d lirc defaults

#绑定模块并校验
sudo modprobe lirc_rpi
dmesg | grep -i lirc

#测试,随便按遥控器,如果出现一堆的pluse和space,表示工作正常
mode2 -d /dev/lirc0


配置一下/etc/hardware.conf

#其余的不用配置
LIRCD_ARGS="--uinput"
DRIVER="default"
DEVICE="/dev/lirc0"


<strong>录制按键:</strong>


#用 irrecord --list-namespace 查看可以使用的key列表,录制前需要关闭sudo /etc/init.d/lirc stop
#开始录制,开始后需要随便按键通过2*80的调试,随后输入key(必须在irrecord --list-namespace 列表中)

irrecord -n -d /dev/lirc0 ~/lircd.conf

#修改 conf中的name为 tv (也可以是别的名称)

sudo mv ~/lircd.conf /etc/lirc/lircd.conf


<strong>我的lircd.conf:</strong>


# Please make this file available to others
# by sending it to <lirc@bartelmus.de>
#
# this config file was automatically generated
# using lirc-0.9.0-pre1(default) on Fri Feb  7 14:20:44 2014
#
# contributed by
#
# brand:                       /home/pi/lircd.conf
# model no. of remote control:
# devices being controlled by this remote:
#

begin remote

  name  tv
  bits           16
  flags SPACE_ENC|CONST_LENGTH
  eps            30
  aeps          100

  header       9011  4499
  one           566  1673
  zero          566   556
  ptrail        564
  pre_data_bits   16
  pre_data       0x32A6
  gap          107974
  toggle_bit_mask 0x0

      begin codes
          KEY_1                    0x807F
          KEY_2                    0x40BF
          KEY_3                    0xC03F
      end codes

end remote



<strong>启动并调试:</strong>

sudo /etc/init.d/lirc start

#使用irw命令测试
#我的测试结果:
0000000032a6807f 00 KEY_1 tv
0000000032a640bf 00 KEY_2 tv
0000000032a6c03f 00 KEY_3 tv

到这里,红外接收已经能正常工作了.
那怎么样才能利用红外执行命令呢?
lircrc该出场了.


<strong>执行系统命令:</strong>
在home目录添加一个.lircrc (这个是个人的)
或者在/etc/lirc/目录下增加一个lircrc(文件前面没有点,这个是全局的)
lircrc内容:

#config的也可以是多个用逗号分隔
begin
    prog = irexec
    button = KEY_1
    config = echo "hello lirc one!"
end
begin
    prog = irexec
    button = KEY_2
    config = echo "hello lirc two!"
end


配置完成后重启lirc:sudo /etc/init.d/lirc restart,并且启动irexec.(irexec是lirc附带的一个工具),看看效果.

<strong>和python结合:</strong>


#安装pylirc库
apt-get install python-pylirc


修改lircrc文件

#prog 随便起一个名字,后面会用到
begin
    prog = myapp
    button = KEY_1
    config = hello
end
begin
    prog = myapp
    button = KEY_2
    config = hello,world
end


测试python
[code lang="python"]
import pylirc

#初始化,这个的myapp需要和lircrc中的prog的名称一致
pylirc.init("myapp", "/etc/lirc/lircrc", 1)

#设置成阻塞模式
pylirc.blocking(1)

#按下按键就能看到对应的config值,如果没有匹配的key则显示为None
print pylirc.nextcode()

```


至此,就介绍完了红外控制树莓派的方法.就等你发挥你的想象力,be fun!
