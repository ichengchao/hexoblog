---
title: java web运行环境搭建
date: 2016-06-14 11:26:27
tags:
  - tech
  - java
  - tomcat
  - web

---

### tomcat 搭建
[官网下载地址](http://tomcat.apache.org/)

1. 启动`bin/catalina.sh start`,默认端口是8080,浏览器访问一下是否启动成功.  
2. 关闭`bin/catalina.sh stop`
3. 删除webapp目录中的所有文件
4. 把应用war包拷贝到webapp目录下,并命名成ROOT.war
5. 在bin目录下增加setenv.sh,里面有些参数可以根据自己的需要自定义
6. 关闭的时候可以使用`bin/catalina.sh stop -force`

```sh
#! /bin/sh

export CATALINA_OPTS="$CATALINA_OPTS -Xms800m"
export CATALINA_OPTS="$CATALINA_OPTS -Xmx800m"

### 测试环境的tomcat debug
#export CATALINA_OPTS="$CATALINA_OPTS -server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8888"


export CATALINA_OPTS="$CATALINA_OPTS -XX:PermSize=150m"
export CATALINA_OPTS="$CATALINA_OPTS -XX:MaxPermSize=150m"

export CATALINA_OPTS="$CATALINA_OPTS -XX:+UseParallelGC"

export CATALINA_PID="$CATALINA_HOME/catalina_pid.txt"

if [ -r "$CATALINA_BASE/bin/appenv.sh" ]; then
  . "$CATALINA_BASE/bin/appenv.sh"
fi

echo "Using CATALINA_OPTS:"
for arg in $CATALINA_OPTS
do
    echo ">> " $arg
done
echo ""

echo "Using JAVA_OPTS:"
for arg in $JAVA_OPTS
do
    echo ">> " $arg
done
echo "_______________________________________________"
echo ""
```

### java常用排查命令

```sh
# 获取tomcat pid,可以配合其他命令使用
`jps |grep Bootstrap | awk '{print $1}'`

# 查看jvm gc,2s打印一次
jstat -gcutil PID 2000

# 查看tcp连接情况
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

# 查看磁盘使用
df -h

# 查看内存使用
free -m

# 查看java进程
jstack PID

# 查看指定线程堆栈,一般是top中拷贝,H显示线程,P按cpu排序
jstack PID |grep `printf %x threadID` -A 10

# 也有现成的脚本
show-busy-java-threads.sh
https://github.com/oldratlee/useful-scripts/blob/master/show-busy-java-threads.sh

# dump内存,导出后用Memory Analyzer分析
jmap -dump:format=b,file=heap.bin PID


```
