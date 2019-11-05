# DRF缓存和django-redis

### 使用drf-extensions包进行缓存

[官方文档](http://chibisov.github.io/drf-extensions/docs/)

在使用到list和retrieve的ViewSet中，添加对CacheResponseMixin的继承

```python
from myapps.serializers import UserSerializer
from rest_framework_extensions.cache.mixins import CacheResponseMixin

class UserViewSet(CacheResponseMixin, viewsets.ModelViewSet):
    serializer_class = UserSerializer
```

在settings.py中添加过期时间设置：

```python
REST_FRAMEWORK_EXTENSIONS = {
    'DEFAULT_CACHE_RESPONSE_TIMEOUT': 60 * 15
}
```

### 在django中配置redis

使用第三方包diango-redis

修改django的缓存配置

```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```

- url

  - redis://: 普通的 TCP 套接字连接

  - rediss://: SSL 包裹的 TCP 套接字连接

  - unix://: Unix 域套接字连接

  - ```python
    redis://[:password]@localhost:6379/0
    rediss://[:password]@localhost:6379/0
    unix://[:password]@/path/to/socket.sock?db=0
    ```

[官方文档](https://django-redis-chs.readthedocs.io/zh_CN/latest/)

