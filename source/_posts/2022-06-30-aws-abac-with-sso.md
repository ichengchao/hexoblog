---
title: AWS ABAC with SSO
date: 2022-06-30 19:03:48
tags:

 - tech
 - cloud
 - aws

---





# 背景知识

上一篇讲了[AWS ABAC的入门](https://blog.chengchao.name/2022/05/27/aws-abac/) , 这篇看看如何把ABAC的能力和SSO配合起来,完成企业用户单点登录到云上后能根据资源标签来授权



# 创建角色

策略方面可以沿用上一篇的`access-same-product-team`. 

创建一个新的角色: `abacRole`,并把`access-same-product-team`授权给这个角色.并修改信任策略. 将TagSession加到Action中

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::433312851566:saml-provider/springrun"
            },
            "Action": [
                "sts:AssumeRoleWithSAML",
                "sts:TagSession"
            ],
            "Condition": {
                "StringEquals": {
                    "SAML:aud": "https://signin.aws.amazon.com/saml"
                }
            }
        }
    ]
}
```





# SAML协议

在saml协议中增加对应的tag

```xml
<saml2:AttributeStatement>
	<saml2:Attribute Name="https://aws.amazon.com/SAML/Attributes/PrincipalTag:team">
		<saml2:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="xsd:string">dev1</saml2:AttributeValue>
  </saml2:Attribute>
  <saml2:Attribute Name="https://aws.amazon.com/SAML/Attributes/PrincipalTag:product">
    <saml2:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="xsd:string">app1</saml2:AttributeValue>
  </saml2:Attribute>
</saml2:AttributeStatement>
```





# 测试

单点登录到`abacRole`这个role的时候,有Secrets Manager中`dev/app1/key`的权限,但是没有`dev/app2/key1`的权限
