---
layout: post
title: lubuntu的环境配置
date: 2015-09-10 22:58
tags:
  - tech
  - linux
  - ubuntu

---

###虚拟机安装

[VisualBox](https://www.virtualbox.org)  
虽然有很多其他的选择,但是这个已经够用了,而且免费

###Linux版本选择
之前一直用ubuntu,但是感觉现在的ubuntu越来越臃肿,而且UI也越来越丑.于是就想找一个比较轻量级的版本.  
发现[Lubuntu](http://lubuntu.net/)完全能满足我的要求

###安装须知
虚拟机中的安装方式就不详细介绍了,就是一步步往下点就行了.
有一点需要注意的是:安装的时候语言选English,选中文的话会有乱码.
安装完系统后,在设备-->安装增强功能.点击后会挂载一个ISO,之后在命令行运行. 

###语言配置
按照[ubuntu中文字体](http://wiki.ubuntu.com.cn/%E5%AD%97%E4%BD%93)安装中文字体

```sh
sudo apt-get install ttf-wqy-microhei  #文泉驿-微米黑
```

在setting中把字体language support中设置成`汉语-->apply-->重启`  

###系统配置
重新安装vim

```sh
sudo apt-get install vim
```

安装vim的插件管理[vim-pathogen](https://github.com/tpope/vim-pathogen),用来管理VIM的插件,然后安装一个[nerdtree](https://github.com/scrooloose/nerdtree)插件试试,把tab换成4个空格

```sh
execute pathogen#infect()
syntax on
filetype plugin indent on

set tabstop=4
set shiftwidth=4
set expandtab
```

设置.bashrc或者.bash_profile,把自己喜欢的配置写好  
我一般修改两个配置

```sh
alias ll='ls -lF'
export PS1='\[\033[01;32m\]\u@\h\[\033[01;36m\]:\w\[\e[01;31m\]$\[\033[00m\] '
```