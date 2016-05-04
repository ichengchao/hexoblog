---
title: (转摘)拷贝文件内容到剪切板
layout: post
tags:
  - os
  - tech
  - repaste
  
---

###为什么要这么做？
- 直接把命令的输出（比如grep/awk/sed/find或是你的程序输出结果）放到剪切板上，这么就可以在IM中CTRL + V粘贴后发出去。 
- 避免操作的繁琐和跳跃：把结果输出到文件、用文本编辑器打开文件、选中文本、CTRL + C。
通过命令将文件内容拷贝到剪切板，以避免拷贝错误、操作的跳跃（跳到文件编辑器）

###Windows
使用系统自带的clip命令
>位于C:\Windows\system32\clip.exe

```sh
# 将字符串Hello放入Windows剪贴板
echo Hello | clip 

# 将dir命令输出（当前目录列表）放入Windows剪贴板
dir | clip

# 将readme.txt的文本放入Windows剪贴板
clip < README.TXT   

# 将一个空行放入Windows剪贴板，即清空Windows剪贴板
echo | clip 


```

###Linux

使用xsel命令

```sh

# 将readme.txt的文本放入剪贴板
cat README.TXT | xsel
cat README.TXT | xsel -b # 如有问题可以试试-b选项
xsel < README.TXT 


# 清空剪贴板
xsel -c
```
###Mac
使用pbcopy命令,对应有个pbpaste命令

```sh
#将字符串Hello World放入剪贴板
echo 'Hello World!' | pbcopy

#复制SSH的公有KEY
pbcopy < ~/.ssh/id_rsa.pub

```

[原文链接](http://oldratlee.com/post/2012-12-23/command-output-to-clip)