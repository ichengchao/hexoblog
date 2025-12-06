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



# 新建测试用户

在服务器管理器中选择"Active Directory用户和计算机"

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_12_03_1764751921.png" alt="Screenshot 2025-12-03 at 16.48.57" style="zoom:50%;" />

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_12_03_1764752816.png" alt="image-20251203170651934" style="zoom:50%;" />

# 配置User SSO

> 以阿里云RAM SSO为例, 在阿里云侧的配置就不做说明了. 这里只展示AD侧的配置

```
# meta xml,在AD机器上下载idp的meta xml,并上传到阿里云
https://charles.com/FederationMetadata/2007-06/FederationMetadata.xml
```

在服务器管理器中选择"AD FS管理", User SSO比较简单,只需要再SAML Response中增加一个NameID的Mapping就可以了,这里要注意的就是这个NameID的要和阿里云中的RAM User要对应,一个小技巧就是将ADFS的后缀统一替换成一个新的.比如我这里就替换成了chengchao.name, 第二在阿里云用户SSO的设置中将"辅助域名"设置成一样的. 这样只要邮箱前缀一样就可以

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_12_03_1764753638.png" alt="image-20251203172033872" style="zoom:50%;" />

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_12_03_1764754006.png" alt="Screenshot 2025-12-03 at 17.25.58" style="zoom:50%;" />

# 配置Role SSO

Role SSO的配置前提和User SSO是一致的,都需要先在AD和阿里云中设置好互相信任. 接着跟User SSO一样在ADFS管理中增加信任方.在"声明颁发策略"中增加3项

- NameID: 和User SSO一样
- RoleSessionName: 这个信息在登录成功后会在控制台显示
- Role: 确定IdP和role的信息

先做一个最简单的测试. 分别用写死的方式增加这个声明,就是不管是什么用户登录都会跳转到同一个role中, 设置两个"自定义规则发送声明"

rolesessionname

```
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
 => issue(Type = "https://www.aliyun.com/SAML-Role/Attributes/RoleSessionName", Value = "chengchao");
```

role

```
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
 => issue(Type = "https://www.aliyun.com/SAML-Role/Attributes/Role", Value = "acs:ram::1483522515186789:role/winserveradfs-testrole,acs:ram::1483522515186789:saml-provider/nxp-winserver-adfs");
```

**接着测试一个更加实际的例子**

1. 使用用户组来区分角色, 比如A,B两个都属于aliyun-admingroup用户组. 在阿里云上也新建一个admingroup的角色, 第一步增加一个"GetADGroup"的规则,这样就能把Group添加到声明中,这个是为了后面role从这个group中提取角色名称

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_12_04_1764815033.png" alt="image-20251204102349763" style="zoom:50%;" />



2. 修改一下rolesessionname的配置

   ```
   c:[Type == "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"]
    => issue(Type = "https://www.aliyun.com/SAML-Role/Attributes/RoleSessionName", Value = c.Value);
   ```

3. 修改一下role的配置, 这里会截取组的后缀变成角色名称-> aliyun-admingroup对应云上admingroup的角色,这个还可以做更加复杂的mapping,比如把aliyun-1483522515186789-admingroup里面的账号和角色都映射出来,用于多个账号的登录

   ```
   c:[Type == "http://schemas.xmlsoap.org/claims/Group", Value =~ "^aliyun-(.+)$"]
    => issue(Type = "https://www.aliyun.com/SAML-Role/Attributes/Role", Value = "acs:ram::1483522515186789:role/" + regexreplace(c.Value, "^aliyun-(.+)$", "$1") + ",acs:ram::1483522515186789:saml-provider/nxp-winserver-adfs");
   ```

   

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

登录ADFS,选择对应的SP进行SSO的跳转
```



# 进阶:用户同步

## 方案一

用第三方 IdP 做中转（推荐）

典型路径：AD → Microsoft Entra ID（原 Azure AD）→ 阿里云（CloudSSO / RAM）

在内网部署 Entra Connect（原 Azure AD Connect），它以代理方式从本地 AD 同步用户到 Entra ID，只需要出站 443，不需要对外开放 LDAP

## 方案二

用阿里云的 IDaaS作为桥接, 通过专线 / VPN / CEN 等私网链路让 IDaaS 访问你内网的 AD（还是 LDAP 协议，但只走专线/VPN，不暴露公网）, 适合在阿里云上部署AD或者已经和阿里云专线打通的客户

## 方案三

自己写一个脚本用于桥接AD (ldap)的读取和SCIM推送, 先测试第一步, 读取LDAP的数据

### test_ldap_read_users.py

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
步骤 1：只从本地 AD / LDAP 读取用户列表，并打印出来，方便排查连接和过滤问题。

依赖：
    pip install ldap3
"""

import logging
from typing import List, Dict

from ldap3 import Server, Connection, ALL, SUBTREE

# =========================
# 配置区：请按实际环境修改
# =========================

CONFIG = {
    # ---------- LDAP / AD 连接 ----------
    "LDAP_SERVER": "192.168.2.202",
    "LDAP_PORT": 389,              # LDAPS 建议用 636 + LDAP_USE_SSL=True
    "LDAP_USE_SSL": False,
    "LDAP_BIND_DN": "CN=chengadmin,CN=Users,DC=charles,DC=com",
    "LDAP_PASSWORD": "YourPassword",
    "LDAP_BASE_DN": "DC=charles,DC=com",

    # 过滤模式: "group" 或 "department"
    # 如果你想先不做过滤，可以设置 FILTER_MODE = "none"
    "FILTER_MODE": "department",

    # 按组过滤：只同步属于指定组的用户
    "GROUP_DN": "CN=aliyun-admingroup,CN=Users,DC=charles,DC=com",

    # 按部门过滤：只同步 department=DEPARTMENT_VALUE 的用户
    "DEPARTMENT_VALUE": "IT",
}

# =========================
# 日志配置
# =========================

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
)


# =========================
# LDAP 辅助函数
# =========================

def build_ldap_filter(config: Dict) -> str:
    """
    根据 FILTER_MODE 生成 LDAP 搜索过滤器。
    默认只筛选 user 对象，并排除 computer 对象。
    """
    # 只查 user，排除 computer
    base_user_filter = "(&(objectClass=user)(!(objectClass=computer)))"

    mode = config.get("FILTER_MODE", "group").lower()
    if mode == "group":
        group_dn = config["GROUP_DN"]
        extra_filter = f"(memberOf={group_dn})"
        logging.info(f"使用按组过滤: memberOf={group_dn}")
    elif mode == "department":
        dept = config["DEPARTMENT_VALUE"]
        extra_filter = f"(department={dept})"
        logging.info(f"使用按部门过滤: department={dept}")
    else:
        extra_filter = ""
        logging.warning("FILTER_MODE=none 或未知，将不做额外过滤。")

    if extra_filter:
        ldap_filter = f"(&{base_user_filter}{extra_filter})"
    else:
        ldap_filter = base_user_filter

    logging.info(f"LDAP search filter = {ldap_filter}")
    return ldap_filter


def get_ldap_connection(config: Dict) -> Connection:
    server = Server(
        config["LDAP_SERVER"],
        port=config["LDAP_PORT"],
        use_ssl=config["LDAP_USE_SSL"],
        get_info=ALL,
    )
    conn = Connection(
        server,
        user=config["LDAP_BIND_DN"],
        password=config["LDAP_PASSWORD"],
        auto_bind=True,
    )
    logging.info("已成功绑定到 LDAP / AD。")
    return conn


def fetch_ad_users(config: Dict) -> List[Dict]:
    """
    从 AD 中抓取用户，并转换为 Python dict。
    """
    conn = get_ldap_connection(config)
    ldap_filter = build_ldap_filter(config)

    # 先选一些常见属性看一下
    attributes = [
        "sAMAccountName",
        "userPrincipalName",
        "givenName",
        "sn",
        "displayName",
        "mail",
        "department",
        "userAccountControl",
    ]

    logging.info("开始从 AD 查询用户...")
    conn.search(
        search_base=config["LDAP_BASE_DN"],
        search_filter=ldap_filter,
        search_scope=SUBTREE,
        attributes=attributes,
    )

    users = []
    for entry in conn.entries:
        e = entry.entry_attributes_as_dict

        def get(attr, default=None):
            v = e.get(attr)
            if isinstance(v, list):
                return v[0] if v else default
            return v if v is not None else default

        user = {
            "dn": entry.entry_dn,
            "sAMAccountName": get("sAMAccountName"),
            "userPrincipalName": get("userPrincipalName"),
            "displayName": get("displayName"),
            "givenName": get("givenName"),
            "sn": get("sn"),
            "mail": get("mail"),
            "department": get("department"),
            "userAccountControl": get("userAccountControl"),
        }
        users.append(user)

    logging.info(f"从 AD 共获取到 {len(users)} 个用户。")
    conn.unbind()
    return users


# =========================
# 主流程
# =========================

def main():
    logging.info("==== 仅 LDAP 读取用户的测试开始 ====")
    users = fetch_ad_users(CONFIG)

    # 为了方便排查，简单打印出来
    for idx, u in enumerate(users, start=1):
        logging.info("----- 用户 %d -----", idx)
        logging.info("DN               : %s", u["dn"])
        logging.info("sAMAccountName   : %s", u["sAMAccountName"])
        logging.info("userPrincipalName: %s", u["userPrincipalName"])
        logging.info("displayName      : %s", u["displayName"])
        logging.info("givenName        : %s", u["givenName"])
        logging.info("sn               : %s", u["sn"])
        logging.info("mail             : %s", u["mail"])
        logging.info("department       : %s", u["department"])
        logging.info("userAccountControl: %s", u["userAccountControl"])

    logging.info("==== LDAP 读取用户的测试结束 ====")


if __name__ == "__main__":
    main()
```

#### ad_to_cloudsso_scim_full.py : 支持同步用户和用户组

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
从本地 AD 读取用户，并通过 SCIM 推送到阿里云 CloudSSO 的完整示例脚本。
功能：
- 按组/部门过滤，从 AD 中读取用户
- 同步用户到 CloudSSO SCIM /Users
- 从用户的 memberOf 中解析出组，并同步到 /Groups
- 使用 PATCH /Groups/{id} 同步组成员关系（首轮就生效）

前置：
    pip install ldap3 requests
"""

import uuid
import json
import logging
from typing import List, Dict, Optional, Set

import requests
from ldap3 import Server, Connection, ALL, SUBTREE, BASE

# =========================
# 配置区：请按实际环境修改
# =========================

CONFIG = {
    # ---------- LDAP / AD 连接 ----------
    "LDAP_SERVER": "192.168.2.202",
    "LDAP_PORT": 389,              # LDAPS 建议用 636 + LDAP_USE_SSL=True
    "LDAP_USE_SSL": False,
    "LDAP_BIND_DN": "CN=chengadmin,OU=cloudplatform,DC=charles,DC=com",
    "LDAP_PASSWORD": "YourPassword",
    "LDAP_BASE_DN": "OU=cloudplatform,DC=charles,DC=com",

    # 过滤模式: "group" 或 "department" 或 "none"
    "FILTER_MODE": "none",

    # 按组过滤：只同步属于指定组的用户
    "GROUP_DN": "CN=aliyun-admingroup,OU=cloudplatform,DC=charles,DC=com",

    # 按部门过滤：只同步 department=DEPARTMENT_VALUE 的用户
    "DEPARTMENT_VALUE": "IT",

    # ---------- SCIM / CloudSSO ----------
    # CloudSSO 的 SCIM Endpoint (注意 region 和目录)
    # 参考文档: https://cloudsso-scim-<regionId>.aliyun.com/scim/v2/
    "SCIM_BASE_URL": "https://cloudsso-scim-cn-shanghai.aliyun.com/scim/v2",

    # 在 CloudSSO 控制台中「目录 -> SCIM 同步 -> 管理 SCIM 密钥」生成的密钥
    "SCIM_BEARER_TOKEN": "SCIM_TOKEN",

    # 把 AD 属性映射到 CloudSSO SCIM user 的 userName
    # 可选值示例: "userPrincipalName" 或 "sAMAccountName"
    "USERNAME_FROM": "userPrincipalName",

    # dry run 模式：True = 只打印要同步的内容，不真正调用 SCIM
    "DRY_RUN": False,
}


# =========================
# 日志配置
# =========================

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
)


# =========================
# LDAP 辅助函数
# =========================

def build_ldap_filter(config: Dict) -> str:
    """
    根据 FILTER_MODE 生成 LDAP 搜索过滤器。
    默认只筛选 user 对象，并排除 computer 对象。
    """
    base_user_filter = "(&(objectClass=user)(!(objectClass=computer)))"

    mode = config.get("FILTER_MODE", "group").lower()
    if mode == "group":
        group_dn = config["GROUP_DN"]
        extra_filter = f"(memberOf={group_dn})"
        logging.info(f"使用按组过滤: memberOf={group_dn}")
    elif mode == "department":
        dept = config["DEPARTMENT_VALUE"]
        extra_filter = f"(department={dept})"
        logging.info(f"使用按部门过滤: department={dept}")
    else:
        extra_filter = ""
        logging.warning("FILTER_MODE=none 或未知，将不做额外过滤。")

    if extra_filter:
        ldap_filter = f"(&{base_user_filter}{extra_filter})"
    else:
        ldap_filter = base_user_filter

    logging.info(f"LDAP search filter = {ldap_filter}")
    return ldap_filter


def get_ldap_connection(config: Dict) -> Connection:
    server = Server(
        config["LDAP_SERVER"],
        port=config["LDAP_PORT"],
        use_ssl=config["LDAP_USE_SSL"],
        get_info=ALL,
    )
    conn = Connection(
        server,
        user=config["LDAP_BIND_DN"],
        password=config["LDAP_PASSWORD"],
        auto_bind=True,
    )
    logging.info("已成功绑定到 LDAP / AD。")
    return conn


def fetch_ad_users(config: Dict) -> List[Dict]:
    """
    从 AD 中抓取用户，并转换为 Python dict。
    这里会额外取出 memberOf，后面做组同步用。
    """
    conn = get_ldap_connection(config)
    ldap_filter = build_ldap_filter(config)

    attributes = [
        "sAMAccountName",
        "userPrincipalName",
        "givenName",
        "sn",
        "displayName",
        "mail",
        "userAccountControl",
        "objectGUID",
        "department",
        "memberOf",
    ]

    logging.info("开始从 AD 查询用户...")
    conn.search(
        search_base=config["LDAP_BASE_DN"],
        search_filter=ldap_filter,
        search_scope=SUBTREE,
        attributes=attributes,
    )

    users = []
    for entry in conn.entries:
        e = entry.entry_attributes_as_dict

        def get(attr, default=None):
            v = e.get(attr)
            if isinstance(v, list):
                return v[0] if v else default
            return v if v is not None else default

        # objectGUID 二进制转为 UUID 字符串，作为 externalId
        raw_guid = get("objectGUID")
        if raw_guid:
            try:
                external_id = str(uuid.UUID(bytes_le=raw_guid))
            except Exception:
                external_id = None
        else:
            external_id = None

        # userName 策略
        if CONFIG["USERNAME_FROM"] == "userPrincipalName":
            user_name = get("userPrincipalName") or get("sAMAccountName")
        else:
            user_name = get("sAMAccountName") or get("userPrincipalName")

        given_name = get("givenName", "") or ""
        family_name = get("sn", "") or ""

        display_name = get("displayName")
        if not display_name:
            display_name = f"{given_name} {family_name}".strip() or user_name

        # 邮箱
        email = get("mail")

        # active 状态，基于 userAccountControl（简单处理）
        uac = get("userAccountControl", 0)
        try:
            uac = int(uac)
        except Exception:
            uac = 0
        # AD：0x2 表示禁用
        active = not bool(uac & 0x2)

        # memberOf 可能是 list / str / 不存在
        member_of_raw = e.get("memberOf", [])
        if isinstance(member_of_raw, str):
            member_dns = [member_of_raw]
        elif isinstance(member_of_raw, list):
            member_dns = member_of_raw
        else:
            member_dns = []

        user_dict = {
            "userName": user_name,
            "displayName": display_name,
            "givenName": given_name,
            "familyName": family_name,
            "email": email,
            "externalId": external_id or user_name,  # 兜底
            "active": active,
            "department": get("department"),
            "raw_dn": entry.entry_dn,
            "memberOf": member_dns,
        }

        # 过滤掉没有 userName 的
        if not user_dict["userName"]:
            logging.warning(f"跳过无 userName 的条目: DN={entry.entry_dn}")
            continue

        users.append(user_dict)

    logging.info(f"从 AD 共获取到 {len(users)} 个用户。")
    conn.unbind()
    return users


def fetch_ad_groups_from_users(config: Dict, ad_users: List[Dict]) -> List[Dict]:
    """
    从用户的 memberOf 收集所有相关组的 DN，并去 AD 读取组信息：
    - externalId 使用 group.objectGUID（若有），否则用 DN
    - displayName 使用 sAMAccountName / name / cn
    - members_external_ids: 组内所有成员的 externalId 集合
    """
    # 收集所有组 DN
    group_dn_map: Dict[str, Dict] = {}
    for user in ad_users:
        ext_id = user["externalId"]
        for gdn in user.get("memberOf", []) or []:
            if gdn not in group_dn_map:
                group_dn_map[gdn] = {
                    "dn": gdn,
                    "displayName": None,
                    "externalId": None,
                    "members_external_ids": set(),  # type: Set[str]
                }
            group_dn_map[gdn]["members_external_ids"].add(ext_id)

    if not group_dn_map:
        logging.info("用户的 memberOf 中没有发现任何组，无需同步组。")
        return []

    conn = get_ldap_connection(config)

    for gdn, ginfo in group_dn_map.items():
        logging.info(f"查询组详情: {gdn}")
        conn.search(
            search_base=gdn,
            search_filter="(objectClass=group)",
            search_scope=BASE,
            attributes=["objectGUID", "sAMAccountName", "name", "cn"],
        )
        if not conn.entries:
            logging.warning(
                f"未在 AD 中找到组 {gdn} 的详细信息，将使用 DN 作为 displayName/externalId"
            )
            ginfo["displayName"] = ginfo["displayName"] or gdn
            ginfo["externalId"] = ginfo["externalId"] or gdn
            continue

        ge = conn.entries[0].entry_attributes_as_dict

        def get(attr):
            v = ge.get(attr)
            if isinstance(v, list):
                return v[0] if v else None
            return v

        raw_guid = get("objectGUID")
        if raw_guid:
            try:
                external_id = str(uuid.UUID(bytes_le=raw_guid))
            except Exception:
                external_id = None
        else:
            external_id = None

        name = get("sAMAccountName") or get("name") or get("cn") or gdn

        ginfo["displayName"] = name
        ginfo["externalId"] = external_id or gdn

    conn.unbind()

    groups = list(group_dn_map.values())
    logging.info(f"共解析出 {len(groups)} 个组需要同步。")
    for g in groups:
        logging.info(
            f"组: dn={g['dn']}, displayName={g['displayName']}, "
            f"externalId={g['externalId']}, members={len(g['members_external_ids'])}"
        )
    return groups


# =========================
# SCIM / CloudSSO 辅助函数
# =========================

def scim_headers(config: Dict) -> Dict[str, str]:
    return {
        "Authorization": f"Bearer {config['SCIM_BEARER_TOKEN']}",
        "Content-Type": "application/json",
    }


def scim_request(
    config: Dict, method: str, path: str, **kwargs
) -> requests.Response:
    """
    统一封装 SCIM 请求。
    path 例如: "/Users", "/Users/<id>", "/Groups", "/Groups/<id>"
    """
    url = config["SCIM_BASE_URL"].rstrip("/") + path
    headers = scim_headers(config)
    headers.update(kwargs.pop("headers", {}))

    logging.debug(f"SCIM {method} {url}")
    resp = requests.request(method, url, headers=headers, timeout=30, **kwargs)
    if not resp.ok:
        logging.error(
            f"SCIM 请求失败: {method} {url} "
            f"status={resp.status_code}, body={resp.text}"
        )
    return resp


def find_scim_user_by_external_id(
    config: Dict, external_id: str
) -> Optional[Dict]:
    """
    通过 externalId 在 CloudSSO SCIM /Users 中查找用户。
    """
    params = {
        "filter": f'externalId eq "{external_id}"',
        "count": 1,
    }
    resp = scim_request(config, "GET", "/Users", params=params)
    if not resp.ok:
        return None

    data = resp.json()
    resources = data.get("Resources", [])
    if resources:
        return resources[0]
    return None


def find_scim_group_by_external_id(
    config: Dict, external_id: str
) -> Optional[Dict]:
    """
    通过 externalId 在 CloudSSO SCIM /Groups 中查找组。
    """
    params = {
        "filter": f'externalId eq "{external_id}"',
        "count": 1,
    }
    resp = scim_request(config, "GET", "/Groups", params=params)
    if not resp.ok:
        return None

    data = resp.json()
    resources = data.get("Resources", [])
    if resources:
        return resources[0]
    return None


def build_scim_user_payload(ad_user: Dict) -> Dict:
    """
    将 AD 用户映射为 CloudSSO SCIM /Users 的 JSON。
    """
    payload = {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "userName": ad_user["userName"],
        "externalId": ad_user["externalId"],
        "displayName": ad_user["displayName"],
        "name": {
            "givenName": ad_user["givenName"],
            "familyName": ad_user["familyName"],
        },
        "active": ad_user["active"],
    }

    emails = []
    if ad_user.get("email"):
        emails.append(
            {
                "value": ad_user["email"],
                "type": "work",
                "primary": True,
            }
        )
    if emails:
        payload["emails"] = emails

    return payload


def patch_group_members(
    config: Dict,
    group_id: str,
    group: Dict,
    user_extid_to_scimid: Dict[str, str],
    user_extid_to_user: Dict[str, Dict],
) -> None:
    """
    使用 PATCH /Groups/{id} 同步组成员关系：
    - 先 remove 所有 members
    - 再 add 当前应该存在的所有 members

    members 元素格式：
    {
        "value": "<userId>",
        "$ref": "<SCIM_BASE_URL>/Users/<userId>",
        "display": "<userName or displayName>"
    }
    """

    if CONFIG["DRY_RUN"]:
        logging.info(
            f"DRY_RUN 模式：不实际 PATCH 组成员: {group['displayName']}"
        )
        return

    base_url = config["SCIM_BASE_URL"].rstrip("/")
    members = []

    for user_external_id in group.get("members_external_ids", []) or []:
        scim_id = user_extid_to_scimid.get(user_external_id)
        if not scim_id:
            logging.warning(
                f"组 {group['displayName']} 的成员 externalId={user_external_id} "
                f"尚未找到对应的 SCIM User id，跳过该成员。"
            )
            continue

        user = user_extid_to_user.get(user_external_id, {})
        display = user.get("userName") or user.get("displayName") or ""

        members.append(
            {
                "value": scim_id,
                "$ref": f"{base_url}/Users/{scim_id}",
                "display": display,
            }
        )

    if not members:
        logging.info(
            f"组 {group['displayName']} 最终没有任何成员需要同步，将清空成员。"
        )

    # 先 remove 全部 members，再 add 当前 members
    patch_body = {
        "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
        "Operations": []
    }

    # remove 所有成员
    patch_body["Operations"].append(
        {
            "op": "remove",
            "path": "members",
        }
    )

    # 再添加现有成员
    if members:
        patch_body["Operations"].append(
            {
                "op": "add",
                "path": "members",
                "value": members,
            }
        )

    logging.info(
        f"通过 PATCH 同步组成员: groupId={group_id}, 成员数={len(members)}"
    )
    logging.debug(
        "SCIM Group PATCH payload: %s",
        json.dumps(patch_body, ensure_ascii=False),
    )

    resp = scim_request(
        config,
        "PATCH",
        f"/Groups/{group_id}",
        data=json.dumps(patch_body),
    )

    if resp.ok:
        logging.info("组成员 PATCH 成功。")
    else:
        logging.error("组成员 PATCH 失败，请检查返回信息。")


def sync_user_to_scim(config: Dict, ad_user: Dict) -> Optional[str]:
    """
    将单个 AD 用户同步到 CloudSSO：
    - 若 externalId 已存在 => PUT 更新
    - 若不存在 => POST 创建
    返回该用户在 SCIM 中的 id（用于组成员关系）
    """
    scim_payload = build_scim_user_payload(ad_user)

    logging.info(
        f"准备同步用户: userName={ad_user['userName']}, "
        f"externalId={ad_user['externalId']}, "
        f"active={ad_user['active']}, "
        f"DN={ad_user['raw_dn']}"
    )
    logging.debug("SCIM User payload: %s", json.dumps(scim_payload, ensure_ascii=False))

    if CONFIG["DRY_RUN"]:
        logging.info("DRY_RUN 模式：只打印用户，不调用 SCIM。")
        return None

    existing = find_scim_user_by_external_id(config, ad_user["externalId"])

    if existing:
        scim_id = existing["id"]
        logging.info(
            f"CloudSSO 中已存在 externalId={ad_user['externalId']} 的用户，"
            f"执行 PUT 更新，id={scim_id}"
        )
        resp = scim_request(
            config,
            "PUT",
            f"/Users/{scim_id}",
            data=json.dumps(scim_payload),
        )
        if resp.ok:
            logging.info("SCIM 用户更新成功。")
            return scim_id
        else:
            logging.error("SCIM 用户更新失败，请检查返回信息。")
            return None
    else:
        logging.info(
            f"CloudSSO 中不存在 externalId={ad_user['externalId']} 的用户，执行 POST 创建。"
        )
        resp = scim_request(
            config,
            "POST",
            "/Users",
            data=json.dumps(scim_payload),
        )
        if resp.ok:
            data = resp.json()
            scim_id = data.get("id")
            logging.info(f"SCIM 用户创建成功，id={scim_id}")
            return scim_id
        else:
            logging.error("SCIM 用户创建失败，请检查返回信息。")
            return None


def sync_group_to_scim(
    config: Dict,
    group: Dict,
    user_extid_to_scimid: Dict[str, str],
    user_extid_to_user: Dict[str, Dict],
) -> Optional[str]:
    """
    将单个 AD 组同步到 CloudSSO：
    - 若 externalId 已存在 => 先确保组存在（PUT 基础信息），再 PATCH 成员
    - 若不存在 => POST 创建组，再 PATCH 成员
    """
    logging.info(
        f"准备同步组: displayName={group['displayName']}, "
        f"externalId={group['externalId']}, "
        f"DN={group['dn']}, "
        f"成员数={len(group.get('members_external_ids', []))}"
    )

    if CONFIG["DRY_RUN"]:
        logging.info("DRY_RUN 模式：只打印组，不调用 SCIM。")
        return None

    existing = find_scim_group_by_external_id(config, group["externalId"])

    scim_group_id: Optional[str] = None

    # 基础组信息（不带 members）
    base_payload = {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "displayName": group["displayName"],
        "externalId": group["externalId"],
    }

    if existing:
        scim_group_id = existing["id"]
        logging.info(
            f"CloudSSO 中已存在 externalId={group['externalId']} 的组，"
            f"执行 PUT 更新基础信息，id={scim_group_id}"
        )
        resp = scim_request(
            config,
            "PUT",
            f"/Groups/{scim_group_id}",
            data=json.dumps(base_payload),
        )
        if resp.ok:
            logging.info("SCIM 组基础信息更新成功。")
        else:
            logging.error("SCIM 组基础信息更新失败，请检查返回信息。")
    else:
        logging.info(
            f"CloudSSO 中不存在 externalId={group['externalId']} 的组，执行 POST 创建。"
        )
        resp = scim_request(
            config,
            "POST",
            "/Groups",
            data=json.dumps(base_payload),
        )
        if resp.ok:
            data = resp.json()
            scim_group_id = data.get("id")
            logging.info(f"SCIM 组创建成功，id={scim_group_id}")
        else:
            logging.error("SCIM 组创建失败，请检查返回信息。")
            return None

    if scim_group_id:
        # 无论是新建还是已存在，都用 PATCH 把成员同步进去（首轮就有关系）
        patch_group_members(
            config,
            scim_group_id,
            group,
            user_extid_to_scimid,
            user_extid_to_user,
        )

    return scim_group_id


# =========================
# 主流程
# =========================

def main():
    logging.info("==== AD -> CloudSSO (SCIM) 同步开始 ====")
    ad_users = fetch_ad_users(CONFIG)

    # 先同步用户，记录 externalId -> SCIM id 的映射，用于组成员关系
    user_extid_to_scimid: Dict[str, str] = {}
    user_extid_to_user: Dict[str, Dict] = {}

    for user in ad_users:
        user_extid_to_user[user["externalId"]] = user
        try:
            scim_id = sync_user_to_scim(CONFIG, user)
            if scim_id:
                user_extid_to_scimid[user["externalId"]] = scim_id
        except Exception as e:
            logging.exception(
                f"同步用户 {user.get('userName')} 时发生异常: {e}"
            )

    # 再根据这些用户的 memberOf，同步相关组信息
    ad_groups = fetch_ad_groups_from_users(CONFIG, ad_users)

    for group in ad_groups:
        try:
            sync_group_to_scim(
                CONFIG,
                group,
                user_extid_to_scimid,
                user_extid_to_user,
            )
        except Exception as e:
            logging.exception(
                f"同步组 {group.get('displayName')} 时发生异常: {e}"
            )

    logging.info("==== AD -> CloudSSO (SCIM) 同步结束 ====")


if __name__ == "__main__":
    main()
```


# 附录

**将证书导入到“受信任的根证书颁发机构", 这样访问adfs的时候就不会有警告了  (非必选)**

- 打开“证书管理器”（运行 certlm.msc）
- 导航到“受信任的根证书颁发机构” > “证书”
- 右键单击“证书”文件夹，选择“所有任务” > “导入”
- 选择 adfs.pfx 文件，并完成导入向导
- 重启一下服务器

**查看LDAP属性**

```powershell
# 查看用户信息
Get-ADUser -Identity "charles" -Properties * | Format-List

# 查看用户组的信息
Get-ADGroup -Identity "aliyun-admingroup" -Properties * | Format-List
```

**ADFS 转换语法**

> 假设需要将aliyun-role-devops这个命名风格的用户组转换成阿里云上role-devops的角色

```
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid", Value =~ "^aliyun-(.*)"]
=> issue(Type = "https://www.aliyun.com/SAML-Role/Attributes/Role", 
         Value = "acs:ram::17642631404*****:role/" + regexreplace(c.Value, "^aliyun-", "") + ",acs:ram::17642631404*****:saml-provider/chengchaoIDP");
```

这个条件会检查组声明的类型是否为 groupsid，并使用正则表达式 ^aliyun-(.*) 匹配以 aliyun- 开头的组名，捕获组名的后缀部分（如 role-devops）,转换成阿里云SAML中role的格式"role-arn,idp-arn"



# 参考资料

- [[阿里云] 在Windows实例上搭建AD域并将客户端加入AD域](https://help.aliyun.com/zh/ecs/use-cases/ecs-instance-building-windows-active-directory-domain)
- [[阿里云] 使用ADFS 进行角色SSO的示例](https://help.aliyun.com/zh/ram/user-guide/implement-role-based-sso-from-ad-fs)
- [[腾讯云] ADFS 单点登录腾讯云指南](https://cloud.tencent.com/document/product/598/42702)
