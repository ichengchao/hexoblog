---
layout: post
title:  Arduino温度采集+展示
date: 2013-05-19 11:14
author: charles
comments: true
tags:
 - arduino
 - tech
 
---

前几天看到有同事用树莓派搞了一个温度采集的东东.
自己手痒也忍不住搞了一个,
不过思路稍有不同,
我用的是arduino,采集的数据是直接发送到自己的aliyun服务器上.

所需材料:
<ul>
	<li>arduino uno</li>
	<li>面包板,杜邦线</li>
	<li>LM35D温度传感器</li>
	<li>扩展网络模块</li>
</ul>
连接好后的效果(连线比较简单):


该传感器能够测量0-100°的温度  
从0开始,每升高1度输出电压提高10mv(0.01v)，  
通过模拟口检测传感器的电压，就能计算出温度.  
模拟口的数值从0~1024对应输入电压0~5v  
所以输入电压的计算公式为: (5/1024)*in,单位是v  
换算成温度就是((5/1024)*in)/0.01  
```c
int in = analogRead(A0);    //读取A0口的电压值
float temp = ((5/1024)*in)/0.01 //计算温度
```

随后只要将采集的数据发送到服务器保留起来就大功告成了.  
完整的代码(从ide的example中改改就行了):  

```c
#include <SPI.h>
#include <Ethernet.h>
#include <FloatToString.h>

// Enter a MAC address for your controller below.
// Newer Ethernet shields have a MAC address printed on a sticker on the shield
byte mac[] = {  0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
IPAddress server(192.168.1.101);
EthernetClient client;
void setup() {
 
  // start the Ethernet connection:
  if (Ethernet.begin(mac) == 0) {
    //Serial.println("Failed to configure Ethernet using DHCP");
    // no point in carrying on, so do nothing forevermore:
    for(;;)
      ;
  }
  // give the Ethernet shield a second to initialize:
  delay(1000);
}
void loop()
{
  int n = analogRead(A0);    //读取A0口的电压值
  float temp = ((5/1024)*in)/0.01 //计算温度
  char buffer[25];
  String tempStr = floatToString(buffer,temp,2); //arduino比较变态的的类型转换
  if (client.connect(server, 8080)) {
    client.println("GET /sendTempData.htm?key=securityKey&tempNum="+tempStr);
    client.println();
  } 
  delay(1000);
  client.stop();
  delay(1800000); //半小时采集一次
}

```

服务端的代码比较简单就不贴出来了.
原始数据是半小时采集一次,加工了一下按小时,按天聚合出最大值,最小值,平均值
