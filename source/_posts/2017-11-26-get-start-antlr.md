---
title: ANTLR 入门(未完待续)
date: 2017-11-26 11:08:02
tags:
  - java
  - tech
  
---


### ANTLR是什么

>ANTLR (ANother Tool for Language Recognition) is a powerful parser generator for reading, processing, executing, or translating structured text or binary files. It's widely used to build languages, tools, and frameworks. From a grammar, ANTLR generates a parser that can build and walk parse trees.


简单的说就是一个语言识别器,什么时候会用到呢?当你想构建自己的DSL(Domain Specific Language)就会用到了.编译语言的前面的步骤就是词法分析,和语法分析.而ANTLR就能帮助我们简化这个过程,通过简单的配置就能生成词法分析器和语法分析器,极大提高了效率.


### 快速开始

安装和简单入门都很简单,官网有[详细介绍](https://github.com/antlr/antlr4/blob/master/doc/getting-started.md)

最重要的资源是他们的[官方的例子](https://github.com/antlr/grammars-v4),模仿是学习新东西很好的一个手段.除了这些我还推荐一个超级好用的插件,一个IntelliJ Idea的插件,貌似也是官方的,地址在[这里](https://github.com/antlr/intellij-plugin-v4)


### 练习
我之前写过一个简单的[四则运算的解析器](http://blog.chengchao.name/2013/04/14/java-binary-tree/),现在我们尝试用ANTLR实现一遍,先来个最简单的版本,先建一个Calculator.g4的文件,内容如下:

```
grammar Calculator;

// SKIP
SPACE:                               [ \t\r\n]+    -> channel(HIDDEN);

expr:   INT (op INT)*
    ;
op
    : AVG|MAX|MIN|SUM|COUNT
   ;

MUL :   '*' ;
DIV :   '/' ;
ADD :   '+' ;
SUB :   '-' ;
INT :   [0-9]+ ;

```
