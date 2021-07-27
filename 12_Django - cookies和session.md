# Django - cookies和session

###  1、会话

定义：从浏览器访问一个网站到关闭浏览器结束此次访问称为会话Cookies和session就是为了保持这个会话状态诞生的两个存储技术

------

### 2、Cookies

cookies是保存再客户端浏览器上的存储空间

特点：

- cookies在浏览器上以字典（键 - 值）的形式进行存储的，键和值都是以ASCII字符串形式存储
- 存储的数据带有生命周期
- cookies中的数据是按域存储隔离的，不同的域之间无法访问
- **cookies的内部的数据会在每次访问次网址时都携带到服务器端，如果cookies过大会降低影响速度**

#### 1）cookies的使用 - 存储

```
HttpResponse.set_cookie(key,value = '',max_age = None,expires = None)

-key:cookie的名字
-value:cookie的值
-max_age:cookie存活时间，秒为单位
-expires:具体过期时间
-当不指定max_age和expires时，关闭浏览器时此数据失效
```

例子：

views.py

```
def set_cookies(request):
	resp = HttpReaponse('set cookies is ok')
	#--------#
	resp.set_cookie('uname','hou',500)	#普通视图下添加set_cookie方法保存会话
	#--------#
	return resp
```

#### 2）cookies的使用 - 删除和获取

删除cookies

```
HttpResponse.delete_cookie(key)
```

获取cookies

```
value = request.COOKIES.get('cookies名','默认值')
```

### 3、Session

定义：在服务器上开辟一段空间用于保留浏览器和服务器交互时重要数据

> 使用session需要在浏览器客户端启动cookie，且在cookie中存储sessionid
>
> 每个客户端都可以在服务器端有一个独立的Session
>
> 注意：不同的请求者之间不贵共享这个数据，与请求者一一对应

#### 1）初始配置

settings.py

```
...
INSTALLED_APPS = [
	...
	#启用sessions应用
	'django.contrib.sessions',
	...
]
...
MIDDLEWARE = [
	...
	'django.contrib.sessions.middleware.SessionMiddleware',
	...
]
```

> 默认存在，如果不存在则自行添加

#### 2）session的使用

保存session的值到服务器

```
requeset.session['KEY'] = VALUE
```

获取session的值

```
value = request.session['KEY']
value = request.session.get('KEY',默认值)
```

删除session

del request.session['KEY']

例子：

views.py

```
#写入session
def set_session(request):
	request.session['uname']='houzi'
	return HttpResponse()
	
#获取session
def get_session(request):
	value = request.session.get['uname']
	return HttpResponse('session value is %s'%(value))
```

#### 3）session问题

django_session表是单表设计；且该表数据量持续增持[浏览器故意删除或过期数据未删除]

**每晚执行 python3 manage.py clearsessions [该命令可删除已过期的数据]**

------

### 4、Cookies和session对比

| 种类    | 存储   | 安全性     |
| ------- | ------ | ---------- |
| Cookies | 浏览器 | 相对不安全 |
| Session | 服务器 | 相对安全   |

