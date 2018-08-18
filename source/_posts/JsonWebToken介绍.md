---
title: JsonWebToken介绍
date: 2018-08-18 22:47:52
tags: jwt
categories: 前端
---

# 前言
一开始了解这个东西以为挺厉害的，后来和 [delveshal](delveshal.github.io) 针对很多场景聊过，对 jwt 有更深刻的认识。

# JWT 是什么
>JSON Web Token（JWT）是一个开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），它定义了一种紧凑且独立的方式，可以在各方之间作为JSON对象安全地传输信息。此信息可以通过数字签名进行验证和信任。JWT可以使用秘密（使用**HMAC**算法）或使用**RSA**或**ECDSA**的公钥/私钥对进行**签名**。

## 结构
JWT 由三个部分组成（由点（.）分隔，它们是：
+ 头(header)
+ 载荷(payload)
+ 签名(signature)

因此，JWT 通常如下所示。
`xxxxx.yyyyy.zzzzz`
### 头
通常由两部分组成：令牌的类型，即 JWT，以及正在使用的散列算法，例如 HMAC SHA256 或 RSA。
例如：
 ```
{
  "alg": "HS256",
  "typ": "JWT"
}
 ```
然后，这个JSON 被编码为 Base64Url，形成 JWT 的第一部分。
### 载荷
令牌的第二部分是载荷，其中包含声明。声明是关于实体（通常是用户）和其他数据的声明(claim)。JWT 规定了7个官方字段：
```
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```
除了官方字段，你还可以在这个部分定义私有字段：
```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
然后，载荷经过 Base64Url 编码，形成 JWT 的第二部分。

请注意，对于签名令牌，此信息虽然可以防止被篡改，但任何人都可以读取。除非加密，否则不要将秘密信息放在 JWT 的有效负载或头元素中。
### 签名
要创建签名部分，必须采用编码标头，编码的载荷，秘钥(secret)，标头中指定的算法，并对其进行签名。

例如，如果要使用 HMAC SHA256 算法，将按以下方式创建签名：
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
签名用于验证消息在此过程中未被更改，并且，在使用私钥签名的令牌的情况下，它还可以验证 JWT 的发件人是否是它所声称的人。

### 合在一起
按照 `头.载荷.签名`，用 `.` 串联起来形成这种形式，最终是以下格式：
![](https://upload-images.jianshu.io/upload_images/5834506-18f7e6719dde507f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然，实际中没有分行，都是一行，只是为了容易看才写成这种形式。

# JWT 怎么使用
在身份验证中，当用户使用其凭据成功登录时，将返回 JWT 。由于令牌是凭证，因此必须非常小心以防止出现安全问题。一般情况下，不应该将令牌保留的时间超过要求。

每当用户想要访问受保护的路由或资源时，用户代理应该使用承载模式发送 JWT ，通常在  `Authorization` 头中。http 头的内容应如下所示：
`Authorization: Bearer <token>`
在某些情况下，这可以是无状态授权机制。服务器的受保护路由将检查 `Authorization` 标头中的有效 JWT ，如果存在，则允许用户访问受保护资源。如果 JWT 包含必要的数据，则可以减少查询数据库以进行某些操作的需要，尽管可能并非总是如此。

如果在 `Authorization` 头中发送令牌，则跨域资源共享（`CORS`）将不会成为问题，因为它不使用 `cookie`。

另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面。

# JWT 使用场景
要讲使用场景，就要从它的优缺点讲起
## 优点
+ 体积小，因而传输速度快
+ 传输方式多样，可以通过URL/POST参数/HTTP头部等方式传输
+ 严格的结构化。它自身（在 payload 中）就包含了所有与用户相关的验证消息，如用户可访问路由、访问有效期等信息，服务器无需再去连接数据库验证信息的有效性，并且 payload 支持为你的应用而定制化。
+ 支持跨域验证，可以应用于单点登录。

## 缺点
+ Token有长度限制
+ Token不能撤销
+ 需要token有失效时间限制(exp)

有了以上的优点缺点的对比，就知道 JWT 使用场景：它适合用作单次使用授权令牌。

例如，您可以运行文件托管服务，用户必须通过身份验证才能下载其文件，但文件本身由单独的无状态“下载服务器”提供。在这种情况下，您可能希望让应用程序服务器（服务器A）发出一次性的“下载令牌”，客户端然后可以使用它从下载服务器（服务器B）下载文件。

当以这种方式使用 JWT 时，有一些特定的属性：

+ 令牌是短时间的。他们只需要有效的几分钟，让客户端启动下载。

+ 令牌只能预期使用一次。应用程序服务器将为每次下载发出一个新的令牌，因此任何一个令牌仅用于请求一次文件，然后丢弃。根本没有持续的状态。

+ 应用程序服务器仍然使用会话。这只是使用令牌授权单个下载的下载服务器，因为它不需要持久状态。

正如你可以看到的，组合会话和JWT令牌是完全合理的 - 它们都有自己的目的，有时候需要两者。只需不要使用JWT持久的长寿命数据。

>参考
>[JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
>[JWT官网](https://jwt.io/introduction/)
>[聊一聊JWT与session](https://juejin.im/post/5a437441f265da43294e54c3#heading-3)
>[cookie-session机制与JWT机制对比](https://www.jianshu.com/p/cbe253281d26)
>[理解JWT（JSON Web Token）认证及实践](https://mp.weixin.qq.com/s/gUgh_kmMu0Hmobeah7wNLQ)
>[请停止使用 JWT 认证](https://www.zcfy.cc/article/stop-using-jwt-for-sessions-joepie91s-ramblings)