---
title: 自定义Jar,War中的MANIFEST.MF
date: 2019-10-31 12:29:41
tags:

 - java
 - tech
---

分享一个小技巧,我们发布的时候需要确认发布的版本是不是最新打包的.可以利用自定义MANIFEST.MF来解决.总的思路就是在maven打包的时候自动加入版本信息打在MANIFEST.MF中,然后暴露Http接口来查看对应的版本号.

首先修改pom.xml,如果是war包就包插件换成`maven-war-plugin`

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <archive>
                    <index>true</index>
                    <manifest>
                        <addClasspath>true</addClasspath>
                    </manifest>
                    <manifestEntries>
                        <version>${myversion}</version>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后打包的时候加入myversion参数

```
mvn clean package -Dmyversion=20191111
```
这样在jar包中的META-INF/MANIFEST.MF中就有version字段了.读取的话可以使用jcabi-manifests,当然自己写也很简单

```xml
<dependency>
    <groupId>com.jcabi</groupId>
    <artifactId>jcabi-manifests</artifactId>
    <version>0.7.5</version>
</dependency>
```

如果是war包的话,可以看看[这篇文章](https://dzone.com/articles/get-handle-manifestmf-webapp)

```java
ServletContext application = getServletConfig().getServletContext();
InputStream inputStream = application.getResourceAsStream("/META-INF/MANIFEST.MF");
Manifest manifest = new Manifest(inputStream);
//get version
manifest.getMainAttributes().getValue("Version");
```
这样就能用来check发布到线上的版本是不是刚刚打出来的版本.毕竟发布系统有时候也会有bug的.^_^

