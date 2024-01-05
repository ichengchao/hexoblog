---
title: Spring Boot应用获取编译时间戳
date: 2024-01-04 12:20:00
tags:

 - tech
 - springboot
 - java

---



# 背景

平时发布应用的时候需要检测一下发布的版本是不是对的,最好的方式就是通过URL暴露一个编译时间出来.具体做法有很多种,大致的思路就是在maven打包的时候塞入或者自动生成一个时间戳到配置文件中

#  方式一: maven参数


在pom.xml文件中增加一个plugin的配置

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifestEntries>
        <Version>${myversion}</Version>
        <Owner>${owner}</Owner>
      </manifestEntries>
    </archive>
  </configuration>
</plugin>
```

这样只要在maven打包的时候传入指定的参数就能在MANIFEST.MF中生成对应的配置

```
mvn clean package -Dmyversion=20240104 -Downer=charles

# 生成的MANIFEST.MF文件内容

Manifest-Version: 1.0
Created-By: Maven JAR Plugin 3.2.2
Build-Jdk-Spec: 17
Implementation-Title: springrun
Implementation-Version: 0.0.1-SNAPSHOT
Owner: charles
Version: 20240104
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: name.chengchao.springrun.SpringrunApplication
Spring-Boot-Version: 2.5.14
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
```

获取MANIFEST.MF的配置项

```java
ApplicationHome applicationHome = new ApplicationHome(getClass());
File jarFile = applicationHome.getSource();
Manifest manifest = new Manifest(jarFile.getManifest());
String version = manifest.getMainAttributes().getValue("Version");
String owner = manifest.getMainAttributes().getValue("Owner");
```

# 方式二: SpringBoot的内置方法

在pom.xml增加一个配置

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <executions>
    <execution>
      <id>build-info</id>
      <goals>
        <goal>build-info</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

执行maven打包后会生成一个build-info.properties的文件

```java
//获取对应的配置项
@Autowired
private BuildProperties buildProperties;
//这个就是打包的时间戳
Instant instant = buildProperties.getTime();
```

可以看到,整个过程比方式一要优雅很多,简单很多
