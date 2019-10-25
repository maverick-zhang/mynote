# JWT用户认证

JSON Web Token

由三部分组成：

1. Header

   ```json
   {
     "alg": "HS256",
     "typ": "JWT"
   }
   ```

包含使用的算法和JWT声明，使用Base64编码

2. Playload

   ```json
   {
       "iss": "lion1ou JWT",
       "iat": 1441593502,
       "exp": 1441594722,
       "aud": "www.example.com",
       "sub": "lion1ou@163.com"
   }
   ```

   iss:签发者，exp:过期时间， sub:面向的用户， aud:接收方， iat:签发时间。

   Base64编码

3. Siganature

   使用header中的算法以及提供的私有秘钥对header和playload进行摘要化，也即是生成Token。

   生成的Token以JSON返回给前段，前段的请求中需要把Token放到http的Authentication 头部中



### JWT使用

1. 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。
2. 后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT。形成的JWT就是一个形同lll.zzz.xxx的字符串。
3. 后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上，退出登录时前端删除保存的JWT即可。
4. 前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题)
5. 后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己（可选）。
6. 验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。



### 第三方包：Django REST framework JWT

[官方文档](https://jpadilla.github.io/django-rest-framework-jwt/)

