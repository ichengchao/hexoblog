---
title: 从wordpress到jekyll
layout: post
date: 2015-02-05 17:49
tags:
  - tech
  - blog
  
---

![jekyll](http://chengchao.name/resource-container/image/logo-jekyll.png)

###起因
我也算是比较早接触blog的,那时还没有新浪博客(当然现在也快没了),当时觉得写blog的人都很高大上.后来越来越多的人都开始写blog,想到自己是不是也可以开个博客玩玩.最早从blogcn开始,到后来的网易个人空间,一直觉得不是很满意.那时甚至想买个虚拟主机自己搭blog,不过当时虚拟主机又贵又不好用于是作罢.后来偶然间用了一下blogger,记得当时已经被google收购,感觉很满意,用邮件发文也很方便,还是自动同步到个人的FTP.心想这才是我想要的blog.于是开始乱写一通,断断续续坚持了大半年,再后来被GFW封了,然后就没有然后了.就这样很长一段时间都没再碰过blog,直到阿里云开始卖虚机,我抱着试试看的心态买了一台.买来当然不忍心这么空着,第一个能想到的就是搭个blog,赶紧装了mysql,apache,wordpress.跑在自己主机上的感觉还真不不错.换主题,装插件,玩得不亦乐乎.使用过程中有些小问题,比如垃圾评论特别多,升级频率太过于频繁,页面加载有点慢.虽然感觉有些许别扭但还是忍着没继续折腾.直到一次偶然的机会看到了[jekyll](http://jekyllrb.com/),号称是`为coder而生的blog`,看了看介绍感觉blog还能这么玩,于是又开始折腾.
当然类似的还有很多,比如[Hexo](https://hexo.io/)  
  

###行动
怎么搭建的就不说了,网上多的是,没有github账号的申请一下.然后按照教程一步步下去就ok了.最重要的还是迁移wordpress的数据.虽然jekyll本身就提供了原生的迁移工具,但我觉得还是太麻烦了.上网搜寻了一下,找到一个很轻量级的从wordpress的xml文件生成jekyll的markdown文件的工具:[wpXml2Jekyll](https://github.com/theaob/wpXml2Jekyll),目前只提供windows版本.生成后的markdown文件基本不用改动就能用了.当然像语法高亮这种还是需要手工调整的.找了一套简洁的模版,做了些修改就用上了.整个blog的源码在[Github](https://github.com/ichengchao/ichengchao.github.io)上都有.需要的请自取.  
  

###感受
自从换成jekyll后,页面全是静态的,渲染速度快了很多.不过github page的访问速度总是时好时坏.我就干脆在自己服务器上也同步一份.用两个二级域名分别绑定.
同时把原来的模本中的一些远程的css和js引用全部换成了自己域名的引用,访问速度就更快了.还有很重要的一点没解决,就是blog中的图片该保存在哪里?原来用wordpress的时候,自带的附件上传的功能.所以没有遇到这个问题.换成jekyll后就开始寻找允许外链的图床服务.什么百度云,网易相册,腾讯通通都没有提供免费的图床服务.后来看到`七牛`有个免费的图床服务,就试了一段时间,感觉还挺不错,图床管理界面中还能看到各种各样的访问统计数据.但放在那里总觉得不是很安心,一个是上传操作比较麻烦,另一个是万一哪天七牛把这个服务停了,又得整理一遍图片链接.想了想,还是放在github上觉得靠谱,于是就新建了一个project专门放图片.再同步到自己服务器,perfect!至此,blog的折腾就告一段落了,毫不夸张的说,现在这个blog方案是我最满意的方案,没有之一.要不信你也可以试试！

###jekyll使用后记

安装jekyll,由于使用了[jekyllbootstrap](http://jekyllbootstrap.com),不兼容最新的jekyll 3.0,所以就安装一个旧一点的版本.

```sh
#更换gem的source,使用aliyun mirrors:
gem source -r https://rubygems.org/
gem source -a http://mirrors.aliyun.com/rubygems/

#安装jekyll 2.5.3 版本
sudo gem install jekyll -v '2.5.3'

#测试
jekyll --version

#bulid
jekyll b

#本地运行
jekyll serve
```

由于内置了RakeFile,所以还可以使用rake命令,比如:

```sh
rake "post[hello-world]"
rake "deploy"
```
