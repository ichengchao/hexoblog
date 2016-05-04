---
title: Flask初识
layout: post
tags:
  - flask
  - python
  - tech
  
---

###环境初始化
- 安装 [python](https://www.python.org/)
- 安装 [pyenv](https://github.com/yyuu/pyenv),用于管理python的版本
- 安装 [pip](https://pip.pypa.io/en/stable/installing/),用于管理python的依赖,最好是在virtualenv的基础上安装

安装完python和pip之后最好安装一下[virtualenv](https://virtualenv.pypa.io/en/latest/).看名字就知道是虚拟环境,用来隔离系统的python环境用的.简单的说就像是虚拟机,可以在上面随便折腾.安装也很简单,直接用pip.

```sh
sudo pip install virtualenv

#指定一个目录,创建虚拟环境
mkdir pythonEnv
cd pythonEnv
virtualenv myvenv

#激活
. myvenv/bin/activate
```

命令行中会带`myvenv`,表示所有的python操作都在这个环境下进行.接着我们安装flask

```sh
pip install Flask
```
这样环境都初始化好了.

###最小应用
flask的[官方文档](http://flask.pocoo.org/docs/0.10/)中有最小应用的示例,稍加修改如下:

```python
from flask import Flask
import os
import logging

__author__ = 'charles'

app = Flask(__name__)

#设置日志
logHandler = logging.FileHandler('/Users/charles/myFlaskApp.log')
logHandler.setLevel(logging.INFO)
app.logger.addHandler(logHandler)
app.logger.setLevel(logging.INFO)


@app.route('/')
def hello():
    app.logger.info('log info test')
    return 'welcome'


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8888, debug=True)

```

直接在IDE中运行,就能通过`127.0.0.1:8888`访问了.

###用gunicorn部署
当然要在生成环境中部署的话,就需要使用容器了.比较有名的就是`gunicorn`

```sh
pip install gunicorn
#使用-i参数指定地址,使用阿里云mirrors
pip install gunicorn -i http://mirrors.aliyun.com/pypi/simple/

#部署
cd projectDir
gunicorn -w 1 myAppName:app --log-file test.log

#参数
-D 表示后台运行
-b 127.0.0.1:8000
```
