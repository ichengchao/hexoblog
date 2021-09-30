---
title: 阿里云实例角色
date: 2021-09-28 19:03:48
tags:

 - tech
 - aliyun

---

# 背景

ECS的「实例角色」是在整个无AK设计中很关键的一环.下面来看看怎么使用.至于为什么需要「实例角色」可以看看AWS的描述

> Applications must sign their API requests with AWS credentials. Therefore, if you are an application developer, you need a strategy for managing credentials for your applications that run on EC2 instances. For example, you can securely distribute your AWS credentials to the instances, enabling the applications on those instances to use your credentials to sign requests, while protecting your credentials from other users. However, it's challenging to securely distribute credentials to each instance, especially those that AWS creates on your behalf, such as Spot Instances or instances in Auto Scaling groups. You must also be able to update the credentials on each instance when you rotate your AWS credentials.



# 获取

1. 进入阿里云ECS控制台的实例详情页中的其他信息版块

   <img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/blog/2021_09_28_15_59_1632815955199.png" alt="Screen Shot 2021-09-28 at 15.58.07" style="zoom:50%;" />

2. 授权,前提是创建好对应的[服务角色](https://help.aliyun.com/document_detail/116800.html)

3. 本机直接Http访问获得STS Token

   ```shell
   curl http://100.100.100.200/latest/meta-data/Ram/security-credentials/MyEcsRole
   ```

   ```json
    {
     "AccessKeyId" : "STS.NTdivWd4drUxehU3HuLRHGXbC",
     "AccessKeySecret" : "ZvpNNRVQNspQNTytiR5TdBm4pH74PXhTUy52x9HFSix",
     "Expiration" : "2021-09-28T14:00:11Z",
     "SecurityToken" : "token",
     "LastUpdated" : "2021-09-28T08:00:11Z",
     "Code" : "Success"
   }
   ```

# 使用



```java
import com.aliyuncs.CommonRequest;
import com.aliyuncs.CommonResponse;
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.auth.InstanceProfileCredentials;
import com.aliyuncs.http.ProtocolType;
import com.aliyuncs.profile.DefaultProfile;

public class InstanceRoleTest {

    // 测试ECS实例角色获取到的STS Token

    public static void main(String[] args) throws Exception {

        String accessKeyId = "STS.NTdivWd4drUxehU3HuLRHGXbC";
        String accessKeySecret = "ZvpNNRVQNspQNTytiR5TdBm4pH74PXhTUy52x9HFSix";
        String securityToken = "Token";
        String expiration = "2021-09-28T14:00:11Z";
        InstanceProfileCredentials credentials = new InstanceProfileCredentials(accessKeyId, accessKeySecret,
                securityToken, expiration, 3600 * 6);
        DefaultProfile AliyunProfile = DefaultProfile.getProfile("cn-hangzhou");
        IAcsClient client = new DefaultAcsClient(AliyunProfile, credentials);

        CommonRequest request = new CommonRequest();
        request.setSysDomain("ecs.aliyuncs.com");
        request.setSysVersion("2014-05-26");
        request.setSysAction("DescribeRegions");
        request.setSysProtocol(ProtocolType.HTTPS);
        CommonResponse response = client.getCommonResponse(request);
        String result = response.getData();
        System.out.println(result);

    }

}

```

