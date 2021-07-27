# Django - 内建用户

更简单的实现登录与校验和退出的操作

详细文件链接：

[https://doce.djangoproject.com/en/2.2/topics/auth]: 

### 1、基本字段

```
from django.contrib.auth.models import User
```

| 类属性       | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| username     | 用户名                                                       |
| password     | 密码                                                         |
| email        | 邮箱                                                         |
| first_name   | 名                                                           |
| last_name    | 姓                                                           |
| is_superuser | 是否是管理员账号（/admin）                                   |
| is_staff     | 是否可以访问admin管理界面                                    |
| is_active    | 是否是活跃用户，默认True。一般不删除用户，而是将用户的is_active设置False |
| last_login   | 上次登录时间                                                 |
| date_joined  | 用户创建时间                                                 |



### 2、创建用户

```
#创建普通用户
from django.contrib.auth.models import User
user = User.object.create_user(username = '用户名',password = '密码'.email = '邮箱',...)
#必填项-username和password
```

```
#创建超级用户
from django.contrib.auth.models import User
user = User.object.create_superuser(username = '用户名',password = '密码'.email = '邮箱',...)
```



### 3、删除用户

使用伪删除

```
from django.contrib.auth.models import User
try:
	user = User.objects.get(username='xxx')
	user.is_active = False	#变为不活跃用户
	user.save()
	print("删除普通用户成功")
except:
	print("删除普通用户失败")
```



### 4、密码校验

```
from django.contrib.auth import authenticate
user = authenticate(username = username,password = password)

#如果用户名密码校验成功返回对应的user对象，否则返回None
```



### 5、修改密码

```
from django.contrib.auth.models import User

try:
	user = User.objects.get(username='xxx')
	user。set_password('123456')
	user.save()
	print("修改密码成功")
except:
	print("修改密码失败")
```



### 6、登录状态的保持

```
#先校验后保存状态
from django.contrib.auth import authenticate
from django.contrib.auth.models import User

from django.contrib.auth import login	#状态保持方法

def login_view(request):
	username = request.GET.get('username')
	password = request.GET.get('password')
	try:
		user = User.objects.get(username='xxx')
		user = authenticate(username = username,password = password)
		login(request,user)
		print("登录状态的保持成功")
	except:
		print("登录状态的保持失败")
```



### 7、登录状态的校验

```
from django.contrib.auth.decorators import login_required

@login_required	#登录状态的校验的装饰器
def index_view(request):
	#当前视图必须为登录状态下才可以访问
	#当前登录用户可通过request.user获取
	login_user = request.user
	...
```



### 8、登录状态的取消

```
from django.contrib.auth import logout
def logout_view(request):
	logout(request)
```





例子：

views.py

```
from django.http import HttpResponseRedirect, HttpResponse
from django.shortcuts import render

from django.contrib.auth.models import User
from django.contrib.auth import logout,authenticate,login
from django.contrib.auth.decorators import login_required

def register_view(request):
	#注册
	if request.method =='GET':
		return render(request,'register.html')
	elif request.method =='POST':
		username =request.POST['username']
		password_1 =request.POST['password_1']
		password_2 =request.POST['password_2']		
		if password_1 !=password_2:
			return HttpResponse('---两次密码输入不一致---')
			
		user = User.objects.create_user(username = username,password = password_1)
		
		return HttpResponseRedirect('/login')
		
def login_view(request):
	#登录
	if request.method =='GET':
		return render(request,'login.html')
	elif request.method =='POST':
		username =request.POST['username']
		password =request.POST['password']
		
		user = authenticate(username = username,password = password)
		if not user:
			return HttpResponse('---用户名或密码错误---')
		else:
			#校验成功
			#记录会话状态
			login(request,user)
			return HttpResponseRedirect('/index')
			
def logout_view(request):
	#登出
	logout(request)
	return HttpResponse('---已退出---')
	
	
@login_required
def index_view(request):
	#首页，必须登录参能访问，未登录跳转至setting.LOGIN_URL
	user = reuqest.user
	return HttpResponse('欢迎%s来到 测试内部验证首页'%(user.username))
```



setting.py

```
...

LOGIN_URL = '/login'
```



urls.py

```
...
urlpatterns = [
	path('admin/',admin.site,urls),
	path('login',views.login_view),
	path('register',views.register_view),
	path('index',views.index_view),
	path('logout',views.logout_view),
]
```

