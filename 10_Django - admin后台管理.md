# Django - admin后台管理

### 1、超级用户与用户

在manage.py的同级目录下

```
python3 manage.py createssuperuser
```

进入127.0.0.1:8000/admin，可以对其进行用户的权限赋予与创建组

------

### 2、注册自定义模型类

在应用app中的admin.py中导入注册要管理的模型models类

如：admin.py

```
from . models import Book

admin.site.register(Book)	#调用方法进行模型类的注册
```

> 完成注册后刷新127.0.0.1:8000/admin即可观察到被注册的模型

------

### 3、模型管理器类

#### 1）定义模型管理器类

<应用app>/admin.py

```
class xxxManager(admin.ModelAdmin):
	...
```

#### 2）绑定注册模型管理器类和模型类

<应用app>/admin.py

```
from django.contrib import admin
from .models import YYY
admin.site.register(YYY,xxxManger)	#绑定YYY模型类与管理器类xxxManager
```

例子：

/bookstore/amdin.py

```
from django.contrib import admin
from .models import Book

class BookManager(admin.ModelAdmin):
	list_display = ['id','title','pub','price']
	
admin.site.register(Book,BookManager)
```

#### 3)模型管理器类的方法

| 方法名             | 作用                                 |
| ------------------ | ------------------------------------ |
| list_display       | 列表显示字段列的名称                 |
| list_display_links | 控制list_display中的字段进入修改页面 |
| list_filter        | 添加过滤器                           |
| search_fields      | 添加搜索框[模糊查询]                 |
| list_editable      | 添加可在列表页编辑的字段             |

