---
title: ADFS SSO配置
date: 2024-11-17 11:00:00
tags:
 - tech
 - cloud
 - adfs
 - sho
---



# 前言

网上已经有这么多的配置教程,为什么还要再写一个, 答案非常简单: "都太复杂了, 我希望能以我认为最简单的方式来体验一下ADFS的配置过程"

# 安装AD域服务

> 假设域为: charles.com

配置AD的过程非常简单,只要一步步按照默认的就行了.

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_11_17_1731812794.png" alt="image-20241117110630411" style="zoom:50%;" />

安装完域服务后,将本服务器提升为域控制器,安装完成后重启服务器,这样就能用域控的方式登录了

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_11_17_1731812886.png" alt="image-20241117110804331" style="zoom:50%;" />

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_11_17_1731812926.png" alt="image-20241117110843714" style="zoom:50%;" />

重启完成后,AD域服务就完成

# 生成自签名的SSL证书

这步是网上最复杂的一步, 所以使用脚本要简单很多, 按照自己的域名修改一下脚本. 保存成cert.ps1文件,并使用powershell运行,正常的话就能在桌面生成adfs.pfx证书文件. 私钥密码为"YourPassword"(可以自己在脚本中修改)

```powershell
# 设置 FQDN 和多个备用域名（SAN）
$dnsNames = @("charles.com", "*.charles.com", "*.adfs.charles.com")  # 将所有域名放入数组中
$certPath = "$env:USERPROFILE\Desktop\adfs.pfx"            # 证书保存路径
$password = ConvertTo-SecureString -String "YourPassword" -Force -AsPlainText  # 设置证书导出密码

# 生成自签名证书，包含多个 SAN
$cert = New-SelfSignedCertificate `
    -DnsName $dnsNames `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -NotAfter (Get-Date).AddYears(1) `
    -KeyAlgorithm RSA `
    -KeyLength 2048

# 将证书导出为 .pfx 文件
Export-PfxCertificate -Cert "Cert:\LocalMachine\My\$($cert.Thumbprint)" `
    -FilePath $certPath `
    -Password $password

Write-Output "Certificate saved to $certPath"
```

# 安装ADFS

这里就比较简单了,在第一步中导入上一步中生成的adfs.pfx证书,其他的就是按照默认配置继续就行了.安装完成后重启服务器

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_11_17_1731813377.png" alt="image-20241117111614654" style="zoom:50%;" />

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_11_17_1731813455.png" alt="image-20241117111734339" style="zoom:50%;" />

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_11_17_1731813487.png" alt="image-20241117111806316" style="zoom:50%;" />

# 开启单点登录并测试

```powershell
# 开启单点登录
Set-AdfsProperties -EnableIdpInitiatedSignonPage $true
# 重启adfs
Restart-Service -Name adfssrv
```

```
# adfs登录地址
https://charles.com/adfs/ls/idpinitiatedSignOn.htm
# meta xml
https://charles.com/FederationMetadata/2007-06/FederationMetadata.xml

# 登录用户
可以在AD用户中新建,或者直接使用administrator,比如administrator@charles.com
```

# 附录

**将证书导入到“受信任的根证书颁发机构", 这样访问adfs的时候就不会有警告了  (非必选)**

- 打开“证书管理器”（运行 certlm.msc）
- 导航到“受信任的根证书颁发机构” > “证书”
- 右键单击“证书”文件夹，选择“所有任务” > “导入”
- 选择 adfs.pfx 文件，并完成导入向导
- 重启一下服务器

**查看某个用户的LDAP所有属性**

```powershell
Get-ADUser -Identity "charles" -Properties * | Format-List
```

**参考资料**

- [[阿里云] 在Windows实例上搭建AD域并将客户端加入AD域](https://help.aliyun.com/zh/ecs/use-cases/ecs-instance-building-windows-active-directory-domain)
- [[阿里云] 使用ADFS 进行角色SSO的示例](https://help.aliyun.com/zh/ram/user-guide/implement-role-based-sso-from-ad-fs)
- [[腾讯云] ADFS 单点登录腾讯云指南](https://cloud.tencent.com/document/product/598/42702)
