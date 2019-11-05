# Serializer relations

### 一对多（主为一多为从）：

在从的model类定义中，外键字段的加入关键字参数related_name="xxx"，如果不指定该参数，就需要使用ORM自动生成的<model_name>_set：

```python
class Album(models.Model):
    album_name = models.CharField(max_length=100)
    artist = models.CharField(max_length=100)

class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    order = models.IntegerField()
    title = models.CharField(max_length=100)
    duration = models.IntegerField()
    class Meta:
        unique_together = ['album', 'order']
        ordering = ['order']

     def __str__(self):
        return '%d: %s' % (self.order, self.title)
```

则在序列化时，先对从进行序列化，把从的序列化类作为主序列化类的一个字段（因为Serializer继承自Field）,注意要在关键字参数上加上many=True，字段名即为在前面model中的related_name，把该字段添加到fields中，即可完成一对多的序列化:

```python
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = "__all__"

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```

这些也适用于一对一，只不过不需要many=Ture，其余操作相同，从为声明外键的model。



这样会获得从model的所有实例的所有字段的信息，以下可以只获得部分信息:

但是需要指定queryset属性

- StringRelatedField	获得从model中的实例\__str__信息

- PrimaryKeyRelatedField   获得从model的实例id

- HyperlinkedRelatedField   自动生成从model实例的url，默认的lookup_field为pk

- HyperlinkedIdentityField   生成从model实例的列表，并把该列表的url信息加入到主序列化中

- SlugRelatedField    获得从model实例的部分字段（添加关键字参数slug_field="field_name"）

  

### 多对一（一为主多为从）：

Django的ORM已经把主作为一个外键字段包含在了从model中，因此可以直接在从类序列化该字段。



### 自定义关系(relations):

需要自定义关联字段类，继承自RelatedField，并且重写to_representation(self, value)方法，返回值作为该字段的value，value可以是某个model_instance。也可以重写get_queryset(self)方法。

