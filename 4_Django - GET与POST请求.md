# GET与POST请求

基本样式

views.py

```
from django.http impoet HttpResponse
def test_get_post(request):
	if request.method == 'GET':
		pass
	esif request.method == 'POST':
		pass
	else:
		pass
	return HttpResponse('--test get post is ok--')
```

------

### 1、GET请求

GET中查询语句：

```
request.GET							#获取GET内容，不输入会报错
request.GET[ <参数名> ]				#返回的是QueryDict对象
request.GET.getlist( <参数名> )		#
request.GET.get( <参数名> , <初始值> )	#获取GET内容，但不输入不会报错
```

例子：

url:128.0.0.1:8000/tset?a=10&a=100

views.py

```
from django.http impoet HttpResponse
def test_get_post(request):
	if request.method == 'GET':
		request.GET						# QueryDict:{a:['10','100']}
		request.GET[ 'a' ]				 # 100
		request.GET.getlist( 'a' )	      # ['10', '100']
		request.GET.get( 'c' , 'no c' )   # no c
	esif request.method == 'POST':
		pass
	else:
		pass
	return HttpResponse('--test get post is ok--')
```

urls.py

```
from django.urls import path
from . import views
urlpatterns = [	
	path('admin/', admin.site.urls),	
	path('test', views.test_get_post)
]
```

------

### 2、POST请求

用于服务器提交大量或者隐私数据

获取数据语句：

```
request.POST							#获取GET内容，不输入会报错
request.POST[ <参数名> ]				#返回的是QueryDict对象
request.POST.getlist( <参数名> )		#
request.GET.get( <参数名> , <初始值> )	#获取GET内容，但不输入不会报错
```

例子：

html

```
<form method='post' action='/test_get_post'>
	用户名：<input type='text' name='umane'>
	<import type='submit' value='提交'>
</form>
```

views.py

```
from django.http impoet HttpResponse
def test_get_post(request):
	if request.method == 'GET':
		pass
	esif request.method == 'POST':
		print('uname is ',request.POST['uname'])	#取得表单中uname数据
		return HttpResponse('--post is ok--')
	else:
		pass
	return HttpResponse('--test get post is ok--')
```

> Django中定义了网络攻击相关的验证，csrf——目前请关闭：setting.py中注释下面代码
>
> ```
> MIDDLEWARE = [
> 	...
> 	#'django.middleware.csrf.CsrfViewMiddleware'.
> 	...
> ]
> ```

