---
title: 身份安全之SAML&OIDC
date: 2025-01-20 20:00:00
tags:
 - tech
 - cloud
 - saml
 - oidc
---

# ![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_01_20_1737342998.png)

# **前言**

在企业数字化转型进程中，如何在安全性、便利性和用户体验之间找到平衡，成为安全领域的一大核心命题。在之前的文章《[AWS的无AK方案](https://blog.chengchao.name/2024/03/20/aws-temporary-credentials/)》中，我们探讨了信息安全的「三要素」：Authentication、Authorization、Accounting，并重点分析了友商在「程序身份」中的无AK方案。然而，除了「程序身份」，「人员身份」同样是认证体系中不可或缺的一部分。如上图所示，与「程序身份」的无AK方案一样，针对「人员身份」的认证需求，单点登录（SSO）已成为一种行之有效的解决方案。SSO 能够让用户通过一组身份凭据无缝访问多个系统和服务，既简化了认证流程，又提升了安全性。在众多 SSO 技术中，SAML（Security Assertion Markup Language）和 OIDC（OpenID Connect）是最常见且广泛应用的两种标准，跟很多客户在交流的过程之中也会问这两个方案的一些细节和不同。当然想要了解这两个技术，最好的办法就是动手做一遍，本文就把阿里云当做SP，同时模拟一下基于SAML和OIDC的IdP，来理解这两个技术是如何实现系统间的身份信任的。

# **共通性**

SAML 和 OIDC 虽然在设计背景和技术实现上有所不同，但本质上都旨在解决身份验证的问题，并且其核心原理有很多相似之处。SAML（Security Assertion Markup Language）是一种专为身份验证而设计的协议，天然具有身份验证的功能，通过 XML 格式的安全断言（Assertions）在不同系统之间传递用户身份信息。而 OIDC（OpenID Connect）则是在 OAuth 2.0 协议的基础上扩展而来，加入了身份验证的能力。尽管 OIDC 的初衷是补充 OAuth 2.0 的认证功能，但它最终实现的身份验证机制与 SAML 十分相似。两种实现都依赖于非对称密钥加密技术，通过使用公钥加密和私钥签名，SAML 和 OIDC 都能够为客户端提供安全的身份信息传递方式，从而验证用户的合法身份。从技术角度来看，SAML 和 OIDC 在实现身份验证的目标和核心技术上具有很大的共通性。

# **SAML**

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_01_20_1737343051.png)

## **步骤介绍**

SAML 的基本概念和工作原理网上已经有大量资源可以参考，比如 [Cloudflare 的介绍](https://www.cloudflare.com/zh-cn/learning/access-management/what-is-saml/)。在这里，我们将跳过基础概念，直接进入主题，通过搭建一个简易的 SAML 身份提供商（IdP）来理解其工作原理。为了简化流程，我们省略一些初始步骤，直接从 IdP 返回 SAML Response 开始，完成一个最小化的演示。

整个流程可以分为以下几个关键步骤：

1. **创建密钥对并签发证书**

首先需要生成一个非对称密钥对（私钥和公钥），并基于公钥签发一张自签名证书。这张证书将在 SAML Response 中用于验证，以确保数据的完整性和来源可信。

1. **生成 IdP 元数据文件（IdP Metadata）并配置到 SP**

创建一个 IdP-meta.xml 文件，其中包含 IdP 的必要信息，例如 SSO 端点、证书等。然后，将该元数据文件配置到服务提供商（SP）侧，使 SP 能够正确识别并信任 IdP。

1. **生成 SAML Response 并提交到 SP**

使用私钥签名生成的 SAML Response，其中包含用户身份信息和相关声明。随后，通过 HTML 表单提交 SAML Response 到 SP 的 Assertion Consumer Service (ACS) URL。

1. **登录验证**

SP 验证 SAML Response 的签名并提取身份信息。如果验证通过，SP 将认为用户已成功认证，并允许用户访问。



通过以上步骤，可以快速模拟一个基本的 SAML IdP 实现。虽然过程简化，但它涵盖了核心要点，更加便于理解 SAML 的核心工作机制以及 IdP 和 SP 之间的交互逻辑。

## **1. 创建密钥对并签发证书**

使用RSA（2048）生成公钥和私钥, 保存备用

```java
Public Key: MIIBIjANBgkqhkiG9w0B********
Private Key: MIIEvgIBADANBgkqhki********
//生成密钥对，打印密钥信息，后面使用
public static void generateKeyPair() throws Exception {
    // 1. 创建 KeyPairGenerator 对象，指定 RSA 算法
    KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");

    // 2. 初始化 KeyPairGenerator，设置密钥长度（2048 位）
    keyPairGen.initialize(2048);

    // 3. 生成密钥对
    KeyPair keyPair = keyPairGen.generateKeyPair();

    // 3. 将密钥编码为 Base64 字符串
    String publicKeyString = Base64.getEncoder().encodeToString(keyPair.getPublic().getEncoded());
    String privateKeyString = Base64.getEncoder().encodeToString(keyPair.getPrivate().getEncoded());

    // 输出密钥字符串
    System.out.println("Public Key: " + publicKeyString);
    System.out.println("Private Key: " + privateKeyString);
}

//利用公钥生成自签名的X509证书
public static X509Certificate generateCertificate(KeyPair keyPair) throws Exception {
    Security.addProvider(new BouncyCastleProvider());
    X500Name issuer = new X500Name("CN=Chengchao CA");
    X500Name subject = new X500Name("CN=Chengchao SSO Demo");
    BigInteger serialNumber = BigInteger.valueOf(System.currentTimeMillis());
    Date notBefore = Date.from(Instant.now().minus(1, ChronoUnit.DAYS));
    Date notAfter = Date.from(Instant.now().plus(365, ChronoUnit.DAYS));
    JcaX509v3CertificateBuilder certBuilder = new JcaX509v3CertificateBuilder(
        issuer,
        serialNumber,
        notBefore,
        notAfter,
        subject,
        keyPair.getPublic());
    ContentSigner signer = new JcaContentSignerBuilder("SHA256WithRSAEncryption")
        .setProvider("BC")
        .build(keyPair.getPrivate());
    X509CertificateHolder certHolder = certBuilder.build(signer);
    return new JcaX509CertificateConverter()
        .setProvider("BC")
        .getCertificate(certHolder);
}
```

## **2. 生成 IdP 元数据文件（IdP Metadata）并配置到 SP**

利用上面生成好的公钥私钥构建IdPMetadata.xml, 并上传到阿里云（访问控制->SSO->角色SSO->SAML）

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_01_20_1737343062.png)

```java
public static final String IDP_ENTITY_ID = "chengchaoIdP-Demo";
//这个地址在本次演示中不会使用到，所以可以随便填
public static final String IDP_LOGIN_LOCATION = "https://test.com/samlLogin.do";

public static void main(String[] args) throws Exception {
    InitializationService.initialize();
    String publicKeyString = "MIIBIjANBgkqhkiG9w0B********";
    String privateKeyString = "MIIEvgIBADANBgkqhki********";
    KeyPair keyPair = paserKeypair(publicKeyString, privateKeyString);
    X509Certificate certificate = generateCertificate(keyPair);
    String idpMetaXMLString = generateIdpMetaXML(certificate); 
    //生成IdP Metadata.xml文件
    FileUtils.write(new File("/Users/charles/Desktop/chengchaoIdPMetadata.xml"), idpMetaXMLString, StandardCharsets.UTF_8);
}

public static KeyPair paserKeypair(String publicKeyString, String privateKeyString) throws Exception {
    byte[] publicKeyBytes = Base64.getDecoder().decode(publicKeyString);
    byte[] privateKeyBytes = Base64.getDecoder().decode(privateKeyString);
    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    PublicKey publicKey = keyFactory.generatePublic(new X509EncodedKeySpec(publicKeyBytes));
    PrivateKey privateKey = keyFactory.generatePrivate(new PKCS8EncodedKeySpec(privateKeyBytes));
    return new KeyPair(publicKey, privateKey);
}

public static String generateIdpMetaXML(X509Certificate certificate) throws Exception {
    EntityDescriptorBuilder entityDescriptorBuilder = new EntityDescriptorBuilder();
    EntityDescriptor entityDescriptor = entityDescriptorBuilder.buildObject();
    entityDescriptor.setEntityID(IDP_ENTITY_ID);

    //------------------------------------------------
    // 避免篇幅太长，省略掉一些代码
    //------------------------------------------------
   
    Transformer transformer = TransformerFactory.newInstance().newTransformer();
    transformer.transform(new DOMSource(element), new StreamResult(baos));
    // XMLHelper.writeNode(element, baos);
    String metaXMLStr = new String(baos.toByteArray());
    return metaXMLStr;
}
```

## **3. 生成 SAML Response 并提交到 SP**

1. 创建一个测试用的角色「RoleForDemoIdP」并分配只读的权限
2. 生成并打印SAML Response
3. 本地使用浏览器打开下面的html文件，将SAML Response粘贴进表单中并提交，可以看到就能成功登录阿里云了

```java
public static void main(String[] args) throws Exception {
    InitializationService.initialize();
    String publicKeyString = "MIIBIjANBgk****";
    String privateKeyString = "MIIEvgIBAD*****";
    KeyPair keyPair = paserKeypair(publicKeyString, privateKeyString);
    X509Certificate certificate = generateCertificate(keyPair);
    String idpMetaXMLString = generateIdpMetaXML(certificate);
    //生成IdP Metadata.xml文件
    FileUtils.write(new File("/Users/charles/Desktop/chengchaoIdPMetadata.xml"), idpMetaXMLString, StandardCharsets.UTF_8);
    
    //生成SAML Response
    Map<String, String> attributeMap = new HashMap<>();
    attributeMap.put("https://www.aliyun.com/SAML-Role/Attributes/RoleSessionName", "chengchao");
    attributeMap.put("https://www.aliyun.com/SAML-Role/Attributes/Role", "acs:ram::17642631*****:role/rolefordemoidp,acs:ram::17642631*****:saml-provider/DemoIdP");
    BasicX509Credential basicX509Credential = new BasicX509Credential(certificate, keyPair.getPrivate());
    String samlResponseStr = buildSamlResponse(basicX509Credential,attributeMap);
    String base64EncodeSAMLResponse = java.util.Base64.getEncoder().encodeToString(samlResponseStr.getBytes());
    System.out.println(base64EncodeSAMLResponse);
}


public static String buildSamlResponse(BasicX509Credential basicX509Credential,Map<String, String> attributeMap) throws Exception {
    //------------------------------------------------
    // 避免篇幅太长，省略掉一些代码
    //------------------------------------------------
    ResponseMarshaller marshaller = new ResponseMarshaller();
    Element element = marshaller.marshall(responseInitial);
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    Transformer transformer = TransformerFactory.newInstance().newTransformer();
    transformer.transform(new DOMSource(element), new StreamResult(baos));
    String responseStr = new String(baos.toByteArray());
    return responseStr;
}
<!-- 非常简单的一个页面，就是将SAML Response粘贴进文本框后点击提交 -->
<!DOCTYPE html>
<html lang="en">

<body>
    <form action="https://signin.aliyun.com/saml-role/sso" method="post">
        <textarea name="SAMLResponse"></textarea>
        <button type="submit">submit</button>
    </form>
</body>

</html>
```

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_01_20_1737343072.png)

## **4. 登录验证**

最后看一下登录的效果（动画）：

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_01_20_1737343083.gif)

# **OIDC**

## **步骤介绍**

同样我们将跳过 OIDC 的基础概念，直接进入主题，通过搭建一个简易的 OIDC 身份提供商（IdP）来理解其工作原理和交互过程。尽管目前我们阿里云没有提供直接的 OIDC SSO 登录能力，但提供了[**AssumeRoleWithOIDC**](https://help.aliyun.com/zh/ram/developer-reference/api-sts-2015-04-01-assumerolewithoidc)的API ，可以验证 OIDC 流程的有效性。通过该 API，阿里云允许用户基于第三方 OIDC 身份提供商的签发凭证，获取临时访问凭据，从而实现 OIDC 与阿里云的集成。

整个流程可以分为以下几个关键步骤：

**1. 创建密钥对**

首先需要生成一个非对称密钥对（私钥和公钥），并通过公钥生成一个 JSON Web Key Set (JWKS) 文件。JWKS 文件将用于发布公钥信息，确保其他服务能够验证签名的 ID Token，从而保证数据的完整性和来源可信。

**2. 发布 openid-configuration,并在阿里云侧完成配置**

发布一个.well-known/openid-configuration的https链接，其中包含 OIDC 身份提供商（IdP）的必要信息，例如 issuer（签发者）、jwks_uri（公钥文件地址）等。然后，在阿里云控制台配置该 OIDC 提供商信息，使阿里云能够识别并信任该IdP。

**3. 签发 ID Token**

使用私钥生成并签名一个符合 OIDC 标准的 ID Token，其中包含用户身份信息和相关声明，例如 iss（签发者）、sub（用户标识）、exp（过期时间）等。生成的 ID Token 将作为凭据，用于向阿里云换取临时凭据。

**4. 调用阿里云 AssumeRoleWithOIDC 接口完成验证**

将签发的 ID Token 提交给阿里云的 AssumeRoleWithOIDC API，同时提供 OIDC 提供商的 ARN 和目标角色的 ARN。阿里云验证 ID Token 的签名和声明内容，通过后将返回一组临时访问凭据（STS），用户即可使用这些凭据访问指定的云资源。

通过以上步骤，可以快速模拟一个基本的 OIDC 流程并与阿里云集成，涵盖了从密钥生成到认证验证的核心逻辑，有助于理解 OIDC 的工作原理。

## **1. 创建密钥对**

使用RSA生成密钥对，并打印成字符串备用

```java
public static RSAKey generateRSAKey() throws Exception {
    RSAKey raskey = new RSAKeyGenerator(2048)
        .keyID("demokey_001")
        .keyUse(new KeyUse("sig"))
        .algorithm(new Algorithm("RS256"))
        .generate();
    return raskey;
}
static final String RasKeyJsonString = "{\"p\":\"_h8cGVKS8yAgEpN3C******";
```

## **2. 发布 openid-configuration,并在阿里云侧完成配置**

这里我使用OSS的静态网站的能力快速搭建了一个endpoint，用来发布公钥信息https://oidctest.chengchao.name/.well-known/openid-configuration

```json
{
  "response_types_supported": [
    "code"
  ],
  "code_challenge_methods_supported": [
    "plain",
    "S256"
  ],
  "jwks_uri": "https://oidctest.chengchao.name/keys",
  "subject_types_supported": [
    "public"
  ],
  "id_token_signing_alg_values_supported": [
    "RS256"
  ],
  "scopes_supported": [
    "openid"
  ],
  "issuer": "https://oidctest.chengchao.name"
}
```

https://oidctest.chengchao.name/keys的内容生成也很简单，利用之前生成的RSAKey

```java
RSAKey MyRasKey = RSAKey.parse(RasKeyJsonString);
RSAKey rsaPublicJWK = MyRasKey.toPublicJWK();
String result = "{\"keys\":[" + rsaPublicJWK + "]}";
System.out.println(result);
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "use": "sig",
      "kid": "demokey_001",
      "alg": "RS256",
      "n": "57lmqqOU8HtpGHhg......."
    }
  ]
}
```

发布完openid-configuration后就可以在阿里云侧进行配置

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_01_20_1737343098.png)

## **3. 签发 ID Token**

利用之前生成的密钥对签发ID Token，其中issuer和audience需要和配置在阿里云侧的信息一一对应

issuer : https://oidctest.chengchao.name

audience : aliyunoss001

```java
public static String buildIDToken(String issuer, String subject, String audience) throws Exception {
    RSAKey MyRasKey = RSAKey.parse(RasKeyJsonString);
    String nonce = "";
    int oneweekseconds = 7 * 24 * 3600;
    int t = (int)(System.currentTimeMillis() / 1000);
    Map<String, Object> payload = new HashMap<>();
    payload.put("iss", issuer);
    payload.put("aud", audience);
    payload.put("exp", t + oneweekseconds);
    payload.put("iat", t);
    payload.put("sub", subject);
    payload.put("nonce", nonce);
    payload.put("jti", subject);
    JWSSigner signer = new RSASSASigner(MyRasKey);
    JWSObject jwsObject = new JWSObject(
        new JWSHeader.Builder(JWSAlgorithm.RS256).keyID(MyRasKey.getKeyID()).build(),
        new Payload(JsonUtils.toJsonString(payload)));
    jwsObject.sign(signer);
    String result = jwsObject.serialize();
    return result;
}

public static void main(String[] args) throws Exception {
    String idToken = buildIDToken("https://oidctest.chengchao.name", "chengchao", "aliyunoss001");
    System.out.println(idToken);
}
```

## **4. 调用阿里云 AssumeRoleWithOIDC 接口完成验证**

最后调用AssumeRoleWithOIDC的API就能成功换取到STS Token， 相当于也完成了基于OIDC的OSS登录

```java
public static void main(String[] args) throws Exception {
    String idToken = buildIDToken("https://oidctest.chengchao.name", "chengchao", "aliyunoss001");
    String stsResult = getSTSTokenWithOIDC(idToken);
    System.out.println(stsResult);
}
public static String getSTSTokenWithOIDC(String idToken) throws Exception {
    DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou");
    IAcsClient client = new DefaultAcsClient(profile);
    String action = "AssumeRoleWithOIDC";
    Map<String, String> params = new HashMap<>();
    params.put("OIDCProviderArn", "acs:ram::1764F******43:oidc-provider/OIDCTest001");
    params.put("RoleArn", "acs:ram::1764F******43:role/role-oidc001");
    params.put("OIDCToken", idToken);
    params.put("RoleSessionName", "chengchao");
    String result = commonInvoke(client, "sts.cn-hangzhou.aliyuncs.com", "2015-04-01", action, params);
    return result;
}
```

# **总结**

![img](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2025_01_20_1737343108.png)





通过以上对 SAML 和 OIDC 身份提供商（IdP）的模拟搭建，我们能够直观地理解它们的核心工作原理。这两种技术的内核实际上是非常相似的：它们都通过非对称加密技术，确保服务提供商（SP）能够信任身份提供商（IdP）签发的身份信息，从而实现单点登录（SSO）。无论是 SAML 中的 SAML Response 还是 OIDC 中的 ID Token，其本质都是由 IdP 使用私钥签名的身份信息载体，SP 使用公钥验证其真实性和完整性。通过这样的机制，双方建立起了基于信任的安全认证流程，使用户能够在不同系统之间无缝切换，无需重复登录。目前，这两种技术已被广泛应用于主流的身份服务商中。例如，Okta、Microsoft Entra ID（AAD）、阿里云的IDaaS等都全面支持 SAML 和 OIDC 协议，以满足企业在身份认证与单点登录方面的多样化需求。

最后，由于篇幅的原因，文中的代码是不完整的，附上[Demo 代码链接](https://gist.github.com/ichengchao/11cce0e42021c287195bb8dc82e46b7f)，有兴趣的同学可以在本地测试一下。
