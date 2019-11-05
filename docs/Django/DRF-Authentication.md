# DRF-Authentication

RESTful设计规范中要求使用OAuth2框架进行认证。

指定认证后端:

[Django AUTHENTICATION_BACKENDS](https://www.cnblogs.com/robinunix/p/7988259.html)

在djangorestframework中，有三个认证类，它们都继承自BaseAuthentication类：

1. BasicAuthenticationn

   功能太弱仅用于开发

2. SessionAuthentication

   - 在authenticate()方法中从request获取user并返回，需要借助cookie进行认证，常用于浏览器的请求验证，但是在前后端分离之后，很少使用。

3. TokenAuthentication

   djangorestframework在APIView中定义perform_authentication()方法，其会遍历所有的认证类进行验证，验证成功之后返回（user, token）

   - 使用时需要先注册APP

     ```python
     INSTALLED_APPS = (
         ...
         'rest_framework.authtoken'
     )
     ```

     其会在数据库中生成一张表：authtoken_token，包含三个字段，key, created, user_id。其中user_id是一个外键，其指向用户表中的用户id。

     为某个View开启TokenAuthentication或者在settibfs.py中的REST_FRAMEWORK中配置全局的认证：

     ```python
     # 配置全局
     DEFAULT_AUTHENTICATION_CLASSES = ('rest_framework.authentication.TokenAuthentication', )	
     # 配置某个具体的View类
     authentication_classes = xxx
     ```

     配置获取Token的URL:

     ```python
     from rest_framework.authtoken import views
     urlpatterns += [
         url(r'^api-token-auth/', views.obtain_auth_token)
     ]
     ```

     在用户注册和登录时，为用户创建令牌：

     ```python
     from rest_framework.authtoken.models import Token
     
     token = Token.objects.create(user=...)
     
     ```

     把生成的Token放到HTTP response中的Authentication头部中，返回给前端：

     ```python
     Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
     ```

     在进行验证时，调用authenticate()方法，并返回(token.user, token)

     这里提供的TokenAuthentication有两个缺点：

     1. Token会存放在一个服务器的数据库中，对于分布式的服务器，需要对这个数据进行同步
     2. token没有设置过期时间，因此存在很大的风险。

和两个和认证相关的中间件:

1. AuthenticationMiddleware

2. SessionMiddleware

   定义了process_request和process_response方法，这个中间件会在request进入视图函数之前和返回response之前被调用。

   其中process_request()通过cookie获取到session_key，并且使用该key从缓存（或者数据库）获取到session放到request.session属性中。



