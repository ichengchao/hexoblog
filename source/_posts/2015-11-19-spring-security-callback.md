---
title: spring security中login的callback 
layout: post
tags:
  - spring
  - java
  - tech
  
---

###问题
在java web应用中,当使用spring security作为权限管理时,有个问题比较头疼.就是login之后的跳转.假设应用的域名为`www.chengchao.name`,当首次访问`http://www.chengchao.name/test.html`时,由于没有权限,会自动跳到默认的`www.chengchao.name/login.do`去做登录操作.之后spring security会自动调到默认配置的页面中.比如`www.chengchao.name/index.html`

```xml
<form-login login-page="/login.do" default-target-url="/index.html" always-use-default-target="true" />
```

这个页面是在spring security中设置的.但这个体验很不好,强制跳转后又得重新输入url,那怎么解决这个问题呢?

###解决

解决这个问题的办法也很简单,就是在登录完成后的login.do中加入callback参数,内容为原来的url,具体操作如下:  
注:`org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint`继承该类

```java

public class MyLoginUrlAuthenticationEntryPoint extends LoginUrlAuthenticationEntryPoint {

    private static Logger logger = LoggerFactory.getLogger(MyLoginUrlAuthenticationEntryPoint.class);

    public MyLoginUrlAuthenticationEntryPoint(String loginFormUrl){
        super(loginFormUrl);
    }

    protected String determineUrlToUseForThisRequest(HttpServletRequest request, HttpServletResponse response,
                                                     AuthenticationException exception) {
        String callBackUrl = request.getRequestURL().toString();
        if (StringUtils.isNotBlank(request.getQueryString())) {
            callBackUrl += "?" + request.getQueryString();
        }
        logger.debug("callBackUrl={}", callBackUrl);
        try {
            callBackUrl = URLEncoder.encode(callBackUrl, "utf8");
        } catch (Exception e) {
            logger.error(e.getMessage());
        }
        return getLoginFormUrl() + "?callBackUrl=" + callBackUrl;
    }

}
```

在spring security的xml配置如下

```xml
<b:bean id="loginUrlAuthenticationEntryPoint"
		class="name.chengchao.springtest.MyLoginUrlAuthenticationEntryPoint" >
		<b:constructor-arg value="/login.do"></b:constructor-arg>
	</b:bean>
```
