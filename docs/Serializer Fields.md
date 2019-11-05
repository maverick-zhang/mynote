# 特殊Serializer Fields

### SerializerMethodField

- 必须是read_only，其作用在于自定义字段，并且定义函数get_<field_name>(self, obj)，在序列化时将会自动调用该函数，并把函数的返回值作为该字段的value，在fields中添加该字段后即可把想要的数据序列化传递给前端。obj为序列化的实例。

  ```python
  alipay_url = serializers.SerializerMethodField(read_only=True)
  def get_alipay_url(self, obj):    
      alipay = AliPay(        
          appid=""
          app_notify_url="http://IP_ADDR:PORT/alipay/return/",                           					q						  app_private_key_path=PRIVATE_KEY_PATH,        																							  alipay_public_key_path=ALIPAY_PUBLIC_KEY_PATH,                                                                                    return_url="http://IP_ADDR:PORT/alipay/return/"  
          )   
  url = alipay.direct_pay(      
	 		 subject=obj.order_sn,        
  		out_trade_no=obj.order_sn,        
  		total_amount=obj.order_amount    ) 
  re_url = "https://openapi.alipaydev.com/gateway.do?{data}".format(data=url)  
   return re_url
  ```

### ImageField

- 如果在序列化时出现嵌套的序列化，内层的序列化对象如果存在ImageField字段时，需要在序列化通过context={"request": self.context["request"]}把request传入到context中，这样序列化之后的ImageField会自动添加上域名，这么做是因为ImageField通常只保存media的路径。

  ```python
  goods_ins_dict = GoodsSerializer(goods_ins, many=False, context={"request":                         						               												   self.context["request"]}).data
  ```

### HiddenField和CurrentUserDefault

- 将两者结合可以，在序列化需要用户字段时，可以从前段传入的Token等信息自动获得User，并且作为默认值传给HiddenField，即不需要把用户字段进行回传。

  ```python
  user = serializers.HiddenField(default=serializers.CurrentUserDefault())
  ```

