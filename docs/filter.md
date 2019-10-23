# Filter

### DjangoFilterBackends

- 需要安装django-filter

  ```shell
  pip install django-filter
  ```

  在INSTALLED_APPS注册"django_filter"。配置通用的过滤：

  ```python
  REST_FRAMEWORK = {
      'DEFAULT_FILTER_BACKENDS': ('django_filters.rest_framework.DjangoFilterBackend',)
  }
  ```

  如果针对某个View类(需要继承自GenericAPIView)进行设置，则需要指定filter_backends属性和filter_fields属性，filter_fields属性只提供了简单的相等匹配：

  ```python
   filter_backends = (DjangoFilterBackend,)
   filter_fields = ("name",  "market_price")
  ```

  在请求url上加上query_parms：

  ```python
  http://127.0.0.1/goods/name=xxx&market_price=xxx
  ```

  但是以上的配置不够灵活，如果需要自定义筛选可以自定义filter类，其继承django_filter.

  ```python
  from django_filters.rest_framework import FilterSet, NumberFilter, CharFilter
  
  from goods.models import Goods
  
  
  class GoodsFilter(FilterSet):
      """
      商品的过滤
      """
      price_min = NumberFilter(field_name="shop_price", lookup_expr="gte")
      price_max = NumberFilter(field_name="shop_price", lookup_expr="lte")
      # icontaons代表模糊查询且不区分大小写，类似于SQL中的like
      name = CharFilter(field_name="name", lookup_expr="icontains")
  
      class Meta:
          model = Goods
          fields = ("price_min", "price_max", "name")
  ```

  lookup_exp参数是筛选的条件，包括：

  gte, lte, gt, lt, exact, iexact, contains, icontains
  
  field_name则是筛选的字段

  在View类中设置filter_class属性：
  
  ```python
  filter_class = GoodsFilter
  ```
  
  在url上添加query_parms，实现筛选：
  
  ```python
  http://127.0.0.1:8000/goods/?price_min=180&price_max=200&name=水果
  ```
  
  

​       

### SearchFilter

- 这是djangorestframework自带的Filter类

  直接在View类中声明以下属性，即可直接使用

  ```python
      filter_backends = (filters.SearchFilter,)
      search_fields = ('name', 'goods_brief', 'goods_desc')
  ```

  在url添加query_parms:

  ```python
  http://127.0.0.1:8000/goods/?search=xxx
  ```

  会在上述的name, goods_brief, goods_desc字段中进行搜索。

可以通过在search_fields前面添加各种字符来限制搜索行为。

- '^' 以指定内容开始.
- '=' 完全匹配
- '@' 全文搜索（目前只支持Django的MySQL后端）
- '$' 正则搜索

比如：

```python
search_fields = ('=name')
```





### OrderingFilter

- 同样是djangorestframework中自带的filter类

  在View类中生命以下属性：








