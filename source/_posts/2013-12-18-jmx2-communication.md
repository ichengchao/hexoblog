---
layout: post
title: JMX2:交互
date: 2013-12-18 15:29
author: charles
comments: true
tags:
 - java
 - jmx
 - tech
---

在上一篇的hello world的例子中我们是用jconsole来操作JMX的.
那是不是还有其他的交互方式呢.答案是肯定的.下面我们来看看都有哪些交互方式
官方的文档可以参考:
<http://docs.oracle.com/javase/6/docs/technotes/guides/management/agent.html>

<strong>1.jconsole(上一节已经示范过)</strong>

<strong>2.同一个jvm下用代码访问:</strong>

```java
MBeanServer mbeanServer = ManagementFactory.getPlatformMBeanServer();
ObjectName objectName = new ObjectName("name.chengchao:type=MyTest");
//相当于调用getName方法
Object value = mbeanServer.getAttribute(objectName, "Name");
//相当于调用setName方法
Attribute attribute = new Attribute("Name", "superMan");
mbeanServer.setAttribute(objectName, attribute);
//调用sayHello方法
Object opParams[] = { "niubi" };
String signature[] = { String.class.getName() };
String sayHello = (String) mbeanServer.invoke(objectName, "sayHello", opParams, signature);
```

<strong>3.写一个adapterServer,这里用的的是HtmlAdaptorServer</strong>

```xml
<!-- pom.xml添加依赖 -->
<dependency>
	<groupId>org.glassfish.external</groupId>
	<artifactId>opendmk_jdmkrt_jar</artifactId>
	<version>1.0-b01-ea</version>
</dependency>
```  

```java
public class Main {

    public static void main(String args[]) throws Exception {
        MBeanServer mBeanServer = ManagementFactory.getPlatformMBeanServer();
        // 包名加 类名 创建一个ObjectName
        ObjectName name = new ObjectName("name.chengchao:type=MyTest");
        MyTest mbean = new MyTest();
        // 在MBean server上注册MBean
        mBeanServer.registerMBean(mbean, name);

        HtmlAdaptorServer adapter = new HtmlAdaptorServer();
        ObjectName adapterName = null;
        adapterName = new ObjectName("name.chengchao:type=htmladapter");
        adapter.setPort(10000);
        mBeanServer.registerMBean(adapter, adapterName);
        adapter.start();

        System.out.println("started");
        // 线程等待 management 操作
        Thread.sleep(Long.MAX_VALUE);
    }

}
```

打开浏览器: http://localhost:10000
可以看到一个web版的mbean管理界面,操作基本和jconsole类似

<strong>4.外部调用</strong>
在vm 参数中增加:

```java
-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote.port=1100
-Djava.rmi.server.hostname=127.0.0.1
```

打开jconsole,在远程进程中输入127.0.0.1:1100,直接进入管理界面

代码外部调用  

```java
public class RemoteMBean {

    public static void main(String[] args) throws Exception {
        JMXServiceURL url = new JMXServiceURL("service:jmx:rmi:///jndi/rmi://127.0.0.1:1100/jmxrmi");
        JMXConnector connector = JMXConnectorFactory.connect(url);
        MBeanServerConnection remote = connector.getMBeanServerConnection();
        ObjectName objectName = new ObjectName("name.chengchao:type=MyTest");
        Object value = remote.getAttribute(objectName, "Name");
        System.out.println(value);
    }

}

```


<strong>附加:jetty开启jmx</strong>
我用的是runjettyrun插件
把jetty-jmx.xml的注释部分打开,启动后打开jconsole在远程进程中输入:
`service:jmx:rmi://localhost:1099/jndi/rmi://localhost:1099/jmxrmi`
这样也能操作mbean
