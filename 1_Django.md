# Django

### 一、安装django

python版本-3.5.，3.6，3.7，3.8

------

###### Windos安装

Windos下打开cmd输入：

```
pip install django==2.2.12
```

测试是否安装成功

```
python
>>>import django
>>>django.get_vetsion()
```

------

###### Linux安装

Linux下打开终端输入：

```
sudo pip3 install django==2.2.12
```

测试是否安装成功

```
sudo pip3 freeze|grep -i 'Django'
```

------

### 二、创建Django项目

选择想创建的路径下打开cmd或终端

```
django-admin startproject <项目名>
```

------

### 三、启动服务与关闭服务

###### 启动调试服务（并非上线服务）

进入项目根目录下打开cmd或终端

```
python3 manage.py runserver <ip地址与端口号>
```

###### 关闭服务

在runserver启动终端下

```
Ctrl+c
```

或者在其他终端下

Windos

```
netstat -ano |findstr <端口号>
testkkill /f /t /im <进程ID>
```

Linux

```
sudo lsof -i:8000   #查询Django进程ID（带LISTEN）
kill -9 <进程ID>
```

------

### 四、URL

结构组成

```
protocol://hostname[:path]/path[?query][#fragment]
协议://域名[:端口号]/路由[?查询][#锚点]
```

###### urls.py

主路由-urls.py样例

```
from django.urls import path
from . import views
urlpatterns = [
	path('admin/', admin.site.urls),
	path('page/2003/', views.page_2003),
	path('page/2004/', views.page_2004),
]
```

###### views.py

如果没有则自己生成，在项目同名文件夹下，与urls.py同级

视图函数-views.py样例

```
from django.http import HttpResponse
def page1_view(request):			#方法名可自己定义，参数第一个必须为request
	html = "<h1>hello world！</h1>"
	return HttpResponse(html)		#必须return一个HttpResponse对象
```

 

### 五、创建新项目时setting.py配置

setting.py

```
...
MIDDLEWARE = [
	...
	# 'django.middleware.csrf.CsrfViewMiddleware',	#解决403问题
	...
	]
...
TEMPLATES = [
	...
    'DIRS': [os.path.join(BASE_DIR,'template')],	#模板路径
    ...
    ]
...
LANGUAGE_CODE = 'zh-Hans'	#语言
TIME_ZONE = 'Asia/shanghai'	#时区
...
STATICFILES_DIRS = (os.path.join(BASE_DIR,"static"),)	#静态文件路径
	
```

