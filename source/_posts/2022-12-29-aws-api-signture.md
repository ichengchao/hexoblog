---
title: AWS API签名
date: 2022-12-29 19:03:48
tags:

 - tech
 - api
 - aws

---



#  背景

调用各大平台的API都需要签名,所以调用AWS的API自然也是一样.原本我想的非常简单并且觉得网上的资料和示例应该一抓一大把.但是事与愿违,如果直接使用平台的提供的SDK或者CLI的话,的确是非常简单,开箱即用.但是几乎没有完全从头开始实现签名的,就连官方文档也是讲的不是特别清楚.所以我准备自己来把自己的实现过程记录下来.

- AWS V4签名认证(Java实现): https://www.jianshu.com/p/66338d20b4ea
  - 这个还算清楚,不过还是不完美,比如没有POST的示例
- 官方文档: https://docs.aws.amazon.com/zh_cn/general/latest/gr/sigv4_signing.html
  - 这里要为AWS中文文档团队点赞,难得看到中文的官方文档质量比英文的高的,但是实现层面还是有很多小坑的



# 准备

我觉得网上资料不清晰的很重要的一个问题就是前置条件模糊.所以在开始之前我们先把一些需要用到的东西罗列一下,中心化的服务Region就填"us-east-1".以下所有的计算过程都是基于这些配置

|     配置项 | 值                                       |
| ---------: | :--------------------------------------- |
|  AccessKey | AKIAWJY3VMZXCIL6BDSQ                     |
|  SecretKey | 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 |
|     Region | us-east-1                                |
|    Service | iam                                      |
|        URL | https://iam.amazonaws.com/               |
| X-Amz-Date | 20221229T082438Z                         |

过程中需要的几个函数也要标准化一下

```
# sha256Hex
空字符串:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
test:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08

# hmacSHA256Hex
密钥:test 值:111
结果:d37dffac8a59021d523b56de30c62f17642ea0d6e13c53c595da87fa3f7c0ccb
```

**

有了这些对齐的字段,后面的计算过程的每一步都是可以校验的



# 概览

AWS的签名过程总共分成四步:

![         签名版本 4 流程       ](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2022_12_29_1672299775.png)

1. 创建规划的请求结构: 就是把HTTP请求按照一定的规划组合在一起
2. 将第一步请求体做sha256Hex摘要加上一些特定字段组合签名目标字符串
3. 按照一定规则派生出密钥,并对第二步的签名目标做签名
4. 将签名结果加到HTTP Header中



# Step1

先看一下HTTP请求体,拼接的规则可以查看官方文档

```
GET /?Action=ListUsers&Version=2010-05-08 HTTP/1.1
Host: iam.amazonaws.com
X-Amz-Date: 20221229T082438Z
Authorization: AWS4-HMAC-SHA256 Credential=AKIAWJY3VMZXCIL6BDSQ/20221229/us-east-1/iam/aws4_request, SignedHeaders=host;x-amz-date, Signature=b1869a003e01966cc1a7667dde231aaaf89db6f2c614822fc7a14012e91e1596
```

拼接出来的字符串,最后面的payload的sha256Hex就是空字符串的sha256Hex

```
GET
/
Action=ListUsers&Version=2010-05-08
host:iam.amazonaws.com
x-amz-date:20221229T082438Z

host;x-amz-date
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

实现代码

```java
//DateFormatUtils.formatUTC(now, "yyyyMMdd'T'HHmmss'Z'");
String datetime = "20221229T082438Z";
String httpMethod = "GET";
String url = "https://iam.amazonaws.com/";
String host = new URL(url).getHost();
Map<String, String> urlParams = new TreeMap<>();
urlParams.put("Action", "ListUsers");
urlParams.put("Version", "2010-05-08");
Map<String, String> headerMap = new TreeMap<>();
headerMap.put("X-Amz-Date", datetime);
headerMap.put("Host", host);
String body = null;
// ---------------------------------------------------------------------------------------------------- 
public static String buildCanonicalRequest(String httpMethod, String url, Map<String, String> urlParams,
        Map<String, String> headerMap, String body) throws Exception {
        StringBuilder sb = new StringBuilder();
        sb.append(httpMethod);
        sb.append("\n");
        sb.append(new URL(url).getPath());
        sb.append("\n");
        if (urlParams != null && !urlParams.isEmpty()) {
            for (Map.Entry<String, String> entry : urlParams.entrySet()) {
                sb.append(entry.getKey());
                sb.append("=");
                sb.append(entry.getValue());
                sb.append("&");
            }
            sb.deleteCharAt(sb.lastIndexOf("&"));
        }
        sb.append("\n");

        if (headerMap != null && !headerMap.isEmpty()) {
            String headerList = headerMap.keySet().stream().map(String::toLowerCase).collect(Collectors.joining(";"));
            for (Map.Entry<String, String> entry : headerMap.entrySet()) {
                sb.append(StringUtils.lowerCase(entry.getKey()));
                sb.append(":");
                sb.append(entry.getValue());
                sb.append("\n");
            }
            sb.append("\n");
            sb.append(headerList);
            sb.append("\n");
        }

        if (StringUtils.isBlank(body)) {
            body = "";
        }
        // add sha256hex payload
        sb.append(DigestUtils.sha256Hex(body));

        return sb.toString();
    }
```

# Step2

构建签名目标字符串,对第二步的结果做sha256Hex

```
GET
/
Action=ListUsers&Version=2010-05-08
host:iam.amazonaws.com
x-amz-date:20221229T082438Z

host;x-amz-date
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
------------------------------------------------------------------------------
sha256Hex: f87889c26a9eafc1418c7291faab6fa0e1533eeebdeeb910390560da1f329304
------------------------------------------------------------------------------

```

拼接成签名目标字符串

```
AWS4-HMAC-SHA256
20221229T082438Z
20221229/us-east-1/iam/aws4_request
f87889c26a9eafc1418c7291faab6fa0e1533eeebdeeb910390560da1f329304
```

代码实现:

```java
String datetime = "20221229T082438Z";
//String scope = date + "/" + region + "/" + service + "/aws4_request";
String scope = "20221229/us-east-1/iam/aws4_request";
String requestHashHex = "f87889c26a9eafc1418c7291faab6fa0e1533eeebdeeb910390560da1f329304";
// ---------------------------------------------------------------------------------------------------- 
public static String buildStringToSign(String datetime, String scope, String requestHashHex) {
        StringBuilder sb = new StringBuilder();
        sb.append("AWS4-HMAC-SHA256");
        sb.append("\n");
        sb.append(datetime);
        sb.append("\n");
        sb.append(scope);
        sb.append("\n");
        sb.append(requestHashHex);
        return sb.toString();
    }
```

# Step3

签名结果:

```
AWS4-HMAC-SHA256
20221229T082438Z
20221229/us-east-1/iam/aws4_request
f87889c26a9eafc1418c7291faab6fa0e1533eeebdeeb910390560da1f329304
-------------------------------------------------------------------
签名结果: fb06ab6c15a164473a7c598df41e3735607f4992aedbd47a534214012ecb8ec2
```

代码实现:

```java
String secretKey = "ZMXkC4V2PlF5mkmLQFb+FecxDXySOjET7pQwR3ZD";
String date = "20221229";
String region = "us-east-1";
String service = "iam";
String stringToSign = 
"""
AWS4-HMAC-SHA256
20221229T082438Z
20221229/us-east-1/iam/aws4_request
f87889c26a9eafc1418c7291faab6fa0e1533eeebdeeb910390560da1f329304
""";
// ---------------------------------------------------------------------------------------------------- 
public static String sign(String secretKey, String date, String region, String service, String stringToSign)
        throws Exception {
        byte[] kSecret = ("AWS4" + secretKey).getBytes("UTF8");
        byte[] kDate = HmacUtils.hmacSHA256(kSecret, date);
        byte[] kRegion = HmacUtils.hmacSHA256(kDate, region);
        byte[] kService = HmacUtils.hmacSHA256(kRegion, service);
        byte[] kSigning = HmacUtils.hmacSHA256(kService, "aws4_request");
        byte[] signByte = HmacUtils.hmacSHA256(kSigning, stringToSign);
        String sign = Hex.encodeHexString(signByte);
        return sign;
    }
```



# Step4

将签名结果加到HTTP Header中:

```
AWS4-HMAC-SHA256 Credential=AKIAWJY3VMZXCIL6BDSQ/20221229/us-east-1/iam/aws4_request,SignedHeaders=host;x-amz-date,Signature=fb06ab6c15a164473a7c598df41e3735607f4992aedbd47a534214012ecb8ec2
```

代码实现:

```java
String accessKey = "AKIAWJY3VMZXCIL6BDSQ";
String scope = "20221229/us-east-1/iam/aws4_request";
String headerList = "host;x-amz-date";
String sign = "fb06ab6c15a164473a7c598df41e3735607f4992aedbd47a534214012ecb8ec2";
// ---------------------------------------------------------------------------------------------------- 
public static String buildAuthorization(String accessKey, String scope, String headerList, String sign) {
        StringBuilder authorizationBuilder = new StringBuilder();
        authorizationBuilder.append("AWS4-HMAC-SHA256");
        authorizationBuilder.append(" ");
        authorizationBuilder.append("Credential=" + accessKey);
        authorizationBuilder.append("/");
        authorizationBuilder.append(scope);
        authorizationBuilder.append(",");
        authorizationBuilder.append("SignedHeaders=");
        authorizationBuilder.append(headerList);
        authorizationBuilder.append(",");
        authorizationBuilder.append("Signature=");
        authorizationBuilder.append(sign);
        return authorizationBuilder.toString();
    }
```



# 最后

分享过程中几个用到的小工具:

- Postman: 这个不用介绍了,HTTP调试神器. 自带AWS的签名方式,可以用来做校验
-  Surge: 分析发出的https的请求,具体可以看这里: https://blog.chengchao.name/2022/05/05/debug-https/
- He3: 一个工具集,网址在这里: https://he3.app/zh/

完整代码在这里: https://gist.github.com/ichengchao/a08ff7052ea0f662b3e5c09d178e3e4c
