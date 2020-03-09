## django xadmin

------

django admin默认提供以下表: 组表，用户表，权限表，组权限表， 用户权限表

可以给任一model注册到后头管理，这样就可以直接在后台中对该model进行CRUD

创建自己的admin model管理自己数据

```python
from django.contrib import admin
class UserAdmin(admin.ModelAdmin)
		pass
	
	admin.site.register(UserModel, UserAdmin)
```

即可在admin后台中管理user类



### xadmin

基于django admin, 使用了bootstrap，使用方法和django admin类似

在app目录下创建adminx.py文件，xadmin会执行自动搜索

和django admin类似需要先创建admin类

```python
import xadmin
class MyXadmin:
		list_display = []  # 在后台管理系统中显示的字段
        search_filds = [] # 在后台管理中可以进行搜索的字段
        list_filter = [] # 设置过滤器字段，会根据字段的类型自动设置

xadmin.site.register(MyModel, MyXadmin)
```

基础配置

```python
import xadmin
from xadmin import views

class BaseSetting:
    enable_themes = True      # 开启主题
    use_bootswatch = True

class GlobalSettings:
    site_title = "xxx"      # 后台页面的标题
    site_footer = "xxx"    # 脚注
    menu_style = "accordion" # 侧边栏折叠
    apps_icons = {"appname":"icon"}  #设置app的显示图标
    
  # 注册配置
xadmin.site.register(views.BaseAdminView, BaseSetting)
 xadmin.site.register(views.CommAdminView, GlobalSettings)
```

inlineAdmin嵌套编辑

假设A是B的外键

```python
class AAdmin:
		model = A
		extra = 1
		
		class BAdmin:
				list_display = ["",""]
				inlines = [A,]
```

这样在编辑B的时候，就可以同时对A进行增删改查

但是要注意一个问题，这里的A，里面一定要写明model =A，不能像B写list_display = [""],否则会报错 'NoneType' object has no attribute '_meta'。extra：表示的是在编辑B的时候，展示几个A的可编辑框。不设置的话默认是3个，设置为0的话，则表示A编辑框是收起的。