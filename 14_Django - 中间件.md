# Django - 中间件

中间件是Django请求/响应处理的钩子框架。他是一个轻量级的、低级的‘插件’系统，用于全局改变Djangp的输入或输出

中间件以**类**的形式体现

### 1、中间件

方法：

```
#中间件类必须继承自 gjango.utils.deprecation.MiddlewareMixin类

#实现的方法

process_request(self,request):	#执行路由之前被调用，在每个请求上调用，返回None或HttpResponse对象

process_view(self,request,callback.callback_args,callback_kwargs):	#调用视图之前被调用，在每个请求上调用，返回None或HttpResponse对象
#此两个方法都常用与对用户的筛选，反扒机制

process_response(self,request,reponse):	#所有响应返回浏览器时被调用，在每个请求上调用，返回HttpResponse对象

process_exception(self,request,exception):	#当处理过程中抛出以上时调用，返回一个HttpResponse对象

process_template_response(self,request,response):	#在视图函数执行完毕且视图返回对象中包含render方法时别调用；该方法需要返回实现了render方法的响应对象

注：中间件中的大多数方法在返回None时表示忽略当前操作进入下一项事件，当返回HttpResponese对象时表示此请求结束，直接返回客户端
```



注册中间件：

在manage.py同级目录下创建文件夹middleware

再在middleware文件夹内创建文件mymiddleware.py

middleware.py

```
from gjango.utils.deprecation import MiddlewareMixin

class MyMw(MiddlewareMixin):
	def process_request(self,request):
		print('MyMw process_request do --')
		
	def process_view(self,request,callback,callback_args,,callback_kwargs):
    	print('MyMw process_view do --')
    	
    def process_response(self,request,reponse):
    	print('MyMw process_response do --')
    	return reponsse	#此方法必须返回reponse
```

setting.py

```
MIDDLEWARE = [
	...
	'middleware.mymiddleware.MyMw',
]
```







例子：

针对访问/test/的用户只能访问5次

mymiddleware.py

```
from gjango.utils.deprecation import MiddlewareMixin
import re	#正则表达式引入

class VisitLimit(MiddlewareMixin):
	visit_times = {}	#访问次数
	
	def process_request(self,request):
        ip_address = request.META['REMOTE_ADDR']	#获取当前访问者的IP地址
        path_url = request.path_info	#访问者的路由

        #用正则匹配路由
        if not re.match('^/test',path_url):
            return
		times = self.visit_time.get(ip_address,0)
		print('ip',ip_address,'已经访问',times)
		self.visit_times[ip_address] = times+1
		if times < 5:
			return
		return HttpResponse('您已经访问过'+str(times)+'次，访问被禁止')
```

setting.py

```
MIDDLEWARE = [
	...
	'middleware.mymiddleware.VisitLimit',
]
```





### 2、CSRF - 跨站伪造请求攻击

csrf防范：

setting.py

```
MIDDLEWARE = [
	...
	django.middleware.csrf.CsrfViewMiddleware	#若注释了请取消注释
	。。。
]
```

xxx.html

```
<form action = '/test_csef/' method = 'post'>
	
	{% csrf_token %}	#当开启setting的csrf时,须在form表单中加这句
	<input type = 'test' name = 'username'>
	<input type = 'submit' value = '提交'>
	
</form>
	
```



若开启setting中的csrf但不想校验时

views.py

```
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def my_view(request):
	return HttpResonse('Hello world')
```

