# OAuth2

模式选择

1. 单一服务器模式
2. SSO模式: CAS单点登录，OAuth2
3. Token模式
    - 优点：token无状态，session有状态；基于标准化：API可以采用标准化的JWT
    - 缺点：占用宽带，在服务器端无法销毁token

透明令牌：随机生成的字符串标识符，无法被猜测，资源服务器需要通过后端渠道发送回OAuth2授权服务器的令牌检查端点，才能检验令牌是否有效，并获取claims/scopes等信息

自包含令牌：包含关于用户和客户的元数据和声明(claims)，通过检查签名，期望的颁发者(issuer),期望的接收人aud(audience),或者scope,资源服务器可以在本地校验令牌，通常实现为签名的JWT

JWT（一种规范，主要实现有JWS，JWE）

1. 认证 Authentication；
2. 授权 Authorization // 注意这两个单词的区别；
3. 联合识别；
4. 客户端会话（无状态的会话）；
5. 客户端机密。

名词解释：

1. JWS：Signed JWT签名过的jwt
2. JWE：Encrypted JWT部分payload经过加密的jwt；**目前加密payload的操作不是很普及**
3. JWK：JWT的密钥，也就是我们常说的scret；
4. JWKset：JWT key set在非对称加密中，需要的是密钥对而非单独的密钥，在后文中会阐释；
5. JWA：当前JWT所用到的密码学算法；
6. nonsecure JWT：当头部的签名算法被设定为none的时候，该JWT是不安全的；因为签名的部分空缺，所有人都可以修改。

三部分：

- header:typ | alg | jti | cty
- payload 这一部分不能放关键信息，会被破解
- Registered Claims
  - iss  【issuer】发布者的url地址
  - sub 【subject】该JWT所面向的用户，用于处理特定应用，不是常用的字段
  - aud 【audience】接受者的url地址
  - exp 【expiration】 该jwt销毁的时间；unix时间戳
  - nbf  【not before】 该jwt的使用时间不能早于该时间；unix时间戳
  - iat   【issued at】 该jwt的发布时间；unix 时间戳
  - jti    【JWT ID】 该jwt的唯一ID编号
- Public Claims
- Private Claims
- signture 这一部分加密

Oauth2

![Oauth2](./img/OAuth2.png)

Code 换取accessToken只能换一次,accessToken可以使用多次

SpringSession

装饰者模式,spsession 在filter里面把原始的request包装成自己的request,后续的request.getsession都是从包装类里面获取,包装类重写了这些方法,变成了在redis中操作session
