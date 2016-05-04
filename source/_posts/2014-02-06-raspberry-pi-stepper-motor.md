---
layout: post
title: raspberry pi控制步进电机+摄像头
date: 2014-02-06 22:09
author: charles
comments: true
tags:
 - raspberrypi
 
---

###控制代码###

```python
#http 服务使用web.py实现
import web
import RPi.GPIO as GPIO
import time
import sys
import os
from array import *

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(18, GPIO.OUT)
GPIO.setup(22, GPIO.OUT)
GPIO.setup(23, GPIO.OUT)
GPIO.setup(24, GPIO.OUT)
GPIO.setup(25, GPIO.OUT)

urls = (
    '/(.*)', 'hello'
)
app = web.application(urls, globals())

arr = [0,1,2,3]
#arr = [3,2,1,0]
ports = [22,23,24,25]
def move(d):
        if d=='left':
                arr = [0,1,2,3]
        if d=='right':
                arr = [3,2,1,0]
        for x in range(0,40):
                for j in arr:
                        time.sleep(0.01)
                        for i in range(0,4):
                                if i == j:            
                                        GPIO.output(ports[i],True)
                                else:
                                        GPIO.output(ports[i],False)


class hello:
    def GET(self, name):
        if name == 'on':
                name = 'test'
                GPIO.output(18,GPIO.HIGH)
        if name == 'off':
                GPIO.output(18,GPIO.LOW)
        if name == 'take':
                print 'take a photo to /var/www/'
                os.system('sudo /usr/bin/fswebcam -r 320x240 --no-banner /var/www/a.jpg')
        if name == 'moveleft':
                print 'move left'
                move('left')
        if name == 'moveright':
                print 'move right'
                move('right')
        if not name:
                name = 'World'
        return 'Hello, ' + name + '!'

if __name__ == "__main__":
    app.run()
	
	
```
<br/>
###页面代码###

```html
<html>
<head>
<script src="http://libs.baidu.com/jquery/1.9.0/jquery.js"></script>
<script type="text/javascript">
        var httpquery = function(action) {
                $.ajax({
                        url : "http://192.168.1.98:8080/" + action,
                        success : function(data) {
                                $("#mydiv").html('success'+data);
                        }
                });
        }
        var refreshIframe = function() {
                var timestamp = Date.parse(new Date());
                var myframe = document.getElementById('iframe');
                myframe.setAttribute('src', 'a.jpg?test=' + timestamp);
        }
        window.onload = function(){
                $("#mydiv").html(new Date());
        }
</script>
</head>
<body>
        <div id="mydiv">msg...</div>
        <h2>Welcome to smart world!</h2>
        <br />
        <button onclick="httpquery('moveright');">Move Right</button>
        &nbsp;
        <button onclick="httpquery('moveleft');">Move Left</button>
        &nbsp;
        <button onclick="httpquery('take');">Take Photo</button>
        &nbsp;
        <button onclick="refreshIframe();">Refresh</button>
        &nbsp;
        <button onclick="httpquery('on');">On</button>
        &nbsp;
        <button onclick="httpquery('off');">off</button>
        <br />
        <hr />

        <iframe id="iframe" frameborder="0" src="a.jpg"
                height="300" width="400"></iframe>

</body>
</html>

```