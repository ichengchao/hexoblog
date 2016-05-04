---
layout: post
title: raspberry pi led
date: 2013-05-23 22:15
tags:
  - raspberrypi
  - tech
  
---


web控制led
---

[视频链接](http://you.video.sina.com.cn/api/sinawebApi/outplayrefer.php/vid=104817419_0/s.swf)

代码(非常简单):

```python
import web
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BCM)
GPIO.setup(18, GPIO.OUT)
urls = (
    '/(.*)', 'hello'
)
app = web.application(urls, globals())
class hello:
    def GET(self, name):
        if name == 'on':
                name = 'test'
                GPIO.output(18,GPIO.HIGH)
        if name == 'off':
                GPIO.output(18,GPIO.LOW)
        if not name:
                name = 'World'
        return 'Hello, ' + name + '!'
if __name__ == "__main__":
    app.run()
```
