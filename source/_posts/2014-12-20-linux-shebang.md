---
layout: post
title: Linux上的Shebang符号(#!)
date: 2014-12-20 10:41
tags:
  - linux
  - tech
  
---

词源与历史
---
Shebang的名字来自于SHArp和bang，或haSH bang的缩写，指代Shebang中#!两个符号的典型Unix名称。 Unix术语中，井号通常称为sharp，hash或mesh；而叹号则常常称为bang。也有看法认为，shebang名字中的sh来自于默认shell Bourne shell的名称，sh，因为常常使用shebang调用之。
在2010年版的Advanced bash scripting guide（revision 6.2）中，shebang被称为"sha-bang"，同时提到"也写作she-bang或sh-bang"，但该文件中没有提到"shebang"这一形式。

用途
---

Shebang这个符号通常在Unix系统的脚本中第一行开头中写到，它指明了执行这个脚本文件的解释程序。

1. 如果脚本文件中没有#!这一行，那么它执行时会默认用当前Shell去解释这个脚本(即：$SHELL环境变量）。
2. 如果#!之后的解释程序是一个可执行文件，那么执行这个脚本时，它就会把文件名及其参数一起作为参数传给那个解释程序去执行。
3. 如果#!指定的解释程序没有可执行权限，则会报错“bad interpreter: Permission denied”。如果#!指定的解释程序不是一个可执行文件，那么指定的解释程序会被忽略，转而交给当前的SHELL去执行这个脚本。
4. 如果#!指定的解释程序不存在，那么会报错“bad interpreter: No such file or directory”。注意：#!之后的解释程序，需要写其绝对路径（如：#!/bin/bash），它是不会自动到$PATH中寻找解释器的。
5. 当然，如果你使用”bash test.sh”这样的命令来执行脚本，那么#!这一行将会被忽略掉，解释器当然是用命令行中显式指定的bash。