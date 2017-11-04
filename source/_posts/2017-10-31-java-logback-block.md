---
title: logback block
date: 2017-10-31 18:44:35
tags:
  - java
  - logback
  - tech
  
---

### 问题

最近线上项目碰到一个问题,在每天凌晨00:00:00的时候,执行任务就会比正常情况慢10s左右.由于这个任务每秒都会发生,而平时正常的执行时间也是1s内就能完成.直觉觉得不会是打日志导致的吧?因为除了定时任务和日志没有别的因素会导致这个问题.检查了一下定时任务的逻辑发现没有只在00:00:00这个时刻启动的.所以基本就可以排除了.

### 确定

接着用了一个比较极端的方法,直接把这个模块打日志给停掉了.等到零点一看,果然就好了.


### 分析

虽然知道是打日志导致的停顿,但具体是什么原因还是不清楚.我们配置的是异步日志,是不是就表示不应该会block住吗? 错! 先来看看我们的配置:


```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
	<appender name="ROLLING_FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${user.home}/logs/logTest.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${user.home}/logs/logTest.log.%d{yyyy-MM-dd}
			</fileNamePattern>
			<maxHistory>1</maxHistory>
		</rollingPolicy>
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<pattern>%date{ISO8601} %-5level [%X{traceId}][%thread] - %msg%n
			</pattern>
		</encoder>
	</appender>

	<appender name="ASYNC_ROLLING" class="ch.qos.logback.classic.AsyncAppender">
		<queueSize>256</queueSize>
		<discardingThreshold>0</discardingThreshold>
		<appender-ref ref="ROLLING_FILE" />
	</appender>

	<root>
		<level value="info" />
		<appender-ref ref="ASYNC_ROLLING" />
	</root>
</configuration>

```

配置其实没有啥问题,唯一比较特别就是`<discardingThreshold>0</discardingThreshold>`这个很表示不丢日志.对着logback的源代码看一看就知道了


```java
//class :  ch.qos.logback.core.AsyncAppenderBase 的 append方法

  @Override
  protected void append(E eventObject) {
    if (isQueueBelowDiscardingThreshold() && isDiscardable(eventObject)) {
      return;
    }
    preprocess(eventObject);
    put(eventObject);
  }
  
  private void put(E eventObject) {
    try {
      blockingQueue.put(eventObject);
    } catch (InterruptedException e) {
    }
  }


```

从代码中可以看出来虽然是异步的,但是把日志塞进队列中用的是put方法.是会block的,直到队列空出位置来.但为什么会卡10s这么久呢?我们可以想象一下日志切换瞬间logback做了什么事情:

1. 创建一个新文件,用来写新的日志
2. 重命名当前日志文件
3. 删除过期的日志文件


这三个事情最有可能有问题的就是第三步,直接在机器上(linux centos6.2 非ssd机器)测试了一下删除40GB的log文件,竟然花了10s.元凶终于确定.



### 解决

知道原因后解决就简单了,第一反应就是logback有点傻,这个删除动作完全可以异步做.上官网看了一下版本的更新记录,果然在1.1.7版本中已经把删除的动作改成异步了.

>March 29th, 2016, Release of version 1.1.7
>
Archive removal by RollingFileAppender is now performed asynchronously.

没啥犹豫的,直接把logback的版本升级到最新的1.2.3  Done!!


