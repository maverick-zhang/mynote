# 跨域请求CROS

### django-cros-headers

CROS: cross-origin resource sharing

由于前后端分离，因此在开发过程中如果需要前后端在本地进行联调，由于同时启用了不同的服务器端口，那么前后端数据沟通就会产生跨域问题。可以通过使用此中间间解决。

安装

```shell
pip install django-cors-headers
```

注册

```python
INSTALLED_APPS = [
    ...
    'corsheaders',
    ...
]
```

添加中间件：

注意该中间件应该放在任何可能产生response的中间件之前（最好放在第一位）

```python
MIDDLEWARE = [  # Or MIDDLEWARE_CLASSES on Django < 1.10
    ...
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    ...
]
```

`CORS_ORIGIN_ALLOW_ALL= True`允许所有站点的跨域访问，默认为False

或者设置白名单：

```python
CORS_ORIGIN_WHITELIST = [
    "http://127.0.0.1:3000"   # Vue的服务器端口
]
```

