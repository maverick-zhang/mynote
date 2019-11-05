# serializers

序列化和反序列化：序列化即是把对象的数据转化为json格式，而反序列化即是把json的数据保存为model对象(以及对象的保存：保存到数据库)

序列化的流程：传入的data首先保存在self.initial_data中(通过property装饰，其实是self._initial_data，validated_data也是如此)，序列化之前调用is_valid()对传入的数据进行验证（包括字段验证，对象验证，和可复用的验证器类，根据不同的需求进行自定义），验证不通过抛出serializers.ValidationError("error_msg")，会被自动捕获，返回给前端{"field_name": ["error_msg"]}以及HTTP_400，验证通过则数据会保存在self.validated_data中，然后再传入到create()或者update()方法中(这两个方法不一定不是必须，可以通过重写save()方法执行非数据库的保存更新等操作)

## Serializer(instance, data=, partial=, many=)

instance为序列化的实例，若没有提供则位None, data为需要序列化的传入数据，patial为False时可以只提供部分的的序列化字段数据进行部分更新，many为Ture时表示同时序列化多个实例，传入的数据k可以是一组queryset。

1. 需要自己定义全部的序列化字段，如果需要保存到数据库，需要重写create()或者update()方法，手动把数据传入到model的实例当中，通过调用save()方法保存一个序列化的实例。可以根据需求重写save()方法，例如调用save()，但是不保存序列化的实例，或者完成一些其他任务，比如发送邮件和短信等等。

2. 这个类提供了序列化器的最基础功能，把传入的字典映射到自己定义的各个字段上。

3. 在传入需要序列化的数据之后，在调用is_valid()方法之前，数据会存放在self.initial_data中

4. 在进行序列化之前，需要调用is_valid()，该方法会执行validate()对各个字段进行正确性验证，需要重写该放阿飞确保能够进行正确的验证，方法进行验证，验证通过之后，会把验证通过的数据存放到self.validated_data中

5. self.validated_data中包含了通过验证后的数据，同样是一个字典。

6. 字段验证

   #### 单个字段验证

   - 定义validate_<field_name>方法，函数传入的数据为<field_name>对应的字段value,，验证通过返回该数据，不通过抛出serializers.ValidationError，如果该字段中关键字参数required=False，则当不包含该字段时不会进行该验证。

     ```python
     from rest_framework import serializers
     
     class BlogPostSerializer(serializers.Serializer):
         title = serializers.CharField(max_length=100)
         content = serializers.CharField()
     
         def validate_title(self, value):
             """
             Check that the blog post is about Django.
             """
             if 'django' not in value.lower():
                 raise serializers.ValidationError("Blog post is not about Django")
             return value
     ```

   #### 对象验证

   -  也可以定义validate()方法进行所有字段的统一验证，传入的数据为字段字典{'field_name': value}，同样验证不通过直接抛出serializers.ValidationError，而所有验证通过时，把传入函数的数据返回。这种验证只局限于对该序列化类。

     ```python
     from rest_framework import serializers
     
     class EventSerializer(serializers.Serializer):
         description = serializers.CharField(max_length=100)
         start = serializers.DateTimeField()
         finish = serializers.DateTimeField()
     
         def validate(self, data):
             """
             Check that the start is before the stop.
             """
             if data['start'] > data['finish']:
                 raise serializers.ValidationError("finish must occur after start")
             return data
     ```

   #### 验证器

   1. 验证器函数

   - 验证器可以是的单个字段的验证器，方式为定义验证函数validator_func，然后在字段实例上声明validators=[validator_func]。

   2. 验证器类Validator

   - 在class Meta下声明validators=SomeTypeValitator(**kwargs)

   ### 嵌套序列化：

   序列化类其实也是继承自Filed类，因此可以作为序列化的字段

   ```python
   class UserSerializer(serializers.Serializer):
       email = serializers.EmailField()
       username = serializers.CharField(max_length=100)    
   
   class CommentSerializer(serializers.Serializer):
       user = UserSerializer()
       content = serializers.CharField(max_length=200)
       created = serializers.DateTimeField()
   ```

   传入data时可以直接传入嵌套的dict数据，嵌套的关系在保存在validated_data中也是嵌套的，ModelSerializer不再支持默认的create()和update()方法，需要自己明确定义，手动进行序列化。







## ModelSerializer

继承自Serializer，提供了到Model字段的自动映射，且实现了update()和create()方法。使用时需要在class Meta:中声明model和fields。

```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
```

可以指定fileds = "\__all__"序列化Model中的所有字段，也可以使用exlude=('filed_name', )来指定不序列化哪些字段，由于提供了自动映射，因此这里的字段名称必须和Model中的字段名称相同。



## Serializer fields

字段可选属性：

- read_only	默认False，不会参与到模型的类的创建写入更新等过程，仅是作为展示的数据。
- write_only   默认False，设置为Ture时，在返回给前段时不会序列化(前端只是作为展示，representation)，这样可以对字段进行一定的取舍，比如密码不应该再返回给前段，且一些没必要的字段如验证码，也不需要再返回给前段。如果设置为True，create和update要能够处理好该字段的序列化。
- required       默认为Ture，要求必须传递该字段
- validators    可以指定具体的验证函数或者验证器类
- error_message    错误码和错误信息的字典，如erro_message = {"required": "必须提供该字段的信息", }
- label    在HTML表单中的名称
- help_text    在HTML表单中的描述文字
- initial    提供给HTML表单中的初始值

包含的fileds种类(常用的)：

- BooleanField

- CharField

- IntergerField

- FloatField

- DateField

- DateTimeField

- SerializerMethodField   

  通过定义的函数获取字段的value，通过使用这种字段可以在序列化时进行一些操作。

  ```python
  from django.contrib.auth.models import User
  from django.utils.timezone import now
  from rest_framework import serializers
  
  class UserSerializer(serializers.ModelSerializer):
      days_since_joined = serializers.SerializerMethodField()
  
      class Meta:
          model = User
  
      def get_days_since_joined(self, obj):
          return (now() - obj.date_joined).day
  ```

[其他的见官方文档](https://q1mi.github.io/Django-REST-framework-documentation/api-guide/fields/)