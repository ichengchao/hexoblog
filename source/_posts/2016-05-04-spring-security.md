---
title: spring security
layout: post
tags:
  - java
  - tech
  - spring
  
---

### 背景知识
什么是spring security.官方解释:
>Spring Security provides comprehensive security services for Java EE-based enterprise software applications.   

简单的说就是给应用提供安全服务的.那安全服务要解决哪里问题呢?  
我认为主要是解决这两个问题:

* 你是谁? (authentication,身份验证)
* 你能干什么? (authorization,访问控制)

对应到spring security中两个重要的概念**authentication**和**authorization**

### Hello World
在一个使用spring框架的web应用中增加spring security是非常之简单的.  
第一步:在pom.xml中增加相应的依赖,这里以4.0.3为例.

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>4.0.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>4.0.3.RELEASE</version>
</dependency>
```

第二步:在web.xml中配置filter,需要注意的是这个配置中的**springSecurityFilterChain**这个名称是不能随便填的,后面再解释原因.

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

第三步:编写一个最简单的security配置,并import到已有的spring配置文件中.

```xml
<b:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:b="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">

    <http>
        <intercept-url pattern="/**" access="hasRole('USER')" />
        <form-login />
        <logout />
    </http>

    <authentication-manager>
        <authentication-provider>
            <user-service>
                <user name="admin" password="password" authorities="ROLE_USER" />
            </user-service>
        </authentication-provider>
    </authentication-manager>
</b:beans>
```

经过这三步配置,最简单的spring security的集成已经完成了.现在可以访问你的应用试试.用户名:admin,密码:passowrd

### 初窥
那spring security又是怎么工作的呢?这一切都得从web.xml的filter配置说起.spring security配置的filter是**org.springframework.web.filter.DelegatingFilterProxy**这是什么东东啊?看看这个类的javadoc解释就能明白:
>Proxy for a standard Servlet Filter, delegating to a Spring-managed bean that implements the Filter interface. 

这就是一个简单的代理类,代理spring container中实现了servlet filter的类,那如果很多实现了filter类该怎么找到想要的那个呢?通过**<filter-name>**中定义的名称.所以上面的web.xml配置中我提到了这个名称不能随意修改.而真正的spring security使用的filter是**org.springframework.security.filterChainProxy**.这个是proxy负责把所有的filter都串起来.下面是官方说明:


1. **ChannelProcessingFilter**, because it might need to redirect to a different protocol

2. **SecurityContextPersistenceFilter**, so a *SecurityContext* can be set up in the *SecurityContextHolder* at the beginning of a web request, and any changes to the *SecurityContext* can be copied to the *HttpSession* when the web request ends (ready for use with the next web request)

3. **ConcurrentSessionFilter**, because it uses the *SecurityContextHolder* functionality but needs to update the *SessionRegistry* to reflect ongoing requests from the principal

4. Authentication processing mechanisms **UsernamePasswordAuthenticationFilter**, **CasAuthenticationFilter**, **BasicAuthenticationFilter** etc - so that the *SecurityContextHolder* can be modified to contain a valid Authentication request token

5. The **SecurityContextHolderAwareRequestFilter**, if you are using it to install a Spring Security aware *HttpServletRequestWrapper* into your servlet container

6. **RememberMeAuthenticationFilter**, so that if no earlier authentication processing mechanism updated the *SecurityContextHolder*, and the request presents a cookie that enables remember-me services to take place, a suitable remembered Authentication object will be put there

7. **AnonymousAuthenticationFilter**, so that if no earlier authentication processing mechanism updated the *SecurityContextHolder*, an anonymous Authentication object will be put there

8. **ExceptionTranslationFilter**, to catch any Spring Security exceptions so that either an HTTP error response can be returned or an appropriate *AuthenticationEntryPoint* can be launched

9. **FilterSecurityInterceptor**, to protect web URIs and raise exceptions when access is denied



### 核心对象

上面介绍了主要filter,下面来说说有哪些核心的对象.  
**SecurityContextHolder**是spring security中最基本的对象,用于存储当前用户的所有安全相关的信息.默认使用ThreadLocal,也可以用SecurityContextHolder修改策略.大部分情况下是不需要修改的.这个类最常用的功能就是获取当前用户的信息,以下是官方提供的例子:

```java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
  String username = ((UserDetails)principal).getUsername();
} else {
  String username = principal.toString();
}
```

**org.springframework.security.core.Authentication**这个接口类也比较重要,里面存放了用户名,密码,以及用户拥有的权限列表.下面是官方的解释:

- **SecurityContextHolder**, to provide access to the SecurityContext.
- **SecurityContext**, to hold the Authentication and possibly request-specific security information.
- **Authentication**, to represent the principal in a Spring Security-specific manner.
- **GrantedAuthority**, to reflect the application-wide permissions granted to a principal.
- **UserDetails**, to provide the necessary information to build an Authentication object from your application’s DAOs or other source of security data.
- **UserDetailsService**, to create a UserDetails when passed in a String-based username (or certificate ID or the like).

### 身份验证(Authentication)
一般web应用的步骤都是:

1. 用传入的用户名和密码生成UsernamePasswordAuthenticationToken
2. 传入AuthenticationManager的实现类做校验
3. 如果验证通过返回一个完整的Authentication
4. 用SecurityContextHolder.getContext().setAuthentication()方法存入context

简化版的代码实现:

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;

public class AuthenticationExample {

    private static AuthenticationManager am = new SampleAuthenticationManager();

    //正确的用户名:admin,密码:1234
    public static void main(String[] args) throws Exception {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));

        while (true) {
            System.out.println("Please enter your username:");
            String name = in.readLine();
            System.out.println("Please enter your password:");
            String password = in.readLine();
            try {
                Authentication request = new UsernamePasswordAuthenticationToken(name, password);
                Authentication result = am.authenticate(request);
                SecurityContextHolder.getContext().setAuthentication(result);
                break;
            } catch (AuthenticationException e) {
                System.out.println("Authentication failed: " + e.getMessage());
            }
        }
        System.out.println("Successfully authenticated. Security context contains: "
                           + SecurityContextHolder.getContext().getAuthentication());
    }
}

class SampleAuthenticationManager implements AuthenticationManager {
    static final List<GrantedAuthority> AUTHORITIES = new ArrayList<GrantedAuthority>();

    static {
        AUTHORITIES.add(new SimpleGrantedAuthority("ROLE_USER"));
    }

    public Authentication authenticate(Authentication auth) throws AuthenticationException {
        if ("admin".equals(auth.getName()) && "1234".equals(auth.getCredentials())) {
            return new UsernamePasswordAuthenticationToken(auth.getName(), auth.getCredentials(), AUTHORITIES);
        }
        throw new BadCredentialsException("Bad Credentials");
    }
}
```

还可以直接向context中塞入Authentication

>In fact, Spring Security doesn’t mind how you put the Authentication object inside the SecurityContextHolder. The only critical requirement is that the SecurityContextHolder contains an Authentication which represents a principal before the *AbstractSecurityInterceptor* (which we’ll see more about later) needs to authorize a user operation.

### 访问控制(Authorization)
AccessDecisionManager来决定一个用户(Authentication)能否访问特定的资源(web应用中一般是url)

AbstractSecurityInterceptor的鉴权步骤:

1. Look up the "configuration attributes" associated with the present request
2. Submitting the secure object, current Authentication and configuration attributes to the AccessDecisionManager for an authorization decision
3. Optionally change the Authentication under which the invocation takes place
4. Allow the secure object invocation to proceed (assuming access was granted)
5. Call the AfterInvocationManager if configured, once the invocation has returned. If the invocation raised an exception, the AfterInvocationManager will not be invoked.

configuration attributes指的是配置文件中的这些配置

```xml
<intercept-url pattern="/imageBridge/**" access="permitAll" /> 
<intercept-url pattern="/system/getStartInfo.htm" access="permitAll" /> 
<intercept-url pattern="/**/*.htm" access="hasRole('ADMIN')" />
```

### 核心组件
**AuthenticationManager**顾名思义就是Authentication的管理器
在spring security中的默认实现是ProviderManager这个类的作用就是串联起一系列的**AuthenticationProvider**,每个Provider都会对用户进行身份验证.如果成功就返回Authentication,失败的话就抛出异常
AuthenticationProvider的定义:

```java
public interface AuthenticationProvider {
    Authentication authenticate(Authentication authentication)throws AuthenticationException;
    boolean supports(Class<?> authentication);
```

**UserDetailsService**用于根据username获取UserDetails对象.很多默认的Provider都需要注入UserDetailsService,当然如果是自己实现的Provider就可以不需要实现UserDetailsService

### 使用
从上面的介绍中可以看出,spring security最重要的两大特性就是身份验证和访问控制.而且可以很灵活的单独使用访问控制.下面我们来介绍几种常见的使用方法.
##### 1.自定义UserDetailsService

这个是简化版本的UserDetailsService的实现.当然现实情况中可能会从数据库中加载user信息.
这里使用的是默认的DaoAuthenticationProvider

```java
public class MyUserDetailsService implements UserDetailsService {

    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        List<GrantedAuthority> grantedAuthorities = new ArrayList<GrantedAuthority>();
        grantedAuthorities.add(new SimpleGrantedAuthority("ROLE_GUEST"));

        // 如果用户为admin,则分配两个权限
        if ("admin".equals(username)) {
            grantedAuthorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        }
        User user = new User(username, "password", true, true, true, true, grantedAuthorities);
        return user;
    }

}
```
spring security的配置

```xml
<http>
    <intercept-url pattern="/admin.do" access="hasRole('ADMIN')" />
    <intercept-url pattern="/**" access="hasAnyRole('GUEST','ADMIN')" />
    <form-login />
    <logout />
</http>

<authentication-manager>
    <authentication-provider user-service-ref="myUserDetailsService">
    </authentication-provider>
</authentication-manager>

<b:bean id="myUserDetailsService" class="name.chengchao.springsecurity.framework.MyUserDetailsService"></b:bean>
```

##### 2.自定义Provider

```java
public class MyAuthenticationProvider implements AuthenticationProvider {

    private static Logger logger = LoggerFactory.getLogger(MyAuthenticationProvider.class);

    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        if (!"password".equals(password)) {
            throw new BadCredentialsException("password error,username:" + username);
        }
        logger.info("username:{},login!", username);
        List<GrantedAuthority> grantedAuths = new ArrayList<GrantedAuthority>();
        grantedAuths.add(new SimpleGrantedAuthority("ROLE_GUEST"));
        if ("admin".equals(username)) {
            grantedAuths.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        }
        return new UsernamePasswordAuthenticationToken(username, password, grantedAuths);
    }

    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }

}
```
spring security的配置

```xml
<http>
    <intercept-url pattern="/admin.do" access="hasRole('ADMIN')" />
    <intercept-url pattern="/**" access="hasAnyRole('GUEST','ADMIN')" />
    <form-login />
    <logout />
</http>

<authentication-manager>
    <authentication-provider ref="myAuthenticationProvider">
    </authentication-provider>
</authentication-manager>

<b:bean id="myAuthenticationProvider" class="name.chengchao.springsecurity.framework.MyAuthenticationProvider"></b:bean>
```

这样就不需要再用到userdetailsService


##### 3.自定义Filter
记得前面说过,spring只关心在AbstractSecurityInterceptor之前,contextHolder中有Authentication信息就行了,至于怎么放进去的是不关心的.  
有很多时候,我们的系统需要和已有的统一登录(单点登录)系统对接.这就需要自定义一个filter来完成这个对接.

```java
public class MySecurityFilter extends GenericFilterBean {

    public Logger logger = LoggerFactory.getLogger(MySecurityFilter.class);

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException,
                                                                                      ServletException {

        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;

        // 这里的user一般来自于统一登录系统
        User user = new User("admin");
        // 再从数据库或者ACL系统查询该用户拥有的权限

        logger.info("welcome,{}", user.getName());

        List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
        authorities.add(new SimpleGrantedAuthority("ROLE_GUEST"));
        authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        Authentication authentication = new UsernamePasswordAuthenticationToken(user.getName(), user.getName(),
                                                                                authorities);
        SecurityContextHolder.getContext().setAuthentication(authentication);
        chain.doFilter(request, response);
    }

    class User {

        private String name;

        public User(String name){
            super();
            this.name = name;
        }

        public String getName() {
            return name;
        }

    }

}
```
spring security的配置

```xml
<http entry-point-ref="myAuthenticationEntryPoint">
    <intercept-url pattern="/admin.do" access="hasRole('ADMIN')" />
    <intercept-url pattern="/**" access="hasAnyRole('GUEST','ADMIN')" />
    <!-- <form-login /> -->
    <custom-filter ref="mySecurityFilter" before="FILTER_SECURITY_INTERCEPTOR" />
    <logout />
</http>

<!-- 自定义的filter -->
<b:bean id="mySecurityFilter"
    class="name.chengchao.springsecurity.framework.MySecurityFilter"></b:bean>

<!-- 定义一个空的entryPoint -->
<b:bean id="myAuthenticationEntryPoint"
    class="name.chengchao.springsecurity.framework.MyAuthenticationEntryPoint"></b:bean>

<authentication-manager></authentication-manager>

```

至此3中比较常见的用法都已经介绍完毕.在自定义的filter中会对每个请求都会做用户信息的查询.这里可以使用cache提高性能.在集群环境中,可以把用户信息放入用户浏览器的cookie中.用来保证不同机器之间的访问只需要登录一次.最后介绍一下权限的动态校验.因为以上举的例子都是静态化的配置.但在真实项目中对权限配置的需求往往是需要动态的(数据库).下面就来介绍一下如何在spring security中使用动态化的配置.  

在开始之前,先要了解一下org.springframework.security.access.AccessDecisionManager.这个接口用来访问控制.

```java
/**
     * Resolves an access control decision for the passed parameters.
     *
     * @param authentication the caller invoking the method (not null)
     * @param object the secured object being called
     * @param configAttributes the configuration attributes associated with the secured
     * object being invoked
     *
     * @throws AccessDeniedException if access is denied as the authentication does not
     * hold a required authority or ACL privilege
     * @throws InsufficientAuthenticationException if access is denied as the
     * authentication does not provide a sufficient level of trust
     */
    void decide(Authentication authentication, Object object,
            Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
            InsufficientAuthenticationException;
```

这个方法有三个参数:

- Authentication:是谁,以及有什么角色
- object:要访问的资源
- configAttributes:访问控制策略的配置,如果在数据库中,这个参数可以忽略

在来看看AbstractAccessDecisionManager有三个实现:

- AffirmativeBased: 只要有一个同意就行
- ConsensusBased: 少数服从多数
- UnanimousBased: 全体一致同意

可以根据自己的需要进行选择.下面的例子比较简单,用的是ConsensusBased

```java
public class MyVoter implements AccessDecisionVoter<FilterInvocation> {

    @Override
    public int vote(Authentication authentication, FilterInvocation fi, Collection<ConfigAttribute> attributes) {
        // FilterInvocation: URL: /admin.do 表示请求的资源
        // Authentication: 表示用户以及他拥有的权限
        // 根据数据库中的配置来决定
        // 可以参考org.springframework.security.web.access.expression.WebExpressionVoter
        return ACCESS_GRANTED;
    }

    @Override
    public boolean supports(ConfigAttribute attribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }

}
```


spring security的配置

```xml
<http entry-point-ref="myAuthenticationEntryPoint"
    access-decision-manager-ref="accessDecisionManager">
    <!-- 从数据库读取
    <intercept-url pattern="/admin.do" access="hasRole('ADMIN')" />
    <intercept-url pattern="/**" access="hasAnyRole('GUEST','ADMIN')" />
    -->
    <custom-filter ref="mySecurityFilter" before="FILTER_SECURITY_INTERCEPTOR" />
    <logout />
</http>

<!-- 自定义的filter -->
<b:bean id="mySecurityFilter"
    class="name.chengchao.springsecurity.framework.MySecurityFilter"></b:bean>

<!-- 定义一个空的entryPoint -->
<b:bean id="myAuthenticationEntryPoint"
    class="name.chengchao.springsecurity.framework.MyAuthenticationEntryPoint"></b:bean>

<authentication-manager></authentication-manager>


<b:bean id="myVoter" class="name.chengchao.springsecurity.framework.MyVoter">
</b:bean>

<b:bean id="accessDecisionManager"
    class="org.springframework.security.access.vote.ConsensusBased">
    <b:constructor-arg ref="myVoter"></b:constructor-arg>
</b:bean>
```

结束!