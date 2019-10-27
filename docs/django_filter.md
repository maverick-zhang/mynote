# django_filter

1. 自定义FilterSet

   继承自FilterSet

   使用实例：

   ```python
   class ProductFilter(filters.FilterSet):
       min_price = filters.NumberFilter(field_name="price", lookup_expr='gte')
       max_price = filters.NumberFilter(field_name="price", lookup_expr='lte')
   
       class Meta:
           model = Product
           fields = ['category', 'in_stock', 'min_price', 'max_price']
   ```

   其中price, category, in_stock为Product中的字段，min_price和max_price为自定义的过滤字段。

   在进行过滤时，需要在包含该FilterSet的视图类的url中加上以这些字段为key的query_params

   如：http://localhost/goods/?category=xxx&in_stock=xxx&min_price=&max_price=

   不过滤某字段是，起对应的value为空。



2. 配置

   - 在INSTALLED——APPS中注册"django_filter"

   - 配置Filter_BACKEND

     - 可以在全局中配置默认值：

       ```python
       REST_FRAMEWORK = {
           'DEFAULT_FILTER_BACKENDS': (
               'django_filters.rest_framework.DjangoFilterBackend',
               ...
           ),
       }
       ```

     - 也可以在具体的view类中配置filter_backends = (xxx, xxx)

       ```python
       class GoodsListViewSet(GenericViewSet, ListModelMixin, RetrieveModelMixin):   
           queryset = Goods.objects.all()    serializer_class = GoodsSerializer    
           filter_backends = (SearchFilter, DjangoFilterBackend, OrderingFilter)
       ```

   - 在视图类中配置filter_class = xxx(第一步中自定义的FilterSet)