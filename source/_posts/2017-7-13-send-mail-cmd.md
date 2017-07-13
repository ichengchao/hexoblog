---
title: OpenJDK源码下载
date: 2017-07-13 23:00:00
tags:
  - tech
  - java

---


用Eclipse翻看代码的时候,经常看着看着就看到了rt.jar中的代码.里面有部分代码不是开源的.所以默认看不到源码.不过可以看OpenJDK的代码,两者差别非常小.[下载地址](http://hg.openjdk.java.net/).找到对应的版本,比如JDK8的[链接](http://hg.openjdk.java.net/jdk8/jdk8/jdk/).在左侧下载zip包导入到Eclipse中就能看了.

这是RednaxelaFX对SunJDK和OpenJDK的解释,[链接](https://www.zhihu.com/question/19646618)

>Sun JDK能用于商业用途的license是SCSL（Sun Community Source License）。JRL（Java Research License）是2004年开始用的，伴随Sun JDK6发布而开始使用，远比JDK7早。

>从代码完整性来说，
Sun JDK > SCSL > JRL > OpenJDK
Sun JDK有少量代码是完全不开发的，即便在SCSL版里也没有。但这种代码非常非常少。
SCSL代码比JRL多一些closed目录里的内容。
JRL比OpenJDK多一些受license影响而无法以GPLv2开放的内容。

>但从Oracle JDK7 / OpenJDK7开始，闭源和开源版的实质差异实在是非常非常小。与其说OpenJDK7是“不完整的JDK”，还不如说Oracle JDK7在OpenJDK7的基础上带了一些value-add，其中很多还没啥用（例如browser plugin）