---
layout: post
title: log4j源码学习
date: 2015-10-25 17:53
tags:
  - tech
  - java
  - log4j

---

![logging service](http://chengchao.name/resource-container/image/log4j-logo.jpg)

###说明
分析的版本是1.2.17,跟最新的log4j2可能有较大出入.[下载地址](https://logging.apache.org/log4j/1.2/download.html)
解压缩后,直接导入成maven工程即可. 
###代码结构
Log4J核心的类：

- Logger:最核心的类, Most logging operations, except configuration, are done through this class.
- Level:日志级别
- Appender:决定日志输出的目的地
- Layout:日志格式

###测试代码
```java
import org.apache.log4j.BasicConfigurator;
import org.apache.log4j.Logger;

public class Test {

    public static Logger logger = Logger.getLogger(Test.class);

    public static void main(String[] args) throws Exception {
        BasicConfigurator.configure();
        logger.debug("Hello world.");
        logger.info("What a beatiful day.");
    }

}

```

输出:

```
0 [main] DEBUG log4jtest.Test  - Hello world.
0 [main] INFO log4jtest.Test  - What a beatiful day.
```
这是最简单的测试代码,不需要配置文件.

###配置文件加载
更多情况下,我们需要使用自己的配置文件,那配置文件又是怎么加载的呢?其实也很简单,在启动的时候org.apache.log4j.LogManager中的static方法会加载log4j的配置文件,如果指定了log4j.configuration,则从指定的文件加载,否则就先读取log4j.xml格式的配置文件,如果没有再读取log4j.properties的文件,如果需要测试此特性,可以在运行的时候加上`-Dlog4j.configuration=myLog4jConfig.properties`即可.  

```java
    static {
        // By default we use a DefaultRepositorySelector which always returns 'h'.
        Hierarchy h = new Hierarchy(new RootLogger((Level) Level.DEBUG));
        repositorySelector = new DefaultRepositorySelector(h);
        /** Search for the properties file log4j.properties in the CLASSPATH. */
        String override = OptionConverter.getSystemProperty(DEFAULT_INIT_OVERRIDE_KEY, null);

        // if there is no default init override, then get the resource
        // specified by the user or the default config file.
        if (override == null || "false".equalsIgnoreCase(override)) {

            String configurationOptionStr = OptionConverter.getSystemProperty(DEFAULT_CONFIGURATION_KEY, null);
            String configuratorClassName = OptionConverter.getSystemProperty(CONFIGURATOR_CLASS_KEY, null);
            URL url = null;
            // if the user has not specified the log4j.configuration
            // property, we search first for the file "log4j.xml" and then
            // "log4j.properties"
            if (configurationOptionStr == null) {
                url = Loader.getResource(DEFAULT_XML_CONFIGURATION_FILE);
                if (url == null) {
                    url = Loader.getResource(DEFAULT_CONFIGURATION_FILE);
                }
            } else {
                try {
                    url = new URL(configurationOptionStr);
                } catch (MalformedURLException ex) {
                    // so, resource is not a URL:
                    // attempt to get the resource from the class path
                    url = Loader.getResource(configurationOptionStr);
                }
            }
            // If we have a non-null url, then delegate the rest of the
            // configuration to the OptionConverter.selectAndConfigure
            // method.
            if (url != null) {
                LogLog.debug("Using URL [" + url + "] for automatic log4j configuration.");
                try {
                    OptionConverter.selectAndConfigure(url, configuratorClassName, LogManager.getLoggerRepository());
                } catch (NoClassDefFoundError e) {
                    LogLog.warn("Error during default initialization", e);
                }
            } else {
                LogLog.debug("Could not find resource: [" + configurationOptionStr + "].");
            }
        } else {
            LogLog.debug("Default initialization of overridden by " + DEFAULT_INIT_OVERRIDE_KEY + "property.");
        }
    }
```

不管是从哪个入口进入,最终会调用`org.apache.log4j.spi.Configurator.doConfigure()`方法.


###Spring中的初始化
在很多情况下是结合spring使用的,在web.xml中的配置文件如下:

```xml
	<context-param>
		<param-name>log4jConfigLocation</param-name>
		<param-value>classpath:log4j.xml</param-value>
	</context-param>

	<listener>
		<listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
	</listener>
```

从源代码中可以看出就是调用log4j的PropertyConfigurator.configure方法.  

```java
	public static void initLogging(String location) throws FileNotFoundException {
		String resolvedLocation = SystemPropertyUtils.resolvePlaceholders(location);
		URL url = ResourceUtils.getURL(resolvedLocation);
		if (ResourceUtils.URL_PROTOCOL_FILE.equals(url.getProtocol()) && !ResourceUtils.getFile(url).exists()) {
			throw new FileNotFoundException("Log4j config file [" + resolvedLocation + "] not found");
		}

		if (resolvedLocation.toLowerCase().endsWith(XML_FILE_EXTENSION)) {
			DOMConfigurator.configure(url);
		}
		else {
			PropertyConfigurator.configure(url);
		}
	}
```

###如何工作

当我们调用`logger.info("msg")`的时候,到底是怎么发生的呢?  
拿最简单的org.apache.log4j.ConsoleAppender举例子

第一步:org.apache.log4j.Category.info(Object)

```java
public void info(Object message) {
        if (repository.isDisabled(Level.INFO_INT)) 
           return;
        if (Level.INFO.isGreaterOrEqual(this.getEffectiveLevel())) 
           forcedLog(FQCN, Level.INFO, message, null);
    }
```
第二步:一直会调用父类中的`c.aai.appendLoopOnAppenders(event);`,除非把c.additive设置成false,默认为true

```java
	protected void forcedLog(String fqcn, Priority level, Object message, Throwable t) {
        callAppenders(new LoggingEvent(fqcn, this, level, message, t));
    }
    public void callAppenders(LoggingEvent event) {
        int writes = 0;
        for (Category c = this; c != null; c = c.parent) {
            // Protected against simultaneous call to addAppender, removeAppender,...
            synchronized (c) {
                if (c.aai != null) {
                    writes += c.aai.appendLoopOnAppenders(event);
                }
                if (!c.additive) {
                    break;
                }
            }
        }
        if (writes == 0) {
            repository.emitNoAppenderWarning(this);
        }
    }
```

第三步:

```java
    public int appendLoopOnAppenders(LoggingEvent event) {
        int size = 0;
        Appender appender;
        if (appenderList != null) {
            size = appenderList.size();
            for (int i = 0; i < size; i++) {
                appender = (Appender) appenderList.elementAt(i);
                appender.doAppend(event);
            }
        }
        return size;
    }
```
第四步:调用真正的logger的doAppend方法

```java

  // Reminder: the nesting of calls is:
  //
  //    doAppend()
  //      - check threshold
  //      - filter
  //      - append();
  //        - checkEntryConditions();
  //        - subAppend();
	public void append(LoggingEvent event) {
	    if (!checkEntryConditions()) {
	        return;
	    }
	    subAppend(event);
	}
  //由于ConsoleAppender没有继承subAppend方法,所以实际就是调用了WriterAppender的subAppend方法
  	protected void subAppend(LoggingEvent event) {
	    this.qw.write(this.layout.format(event));
	    if (layout.ignoresThrowable()) {
	        String[] s = event.getThrowableStrRep();
	        if (s != null) {
	            int len = s.length;
	            for (int i = 0; i < len; i++) {
	                this.qw.write(s[i]);
	                this.qw.write(Layout.LINE_SEP);
	            }
	        }
	    }
	    if (shouldFlush(event)) {
	        this.qw.flush();
	    }
	}
	//其中的qw由ConsoleAppender初始化
	public void activateOptions() {
        if (follow) {
            if (target.equals(SYSTEM_ERR)) {
               setWriter(createWriter(new SystemErrStream()));
            } else {
               setWriter(createWriter(new SystemOutStream()));
            }
        } else {
            if (target.equals(SYSTEM_ERR)) {
               setWriter(createWriter(System.err));
            } else {
               setWriter(createWriter(System.out));
            }
        }

        super.activateOptions();
  }
    
```
通过这几个步骤就能把日志打印出来了.
