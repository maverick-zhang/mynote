 

# flask-RESTful

1. CBV

   继承Resouce类，可以自定义get，post，delete，put, patch等方法。



2. 序列化

   - 使用marshal()函数，两个参数：

     ​	data:即为需要进行序列化的model对象或者字典

     ​	fields:序列化的字段，为字典：

     ​	其中fields.String和fields.Float为flask_restful中提供的类，默认情况下，映射关系为字段名为序列化数据的字段名，也可以自己指定如:  "alias_name":fields.String(attribute="property_name")， 每一个字段可以提供一个默认值(Integer字段提供了默认值0)

   ```python
   goods_fields = {
       "name": fields.String,
       "price": fields.Float(default=0.0),
   }
   ```

   - fields的嵌套

     可以直接提供python的字典对象进行序列化，如果在字典中嵌套model对象或者另外一个字典结构，则需要引入fields.Nested(another_fields)，another_fields即为被嵌套结构的序列化字段。

     fields的结构和json的结构一致，如果需要加入[]结构，则只需要引入fields.List字段，如：

     

   - 使用@marshal_with(fields)对CBV中的post等方法进行装饰

     fields参数中的字段可以比实际要序列化的字段多或者少，如果实际序列化的内容字段不存在则不会报错，显示内容为null，多出来的字段不会显示。因此最终显示的数据以提供的fields为准。

     

     被装饰的函数的返回值为需要被序列化的model对象，

     ```python
     goods_fields = {
         "name": fields.String,
         "price": fields.Float,
     }
     
     @marshal_with(goods_fields):
        def post(self):
         	...
             return goods
     ```

     

     

     对于嵌套的结构，则需要：

     ```python
     goods_fields = {
         "name": fields.String,
         "price": fields.Float,
     }
     single_goods_fields = {
         "msg": fields.String,
         "status": fields.Integer,
         "data":fields.Nested(goods_fields)
     }
     
     @marshal_with(single_goods_fields):
        def post(self):
         	...
             data = {
                 "msg": "ok",
                 "status":200,
                 "data":goods,
             }
             return data
     ```

     返回多个序列化的嵌套结构，则：

     ```python
     multi_goods_fields = {
         "msg": fields.String,
         "status": fields.Integer,
         "data":fields.List( fields.Nested( goods_fields ) )
         
     @marshal_with(multi_goods_fields)
      def get(self):
     	goods = Goods.query.all()
     	 data = {
                 "msg": "ok",
                 "status":200,
                 "data":goods,
             }
        	 return data
     
     ```



### 自定义字段：

​	需要继承fields模块中的Raw类，并定义formate()和output()函数，在进行序列化是先调用output()获取到数据，然后调用formate对数据进行格式化，最终返回value，因此formate()方法是可选的。

