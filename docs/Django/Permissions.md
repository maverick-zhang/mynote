# Permissions

### 使用方法：

可以在seetings.py中设置全局的认证类

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    )
}
```

如果不指定，默认为AllowAny

或者在视图类上指定permission_classes(IsAuthenticated, xxx)，或者通过装饰器对**视图函数**进行装饰

```python
@api_view(['GET'])
@permission_classes((IsAuthenticated, ))
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
```



#### 对象级别的权限认证

重写get_object()方法，在其中显示的调用check_object_permission(request, obj)

### IsAuthenticated

- 基础权限认证，只要求是否是认证过的用户

### AlloAny

- 允许不受限制的访问

### IsAdminUser

- 判读user的is_staff字段是否为Ture

### IsAuthenticatedOrReadOnly

- 认证过的用户拥有所有的权限而没有认证的用户只能有读的权限。



### DjangoModelPermissions

- POST请求要求用户需要有添加权限
- PUT和PATCH要求用户对模型有更改的权限
- DELTE要求用户对模型具有删除的权限



[其余权限见官方文档](https://q1mi.github.io/Django-REST-framework-documentation/api-guide/permissions_zh/)



### 自定义权限

继承自permission.BasePermission

并且需要实现以下方法中的一个:

- `.has_permission(self, request, view)`
- `.has_object_permission(self, request, view, obj)`

验证成功需要返回True, 如果失败则需要返回False

```python
class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    对象级权限仅允许对象的所有者对其进行编辑
    假设模型实例具有`owner`属性。
    """

    def has_object_permission(self, request, view, obj):
        # 任何请求都允许读取权限，
        # 所以我们总是允许GET，HEAD或OPTIONS 请求.
        if request.method in permissions.SAFE_METHODS:
            return True

        # 示例必须要有一个名为`owner`的属性
        return obj.owner == request.user
```