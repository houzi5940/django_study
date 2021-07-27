# Django - 路由配置



### 1、基本使用

------

##### 1、path()

```
from django.urls import path	#导入

path(route, views, name=None)	#语法
参数意义：
	route：字符串类型，匹配的请求路径
	views：指定路径所对应的视图处理函数名称
	name：为地址起别名，在模板中地址反向解析时使用
```

------

##### 2、path转换器

作用：若转换器类型匹配到对应类型的数据，则将数据按照**关键字传参**的方式传递给视图

```
path('page/<int:page>', views.xxx)	#例子
```

数据类型

| 转换器类型 | 作用                   |
| :--------- | :--------------------- |
| str        | 非 '/' 字符串          |
| int        | 整形                   |
| path       | 匹配非空字段，包括 '/' |
| slug       | 匹配任意ASCII          |

例子：

urls.py

```urls.py
from django.urls import path
from . import views
urlpatterns = [
	path('page/2003/', views.page_2003),
	path('page/2004/', views.page_2004),
	path('page/<int:num>/', views.page_n),
]
```

views.py

```views.py
from django.http import HttpResponse

def page_2003(request):
	html = 'this is page_2003'
	return HttpResponse(html)

def page_2004(request):
	html = 'this is page_2004'
	return HttpResponse(html)
	
def page_n(request,num):
	html = 'this is ' + num
	return HttpResponse(html)
	
```

------

##### 3、re_path()

使用正则表达式进行**精确匹配**

```
from django.urls import re_path
re_path(reg, view ,name = xxx)

reg：(?P<name>pattern)		#用法
```

例子：

匹配两位数

urls.py

```urls.py
from django.urls import re_path
from . import views

urlpatterns = [
	# 变量名为x,y   两位数以内
	re_path(r'^(?P<x>\d{1,2})/(?P<y>\d{1,2})$', views.cal_view)
]
```

views.py

```views.py
from django.http import  HttpResponse
def cal_view(request,x,y):
	html ='x为：%n /ny为 %n ' %(x,y)
	return HttpResponse(html)
```



### 2、进阶使用

------

##### 1）url书写规范

```
"http://127.0.0.1:8000/test_url_result"	#绝对地址
"/test_url_result"						#带斜杠的相对地址
"test_url_result"						#不带斜杠的相对地址
```

结果：

```
初始地址为：http://127.0.0.1:8000/test

http://127.0.0.1:8000/test_url_result	#使用绝对地址跳转后的地址
http://127.0.0.1:8000/test_url_result	#使用带斜杠的相对地址跳转后的地址
http://127.0.0.1:8000/test/test_url_result	#使用不带斜杠
```



##### 2）url反向解析

用path定义的别名来动态查找相对应的路由

语法：

urls.py

```
path(route,view,name = '别名')
```

html

```
{% url '别名' %}
{% url '别名' '参数值1' '参数值2' %}
```

例子：

urls.py

```
...
path('test/<int:num>',views.test,name = 'test_url')
...
```

html

```
...
<body>
	...
	<a href= {% url 'test_url' num='10' %}></a>
	...
</body>
...
```

views.py

```
...
def test (request num):
	return render(request,'test.html')
...
```

------

##### 3）url反向解析2

在view视图中使用反向解析

语法：

```
from django.urls import reverse

reverse('别名'，args=[],kwargs={})
```

例子：

urls.py

```
...
path('test1',views.test1)
path('test/<int:num>',views.test,name = 'test_url')
...
```

views.py

```
...
def test1 (request):
	num = 100
	url = recerse('test_url',args[num])
	return HttpRserponseRedirect(url)	#页面跳转
def test (request num):
	return render(request,'test.html')
...
```



### 3、分布式路由

------

##### 1）创建应用

在项目文件夹中，与manage.py同级目录下打开终端/cmd

```
python3 manage.py startapp <应用名>
```

setting.py

```
...

INSTALLED_APPS = [
...
'<应用名>',
]
```

##### 2）分布式url

主项目的urls.py

```
from django.contrib import admin
from django.urls import path
from . import views

urlpatterns = [
	path('admin/',admin.site.urls),
	#http://127.0.0.1:8000/<应用名>/<应用名>
	path('<应用名>', include('<应用名>.urls'))
]
```

应用中的urls.py(若没有则自己创建)

urls.py

```
from django.urls import path
from . import views

urlpatterns = [
	#http://127.0.0.1:8000/<应用名>/<应用名>
	path('<应用名>',views)
]
```

##### 3）应用中的模板

在应用文件夹下创建templates

在应用下的templates下，创建应用同名的文件夹

将模板放入这个同名文件夹下

调用模板

应用中的views.py

```
def test(request):
	return render(request,'<应用名>/模板名')
```

