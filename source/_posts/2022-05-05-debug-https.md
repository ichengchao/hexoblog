---
title: https调试工具
date: 2022-05-05 19:03:48
tags:

 - tech
 - http

---

![image-20220505173714061](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2022_05_05_1651743434.png)



# 背景

现在绝大多数网站都已经是https,更安全的同时,调试起来也变复杂了.在这里推荐两个工具

- [Charles](https://www.charlesproxy.com/)
- [Surge](https://nssurge.com/)



# Surge

这里就介绍一下Surge的配置方式

### 1. 安装系统证书

![Screen Shot 2022-05-05 at 17.40.15](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2022_05_05_1651743701.png)

这样就能在Keychain Access中看到安装的证书

![image-20220505174320585](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2022_05_05_1651743800.png)

### 2. 安装证书到JVM中

由于我需要调试Java程序中的https请求,所以需要安装到JVM中

先导出证书文件

![Screen Shot 2022-05-05 at 17.47.52](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2022_05_05_1651744124.png)

命令行安装到JVM中,默认密码是`changeit`

```bash
sudo keytool -import -alias surge-20220505 -file ~/Desktop/test.crt -keystore /Library/Java/JavaVirtualMachines/jdk-11.0.12.jdk/Contents/Home/lib/security/cacerts -storepass changeit
```

推荐使用[KeyStore Explorer](https://keystore-explorer.org/),查看是否正确导入,如果用管理员权限打开的话,还可以删除证书

```bash
sudo  /Applications/KeyStore\ Explorer.app/Contents/MacOS/KeyStore\ Explorer
```



![Screen Shot 2022-05-05 at 17.59.43](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2022_05_05_1651744848.png)

# 测试

打开surge的监测功能就能看到https的请求了,done!

![Screen Shot 2022-05-05 at 18.13.17](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2022_05_05_1651745638.png)

