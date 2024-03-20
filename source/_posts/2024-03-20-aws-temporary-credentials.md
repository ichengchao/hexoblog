---
title: AWS的无AK实践
date: 2024-03-20 09:00:00
tags:
 - tech
 - cloud
---



# 前言![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897621.png)

数字化时代，信息安全已成为组织和个人最为关注的核心问题之一。随着技术的快速发展，尤其是云计算技术的广泛应用，信息安全的挑战也日益复杂和多样化。在众多的信息安全措施和理论中，有三个基本但至关重要的概念构成了保护信息系统不可或缺的基石，它们通常被称为信息安全的：「三要素」：Authentication、Authorization、Accounting。这三个要素共同构成了确保信息系统安全的基本框架，帮助组织有效防御各种安全威胁，确保数字资产的安全。
而认证是信息安全的第一道防线，它的主要目的是确保访问系统和数据的是合法的用户或设备。在一个安全的信息系统中，认证过程是用户或设备在请求访问资源之前必须完成的步骤，通过这一过程，系统能够验证请求方的身份是否为其声称的那个身份。所以认证机制的强度直接影响到整个信息系统的安全性。一个弱的认证系统可能容易被攻击者破解，导致未经授权的访问，而一个强的认证系统可以大大减少这种风险。认证技术有多种形式，包括传统的用户名和密码、数字证书、生物识别技术、一次性密码「OTP」以及多因素认证「MFA」等等。随着技术的进步，这些认证方法也在不断发展，以应对不断变化的安全威胁。而当我们讨论「Authentication」一般会分成两个大类：「人员身份」和「程序身份」。本文就是从「程序身份」的角度出发看看AWS是如何推荐他的用户通过使用「临时凭证」来最大程度的保证身份安全的。

# 为什么要使用「临时凭证」

![image-20240320092107708](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897671.png)
在讨论程序身份认证的时候，我们必须认识到保护身份凭据安全的两个核心原则：「减少暴露面积」和「减少暴露时长」，这两个原则对于程序凭证的安全使用至关重要。之前我们用程序访问AWS的时候一般都是通过AK（包含access key ID和access key secret)来实现。这种做法虽然简单直接，但存在一些显著的安全隐患。

1. 生成的AK都是**永久有效**的，这直接**违背了「减少暴露时长」的安全原则**。意味着一旦密钥被泄露，攻击者就可以不受时间限制内访问账户资源，除非密钥被发现并撤销。
2. 因为便利性，许多开发同学将这些AK**硬编码**在程序代码中，进一步增加了被恶意用户发现和利用的风险。这种做法**违背了「减少暴露面积」的原则**，因为代码库、配置文件或日志等多个地方可能会不小心暴露这些敏感信息。这样不当使用导致AK的例子举不胜举，造成严重后果更是比比皆是。
3. **AK就相当于一套静态的用户名+密码**，随着管理的AK数量越来越多，定期凭证轮换并确保这些AK的安全存储是非常有挑战的。

永久AK有这么多的问题，那有没有更好的方案呢？**有，答案就是不使用永久AK**，所以近些年来AWS不管在产品层面还是解决方案层面都强烈推荐用户使用「临时凭证」来替代永久AK，接下来我们来看看AWS是如果在各个场景中来做到真正的无AK的。

# 如何使用「临时凭证」

![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897690.png)
当我们在AWS的IAM控制台创建一把AK的时候，会弹出来一个前置界面。这里面很好的列举了几种常见应用场景，虽然看起来有很多，当本质上都是通过各种各样的方式去扮演Role从而获取到「临时凭证」。下面我们将用5个高频场景来说明：

1. CLI 
2. 本地开发
3. 线上部署「AWS环境」
4. 线上部署「非AWS环境」
5. App端使用AWS服务

### 场景一：CLI

命令行界面（CLI）是程序开发中的一个重要工具，当我们需要使用CLI访问AWS的时候有两种方式：1. 直接使用AWS控制台的Cloudshell，里面预置了登录态的身份凭据以及其他常用的工具。2. 本地使用CLI。第一种方式本来就是不涉及AK，我们重点来看看在本地使用CLI的方式。
✕ 不推荐： 在本地直接使用AK的方式是很简单的，配置完后就会在~/.aws/credentials中把刚才配置的AK明文保存下来
![Screenshot 2024-03-07 at 17.16.24.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897712.png)
✓推荐：**配合IAM Identity Center的SSO方案**

> 使用这个方案的前提是已经配置好了IdP和IAM Identity Center的SSO集成，而且已经为对应的用户配置好了权限。这部分的操作就不演示了，有需要的可以查看官方的文档。

![Screen-2024-03-07-190056.gif](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897945.gif)
这里我模拟了一个自己的IdP（当然也可以使用AAD或者Okta之类的代替），效果就是只需要使用IdP的用户名密码（类似我们内部的BUC账号)就能换取CLI的临时密钥。临时凭证就保存在了~/.aws/sso/cache/***.json中

```json
{
  "startUrl": "https://d-9a672420ed.awsapps.com/start#",
  "region": "us-east-2",
  "accessToken": "aoaAAAAAGX...........",
  "expiresAt": "2024-03-07T12:00:32Z",
  "clientId": "XOpMzFb...........",
  "clientSecret": "eyJraWQiO...........",
  "registrationExpiresAt": "2024-06-05T10:59:54Z",
  "refreshToken": "aorAAAAAGXqDs..........."
}
```

可以看到临时凭证的有效期只有一个小时，但是过期了会使用refreshToken自动刷新( 如果refreshToken过期了需要使用aws sso login --sso-session XXX重新登录)，所以使用过程中无感，体验非常好。总结一下：这种方式就是使用现有IdP的账号密码换取「临时凭证」来供CLI使用，相比于明文AK（当于一套静态的账户和密码，越多套账密就越不安全）来说，不仅极大地缩小了潜在的安全风险窗口，还保持了很高的便利性。

### 场景二：本地开发「以Java SDK为例」

✕ 不推荐：使用默认明文AK的profile（下面这个例子中使用的default profile是明文的AK）

```java
package democmp.aws;

import software.amazon.awssdk.auth.credentials.ProfileCredentialsProvider;
import software.amazon.awssdk.services.sts.StsClient;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityRequest;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityResponse;

public class StsGetCallerIdentityExample {
    public static void main(String[] args) {
        // 创建一个 StsClient 实例
        StsClient stsClient = StsClient.builder()
            .credentialsProvider(ProfileCredentialsProvider.builder().build()).build();

        // 构造请求并获取调用者身份信息
        GetCallerIdentityRequest request = GetCallerIdentityRequest.builder().build();
        GetCallerIdentityResponse response = stsClient.getCallerIdentity(request);

        // 打印调用者身份信息
        System.out.println("User ID: " + response.userId());
        System.out.println("Account: " + response.account());
        System.out.println("ARN: " + response.arn());
    }
}
```

✓推荐：配合场景一中已经配置好的SSO profile，这个方案简单方便，而且跟CLI共享一套逻辑。当然这里也可以使用场景四中「Roles Anywhere」的Profile。

```xml
<dependency>
  <groupId>software.amazon.awssdk</groupId>
  <artifactId>sts</artifactId>
  <version>2.25.3</version>
</dependency>
<dependency>
  <groupId>software.amazon.awssdk</groupId>
  <artifactId>sso</artifactId>
  <version>2.25.3</version>
</dependency>
<dependency>
  <groupId>software.amazon.awssdk</groupId>
  <artifactId>ssooidc</artifactId>
  <version>2.25.3</version>
</dependency>
```

```java
package democmp.aws;

import software.amazon.awssdk.auth.credentials.ProfileCredentialsProvider;
import software.amazon.awssdk.services.sts.StsClient;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityRequest;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityResponse;

public class StsGetCallerIdentityExample {

    public static void main(String[] args) {
        // 通过sso profile创建
        StsClient stsClient = StsClient.builder()
        .credentialsProvider(ProfileCredentialsProvider.create("AWSReadOnlyAccess-433312851566")).build();

        // 构造请求并获取调用者身份信息
        GetCallerIdentityRequest request = GetCallerIdentityRequest.builder().build();
        GetCallerIdentityResponse response = stsClient.getCallerIdentity(request);

        // 打印调用者身份信息
        System.out.println("User ID: " + response.userId());
        System.out.println("Account: " + response.account());
        System.out.println("ARN: " + response.arn());
    }
}

```

```java
User ID: AROAWJY3**********:cindy@chengchao.name
Account: 4333128******
ARN: arn:aws:sts::4333128******:assumed-role/AWSReservedSSO_AWSReadOnlyAccess_1ee707d279e5b2c2/cindy@chengchao.name
```

### 场景三：线上部署「AWS环境」

![roles-usingrole-ec2roleinstance.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897967.png)
以AWS的EC2为例，就是使用EC2的Instance Profile的功能，使用起来也很简单，在控制台上给运行服务的EC2设置Instance Profile，这个过程就是赋予EC2一个角色。
![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897974.png)
![Screenshot 2024-03-12 at 11.36.11.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710897995.png)

```java
package democmp.aws;

import software.amazon.awssdk.auth.credentials.InstanceProfileCredentialsProvider;
import software.amazon.awssdk.services.sts.StsClient;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityRequest;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityResponse;

public class StsGetCallerIdentityInstanceRoleExample {

    public static void main(String[] args) {

        StsClient stsClient = StsClient.builder()
            .credentialsProvider(InstanceProfileCredentialsProvider.builder().build()).build();

        // 构造请求并获取调用者身份信息
        GetCallerIdentityRequest request = GetCallerIdentityRequest.builder().build();
        GetCallerIdentityResponse response = stsClient.getCallerIdentity(request);

        // 打印调用者身份信息
        System.out.println("------------------------------------------");
        System.out.println("User ID: " + response.userId());
        System.out.println("Account: " + response.account());
        System.out.println("ARN: " + response.arn());
        System.out.println("------------------------------------------");

    }

}
```

```
------------------------------------------
User ID: AROAWJY3VMZXD2PLPYNXQ:i-0b24c62fe9108cccc
Account: 433312851566
ARN: arn:aws:sts::433312851566:assumed-role/my-ec2-test-role/i-0b24c62fe9108cccc
------------------------------------------
```

### 场景四：线上部署「非AWS环境」

![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898009.png)
场景三的Instance Profile虽然非常方便，也不用关心凭证轮转的问题。但是如果workload不是部署在AWS的环境中，这个方案就用不了了。为了解决这个问题，AWS在[2022年7月6号](https://aws.amazon.com/cn/about-aws/whats-new/2022/07/aws-identity-access-management-iam-roles-anywhere-workloads-outside-aws/)发布了一项新的服务「[Roles Anywhere](https://docs.aws.amazon.com/rolesanywhere/latest/userguide/introduction.html)」，有了这个方案，就可以在非AWS的环境来使用基于角色的临时凭证了。下面我们就用阿里云的ECS以及阿里云的私有证书服务来演示一下如何使用「Roles Anywhere」。
**第一步**：在阿里云的数字证书管理服务中创建一个测试用的私有CA(当然也可以使用openssl来代替)，这里需要注意的一个点就是私钥算法需要选择RSA_2048及以上，**RSA_1024的算法AWS已经不支持**。启动子CA，并申请一张证书。完成后会拿到3个文件：

1. CA的证书内容，点开子CA的「详情」里可以直接复制
2. 下载由子CA签发的证书，选择「PEM」格式，包含两个文件：XXX.key 和 XXX.pem

![Screenshot 2024-03-12 at 23.36.52.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898033.png)![Screenshot 2024-03-12 at 21.48.26.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898042.png)
**第二步**：把子CA的证书上传到AWS的RoleAnywhere作为信任端点，并创建对应的描述配置并加入对应的角色（可以是多个角色），这个就不多介绍，直接看AWS官方文档就可以
**第三步**：下载[aws_signing_helper](https://docs.aws.amazon.com/zh_cn/rolesanywhere/latest/userguide/credential-helper.html)，按照下面的格式分别填入对应的参数就可以了，最后一个参数就是需要扮演的role name，如果没有意外的话应该就能换出来临时的凭据了

```
/Users/charles/.aws/aws_signing_helper credential-process \
    --certificate /Users/charles/test/aliyunca/server.pem --private-key /Users/charles/test/aliyunca/server.key \
    --trust-anchor-arn arn:aws:rolesanywhere:us-east-2:433312851566:trust-anchor/33141e10-5d6c-40e7-870d-1e6deca4bc55 \
    --profile-arn arn:aws:rolesanywhere:us-east-2:433312851566:profile/974afdcb-c416-4921-a1c3-37e523f8b4ef \
    --role-arn arn:aws:iam::433312851566:role/rolesanywhere-ec2readonly
```

![Screenshot 2024-03-12 at 23.31.31.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898053.png)
配置一个credential_process到CLI的config里面，这样CLI也能使用Roles Anywhere了
![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898064.png)
![35170890-0c94-42fe-b8cb-3a11264b0c18](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898849.png)

用SDK的时候同样也能使用profile的方式去使用，以Java SDK为例：

```java
package democmp.aws;

import software.amazon.awssdk.auth.credentials.ProfileCredentialsProvider;
import software.amazon.awssdk.services.sts.StsClient;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityRequest;
import software.amazon.awssdk.services.sts.model.GetCallerIdentityResponse;

public class StsGetCallerIdentityExample {

    public static void main(String[] args) {
        // 创建一个 StsClient 实例,使用credential_process的profile
        StsClient stsClient = StsClient.builder()
        .credentialsProvider(ProfileCredentialsProvider.create("roleanywheretest")).build();

        // 构造请求并获取调用者身份信息
        GetCallerIdentityRequest request = GetCallerIdentityRequest.builder().build();
        GetCallerIdentityResponse response = stsClient.getCallerIdentity(request);

        // 打印调用者身份信息
        System.out.println("User ID: " + response.userId());
        System.out.println("Account: " + response.account());
        System.out.println("ARN: " + response.arn());

    }

}
```

看完上面这个例子，大家可能会有疑问，万一客户端的证书泄露了怎么办，其实也很简单，「Roles Anywhere」是支持CRL（certificate revocation list）的导入的，简单的说就是一串已经注销的的证书列表，这样在验证的时候如果命中了CRL就默认不通过。所以可以看到「Roles Anywhere」很巧妙的把证书那套完善的管理流程都嫁接过来了。

### 场景五：App端使用AWS服务

App端使用AWS服务最常见的一个例子就是使用S3来上传和下载文件。不管是社交软件发布照片，还是电商软件发布商品详情，都大量使用了这个服务。这里有个前提假设就是终端用户上传下载的文件数据是不经过过服务端的，因为流量大的话服务端可能会扛不住，希望使用S3来卸载掉这个压力。那问题就来了，我总不能生成明文的AK下发到App端吧（虽然我知道有些App就是这么干的囧）？AWS提供了几个方案来解决这个问题，比如使用「Cognito」，预签名URL等。下面我就演示一下如何使用预签名URL的方式在App端完成上传和下载。我摘取了几个网上的图片，很形象的解释了这个过程。
**上传场景**
![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898077.png)
![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898084.png)
**下载场景**
![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898096.png)

![c5722fa4-c821-419b-8502-8f34bbf79ec1](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898446.webp)

生成预签名URL的过程也比较简单，使用Java SDK来写个例子

```java
package democmp.aws;

import java.time.Duration;

import software.amazon.awssdk.services.s3.model.GetObjectRequest;
import software.amazon.awssdk.services.s3.model.PutObjectRequest;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;
import software.amazon.awssdk.services.s3.presigner.model.GetObjectPresignRequest;
import software.amazon.awssdk.services.s3.presigner.model.PresignedGetObjectRequest;
import software.amazon.awssdk.services.s3.presigner.model.PresignedPutObjectRequest;
import software.amazon.awssdk.services.s3.presigner.model.PutObjectPresignRequest;

public class S3PresignedUrlTest {

    public static void main(String[] args) {
        S3Presigner presigner = S3Presigner.create();
        String bucketName = "charlesdemoappbucket";
        String fileUpload = "myupload.txt";
        String fileDownload = "mydownload.txt";

        // 生成用于上传的URL
        PutObjectRequest putObjectRequest = PutObjectRequest.builder()
            .bucket(bucketName)
            .key(fileUpload)
            .build();

        PutObjectPresignRequest putObjectPresignRequest = PutObjectPresignRequest.builder()
            .signatureDuration(Duration.ofMinutes(10)) // The URL expires in 10 minutes.
            .putObjectRequest(putObjectRequest)
            .build();

        PresignedPutObjectRequest presignedPutObjectRequest = presigner.presignPutObject(putObjectPresignRequest);

        // 生成用于下载的URL
        GetObjectRequest getObjectRequest = GetObjectRequest.builder()
            .bucket(bucketName)
            .key(fileDownload)
            .build();

        GetObjectPresignRequest getObjectPresignRequest = GetObjectPresignRequest.builder()
            .signatureDuration(Duration.ofMinutes(10)) // The URL will expire in 10 minutes.
            .getObjectRequest(getObjectRequest)
            .build();

        PresignedGetObjectRequest presignedGetObjectRequest = presigner.presignGetObject(getObjectPresignRequest);

        System.out.println("--------------upload------------------");
        System.out.println(presignedPutObjectRequest.url().toString());
        System.out.println("--------------download------------------");
        System.out.println(presignedGetObjectRequest.url().toString());

    }

}

```

![image.png](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2024_03_20_1710898138.png)
用下面的命令测试一下：

```bash
# upload test
curl -X PUT -T "upload_file_path" "presigned_URL_upload"
# download
wget -O downlaod.txt "presigned_URL_download"

```

# 总结

至此，我们快速的过了一遍各个常用场景下是如何使用「临时凭证」来代替AK的，从AWS走过的路可以看出来「无AK」的落地肯定是一个系统工程，就好比今天我们在生鲜领域的「全程冷链」一样，需要从生产->仓储->干线运输->前置仓->最后一公里配送一系列的环节都做到冷链，否则中间哪个环节失效了，对最终客户来说就是失效的。可以看到近些年来AWS正在从心智运营，解决方案，产品能力同时发力，三位一体不断逼近真正意义上的「无AK」。虽然上面的过程是使用AWS作为例子，但里面提到的绝大多数能力，我们阿里云也都是可以对标的。同时也给了我们一个可参考的路线，去实现阿里云自己全方位的「无AK」方案。
