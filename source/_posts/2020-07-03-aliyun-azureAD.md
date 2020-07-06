---
title: 阿里云体验之集成Azure Active Directory
date: 2020-07-03 10:50:43
tags:

 - tech
 - aliyun
 - azure
 - sso
 - ram
---



<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_07_06_10_17_1594001820388.png" alt="image-20200706101659716" style="zoom:50%;" />

### 背景

账号管理是企业上云过程中很重要的一环,而Azure Active Directory是其中的佼佼者.今天就来介绍一下如何用Azure Active Directory来SSO到阿里云

### Azure配置

#### 新增企业应用程序

1. 进入[AD的主页](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview), 并点击左侧菜单中的`企业应用程序`
2. 新建应用,不需要从模板中新建(跟阿里云官网的文档不太一样)

3. 进入应用后,点击左侧菜单的`单一登录`,并选择SAML

4. 下载阿里云提供的[meta模板](https://signin.aliyun.com/saml-role/sp-metadata.xml),上传到这里,如图

   <img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_07_06_12_43_1594010606821.png" alt="Screen Shot 2020-07-06 at 12.42.40" style="zoom:50%;" />

5. 编辑`用户属性和声明`

   ```
   # https://www.aliyun.com/SAML-Role/Attributes/Role的填写格式
   acs:ram::{账号ID}:role/{角色名称},acs:ram::{账号ID}:saml-provider/{身份提供商名称}
   示例:
   账号ID: 1764263188888888
   角色名称: AzureAdmin
   Idp名称: Azure
   
   acs:ram::1764263188888888:role/AzureAdmin,acs:ram::1764263188888888:saml-provider/Azure
   
   # https://www.aliyun.com/SAML-Role/Attributes/RoleSessionName的填写格式
   这里可以随便取个名字
   
   # 如下图
   
   ```

   

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_07_06_11_37_1594006662769.png" alt="image-20200706113742603" style="zoom:50%;" />

6. 登录测试



#### 删除企业应用程序

> 跟本流程无关

顺便介绍一下怎么删除应用,进入你要删除的应用程序

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_07_06_10_47_1594003641865.png" alt="Screen Shot 2020-07-06 at 10.46.35" style="zoom:50%;" />

如果这里的删除按钮是灰色,则可以使用PowerShell来删除

```powershell
# 连接AD
Connect-AzureAD

# 获取应用列表
Get-AzureADServicePrincipal

# 根据objectID删除应用
Remove-AzureADServicePrincipal -objectid [ObjectID]
```



### Aliyun配置

1. 进入RAM的[SSO设置页面](https://ram.console.aliyun.com/providers),新建身份提供商,也就是Azure Active Directory

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_07_06_11_04_1594004686583.png" alt="Screen Shot 2020-07-06 at 11.03.58" style="zoom:50%;" />

2. 新建RAM角色并授权,AzureAD SSO过来的用户就是用这个角色登录到Aliyun的

   <img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2020_07_06_11_07_1594004872761.png" alt="Screen Shot 2020-07-06 at 11.06.06" style="zoom:50%;" />

