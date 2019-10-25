# djangorestframework——View

- ### View层级结构

  - #### **第一层django自带的View类**

    

    - #### 第二层APIView(View)

      1. 加入renderer_classes，parser_classes，authentication_classes，throttle_classes，permission_classes等属性，并定义了获取这些属性的方法
    
      2. 并重写了as_view()方法，在返回的结果中进行了csrf_exempt
      
      3.  重写了dispatch()方法，对不同的request.method分发给对应的函数（get, post, delete, update, put等）进行处理
      
      4. 定义了initial_request()方法，返回包含request，渲染类，解析类，认证类，节流类的集合
      
      5. 定义initial()方法，该方法会进行认证，权限认证，以及节流检查
      
      6. 直接继承使用该View类，需要自己定义get, post, update, delete等等http query method，完成数据库查询，序列化反序列化等等操作，并且在方法中返回Response或者django自带的HttpResponse/JsonResponse
      
         
      
      - #### 第三层GenericAPIView(APIView)
      
        其在APIView的基础上加入了queryset，serializer_class,  paginator_class，filter_backends(需要自己定义), lookup_field等属性，并且定义了相关的方法，从这些属性中获取到具体的查询集以及筛选，序列化和反序列化，分页类等。可以自己重写get_queryset()方法，从前段传入的过滤信息进行特定的筛选，并返回queryset，如果重写此方法，则不再需要设置queryset属性。其他的属性也类似，可以自己进行定制。
        
        
        
        通过继承GenericAPIView和CRUDL父类，即得到了以下：
        
        ```python
        CreateAPIView		#定义post()，在返回时调用继承的create()方法
        ListAPIView				#定义get()，返回时调用继承的list()方法。以下类类似。
        RetrieveAPIView
        DestroyAPIView
        UpdateAPIView
         # 多方法继承
        ListCreateAPIView
        RetrieveUpdateAPIView
        RetrieveDestroyAPIView
        RetrieveUpdateDestroyAPIView
        ```
        
        
        
        #### 第四层GenericViewSet(GenericAPIView,  ViewSetMixin）
        
        1. ViewSetMixin重写了as_view()，和initial_request()方法 ，对request method和action进行了映射，并且通过get_extra_action()方法获取通过装饰器（@action）自定义的action，并且进行了映射。
        
        2. @action需要指定映射的request method
        
           detail表示是collection还是单个对象
        
           ```python
            @action(detail=True, methods=['post'], permission_classes=[IsAdminOrIsSelf])
           ```
        
        3. GenericViewSet实际上没有添加任何内容，该类的所有新特性都体现在了ViewSetMixin中
        
        4. 其目的是 “to combine the logic for a set of related views in a single class”（官方原文），可以只写一次queryse，并且给不同的CRUDL使用
        
         5. 在对该视图类进行注册时，要使用router class进行注册，简化大量url的注册
        
         官方文档
        
         There are two main advantages of using a `ViewSet` class over using a `View` class.
        
           - Repeated logic can be combined into a single class. In the above example, we only need to specify the `queryset` once, and it'll be used across multiple views.
           - By using routers, we no longer need to deal with wiring up the URL conf ourselves.
        
         
        
           通过对GenericViewSet和CRUDL父类的继承，得到以下的类
        
        ```python
           ReadOnlyModelViewSet  #包含list和retreieve
           ModelViewSet  #包含所有
        ```
        
           



### mixin模块(封装CRUDL)，在这里称其为CRUDL父类

- ListModelMixin(objext)

  定义了list()方法，获取一组序列化对象，返回Response([serializer.data], [status.XXX])

  

- CreateModelMixin(object)

  定义了create(),和perform_create()，前者获取序列化对象，并进行is_valid验证，返回Response([serializer.data], [status.XXX]); 后者只是执行了save()操作

  

- RetrieveModelMixin(object)

  定义了retreieve()方法，获得一个序列化对象, 返回Response([serializer.data], [status.XXX])

  

- UpdateModelMixin(object)

  定义了update()和partial_updata()方法，以及perform_update()，前者返回Response([serializer.data], [status.XXX])

  

- DestroyModelMixin(object)

  定义了destroy()和perform_destroy()方法，前者返回Response([serializer.data], [status.XXX])

​          

