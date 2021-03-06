---
title: 生日提醒服务
date: 2016-05-18 08:07:30
tags:
  - tech
  - life
  - tool 
 
---

>更新: 现在可以用更简单的方式来使用这个服务了,用微信扫一扫下面这个小程序码

![](http://www.chengchao.name/resource-container/image/xiaochengxu_code.jpg)


### 起因
由于家里人比较多,总是记不住每个人的生日.阳历生日还比较好办,直接在日历中设置一个**循环事件**就可以了.关键是农历生日就比较麻烦了(我老家还是习惯过农历生日),基本上现在所有的日历应用都不支持农历的循环提醒功能.本着万事问Google的原则,搜寻了一番,都没有找到满意的解决方案.于是就准备自己动手.


### 动手
在开始之前,我给自己提了几点要求:

- 操作简单:农历或者阳历只需录入一项
- 手机上能提前(一般是提前一天)收到提醒
- 能方便的查询最近(一般是最近一个月)要生日的人
- 能跨平台(mac,windows,iphone,android)支持
- 一劳永逸:设定一次后就能永远自动提醒

想好要求就后,实现就比较简单了.  
首先找了一个[农历阳历转换的组件](https://github.com/isee15/Lunar-Solar-Calendar-Converter).这个还不错,提供了很多语言的版本.  
对象模型很简单

```java
//姓名
String  name;

//农历
String  birthdayLunar;

//阳历
String  birthday;
```

开发一个简单的编辑界面+定时任务,再结合*钉钉*和*微信公众号*的消息收发功能就基本能满足我个人的需求了.但问题来了,如何开放给别人用呢?  
无意中看到iphone中的*日历应用*中有个*中国节假日*的[订阅](https://p36-calendars.icloud.com/holidays/cn_zh.ics).看了一下url是一个以ics结尾的东东.于是去了解了一下,发现这个就是我想要的东东.使用的是iCalendar格式

- iCalendar的[wiki介绍](iCalendar)
- iCalendar的[官网](http://icalendar.org),这里有比较详细的介绍

原理很简单就是生成一份符合iCalendar格式规范的文本就行了.

```
//http的Content-Type
Content-Type: text/calendar;charset=UTF-8

//ics示例
BEGIN:VCALENDAR
VERSION:2.0
PRODID:ichengchao-cal
CALSCALE:GREGORIAN
X-WR-CALNAME:家人生日
X-APPLE-LANGUAGE:zh
X-APPLE-REGION:CN
BEGIN:VEVENT
DTSTAMP:20160509T133150Z
UID:ichengchaobaf8a110307e4dda9c23a3097aad6694
DTSTART;VALUE=DATE:20160901
SUMMARY;LANGUAGE=zh_CN:张三[阳历]
DESCRIPTION:张三,阳历生日[1990-9-1]
END:VEVENT
END:VCALENDAR
```



### 客户端订阅

##### iphone

设置  -> 密码与账户 -> 添加帐号

![](http://chengchao.name/resource-container/image/iphone_add_calendar.jpg)



##### Mac

Calendar -> File -> New Calendar Subscription

![](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/birthday-reminder_mac_subscription.png)

我开放了这个服务,如果你也有需要的话可以试试.[服务地址](http://www.chengchao.name/springrun/page/birthday/birthday.html)

