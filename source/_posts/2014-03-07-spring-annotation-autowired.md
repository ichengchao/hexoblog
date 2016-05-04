---
layout: post
title: 转摘-Spring中@Autowired标签与@Resource标签的区别
date: 2014-03-07 17:42
author: charles
comments: true
tags:
 - java
 - spring
 - tech
 - repaste
 
---

转摘:

```java

Spring不但支持自己定义的@Autowired注解，还支持由JSR-250规范定义的几个注解  
如：@Resource、 @PostConstruct及@PreDestroy。

1. @Autowired
    @Autowired是Spring 提供的，需导入
    Package:org.springframework.beans.factory.annotation.Autowired;
    只按照byType 注入。
2. @Resource
    @Resource默认按 byName 自动注入,是J2EE提供的， 需导入Package:  
    javax.annotation.Resource;
    @Resource有两个中重要的属性：name和type ，而Spring将@Resource注解的name属性解析为bean的
    名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用
    type属性时则使用 byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用by
    Name自动注入策略。

@Resource装配顺序
　 (1). 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常;
　 (2). 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常;
　 (3). 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常;
　 (4). 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一
    个原始类型进行匹配，如果匹配则自动装配；
    @Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入。
3. 使用区别
    @Resource(name="loginService") 
    private LoginService loginService;

@Autowired(required=false)@Qualifier("loginService") 
    private LoginService loginService;

(1).@Autowired 与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上;
   (2).@Autowired 默认按类型装配，默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设
    置它的required属性为false，如：@Autowired(required=false) .

如果我们想使用名称装配可以结合 @Qualifier注解进行使用;

    (3).@Resource（这个注解属于J2EE的），默认安装名称进行装配，名称可以通过name属性进行指定，如果没
    有指定name属性，当注解写在字段上时，默认取字段名进行安装名称查找，如果注解写在setter方法上默认取属
    性名进行装配。当找不到与名称匹配的bean时才按照类型进行装 配。但是需要注意的是，如果name属性一旦指
    定，就只会按照名称进行装配。

推荐使用@Resource注解在字段上，这样就不用写setter方法了.并且这个注解是属于J2EE的，减少了与Spring
    的耦合,这样代码看起就比较优雅 。

另外，通过实践，还总结出一条规律：

如果将@requied或者@autowired写了set方法之上，则程序会走到set方法内部。

但如果写在了field之上，则不会进入set方法当中。  

```