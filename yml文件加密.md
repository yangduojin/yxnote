# yml 配置文件加密

引入该依赖
``` 
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

引入这个依赖,下面的配置可以省略 就是简单配置一下前缀后缀，为了解析加密字符串用的

``` yml
jasypt:
  encryptor:
    property:
      prefix: ENC(
      suffix: )```
```

配置密钥,在yml(及其不推荐)中或者启动参数
``` yml
jasypt:
encryptor:
property:
prefix: ENC(
suffix: )
password: # 你的密钥
```

推荐 ``VM options:-Djasypt.encryptor.password=你的密钥``

获取加密的数据 
``` java
public class Jasypt {
    public static void main(String[] args) {
        BasicTextEncryptor textEncryptor = new BasicTextEncryptor();
        // 加密密钥
        textEncryptor.setPassword("sirwsl");
        // 要加密的数据（如数据库的用户名或密码）
        String username = textEncryptor.encrypt("root");
        String password = textEncryptor.encrypt("123");
        System.out.println("加密：username:" + username);
        System.out.println("加密：password:" + password);
    }
}
```
或者这样
``` java
@SpringBootTest
class JasyptTest {

    @Resource
    private StringEncryptor stringEncryptor;
    @Test
    public void test() {
        //加密
        String username = stringEncryptor.encrypt("root");
        System.out.println("加密username: " + username);

        String decUsername = stringEncryptor.decrypt(username);
        System.out.println("解密username: " + decUsername);

        //加密
        String password = stringEncryptor.encrypt("123456");
        System.out.println("password: " + password);
        String decPassword = stringEncryptor.decrypt(password);
        System.out.println("解密password: " + decPassword);}}
```
最后这样将加密字符串输入yml
``` yml
spring:
  application:
    name: demo-encryption
# 数据库驱动：
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/demo?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true
    password: ENC(Y8CJa4AfPV+/snhdJ6ADg0wWuNQIJ1v2UQuyejJm7lPE76jdbr2I82rMvLRX2sT9)
    username: ENC(cN5buxefuYaZBJ8/XCXWA3GAJdkPz0hBogIwG9uGjI8DH1v2oKLm1TQYD8aBhX9A)

#下面的配置可忽略
jasypt:
  encryptor:
    property:
      prefix: ENC(
      suffix: )
    #password: xxx
```