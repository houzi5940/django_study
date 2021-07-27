# Django - 上传文件和发送邮件

### 1、上传文件规范

#### 1）html

```
<form enctype = "multipart/fotm-data">

<input type="file" name="xxx">

</form>
```



例子：

html:

```
<body>
<form action="/test/upload" method="post" enctype="multipart/fotm-data">
	<p>	<input type="text" name="title"></p>
	<p>	<input type="file" name="myfile"></p>
	<p>	<input type="submit" value="上传"></p>
</form>
</body>
```



#### 2）视图函数 views.py

```
file=request.FILES['xxx']

1.FILES 的 key 对应页面中 file 框的 name
2.file 绑定文件流对象
3.file.name 文件名
4.file.file 文件的字节流数据
```





#### 3）配置访问路径与存储路径

setting.py 中的 MEDIA

```
...
MEDIA_URL = '/media/'	#访问路由，网页发送相关路由的请求时，代表要读取上传的文件
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')	#本地的保存路径
```

MEDIA_URL 和 MEDIA_ROOT 需要手动绑定到主路由

主路由 urls.py

```
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
	...
]

urlpatterns += static(setting.MEDIA_URL , document_root = setting.MEDIA_ROOT )
```

> 等价与做了 MEDIA_URL  开头的路由，Django街道该特征的请求后去MEDIA_ROOT路径查找资源

#### 4）文件存储

views.py

```
#传统open方法
def upload_view(request):
	if request.method =='GET':
		return render(request,'test_upload.html')
	elif request.method =='POST':
		a_file = request.FILES['myfile']
		print("上传的文件名是：",a_file.name)
		filename = os.path.join(settings.MEDIA_ROOT,a_file.name)
		with open(filename,'wb') as f:
			data = a_file.file.read()
			f.write(data)
		return HttpResponse("接收文件："+a_file.name+"成功")
```

> 此方法可能存在上传的文件重名，须自行添加判断代码

```
#ORM方法
def upload_view_dj(request):
	if request.method =='GET':
		return render(request,'test_upload.html')
	elif request.method =='POST':
		title = request.POST['title']
		a_file = request.FILES['myfile']
		
		Content.objects.create(decs = title, myfile = a_file)
		return HttpResponse("接收文件："+a_file.name+"成功")
```

> 此方法须在数据库的表中添加 FileField(upload = '<子目录名>')
>
> 此方法解决了重命名的问题，但他是针对数据库表进行操作，将存储地址放进 FileField字段中



例子：

models.py

```
class Content(models.Model):
	
	title = models.CharField('文章名字'，max_length = 11)
	picture  = models.FileField(upload_to = 'picture')
```

> 记得保存后进行 python manage.py makemigrations
>
> python manage.py migrate

views.py

```
from upload_app.modles import Content
def upload_view_dj(request):
	if request.method =='GET':
		return render(request,'test_upload.html')
	elif request.method =='POST':
		title = request.POST['title']
		a_file = request.FILES['myfile']
		
		Content.objects.create(title = title, picture = a_file)
		return HttpResponse("接收文件："+a_file.name+"成功")
```

------



### 2、发送邮件

业务场景

- 业务警告
- 邮件验证
- 密码找回

协议

- SMTP，简单邮件传输协议（25号端口），属于推送协议（发送协议）
- IMAP，交互式邮件访问协议（143号端口），属于拉取协议（收取协议）
- POP3，也是拉取行协议（收取协议）

| IMAP                                             | POP3                               |
| ------------------------------------------------ | ---------------------------------- |
| 具备照耀浏览功能，可预览部分摘要，再下载整个邮件 | 必须下载全部邮件，无照耀功能       |
| 双向协议，客户端操作可反馈给服务器               | 单向协议，客户端操作无法同步服务器 |

#### 1）django发邮件

原理：

- 给 django 授权一个邮箱
- django用该邮箱给对应收件人发邮件
- django.core.mail 封装了电子邮件的自动发送 SMTP 协议



#### 2）授权邮箱

以qq邮箱为例：

登录自己 qq 号的 qq 邮箱，修改‘QQ邮箱 -> 设置 -> 账户 -> “POP3/IMAP....服务”’

点击启动，更具提示拿到授权码

在django中的setting.py进行设置

```
.....

EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.qq.com'	#腾讯QQ邮箱的SMTP服务器地址
EMAIL_PORT = 25	#SMTP服务的端口号
EMAIL_HOS_USER = 'XXXX@qq.com'	#发送邮件的QQ邮箱
EMAIL_HOST_PASSWORD = '**********'	#在QQ邮箱 -> 设置 -> 账户 -> "POP3/IMAP....服务"得到的授权码
EMAIL_USE_TLS = False #与SMTP服务器通时，是否启动TLS链接（安全链接）默认False
```



#### 3）函数调用

```
from django.core import mail

mail.send_mail(
	subject,	#题目
	massage,	#消息内容
	from_email,	#发送者[当前配置的邮箱]
	recipient_list=['xxx@qq.com'],	#接收者邮件列表
)
```







例子：

写一个视图错误，将错误以邮件方式发送

使用中间件——根目录下的 middleware/mymiddleware.py

```
from django.core import mail
import traceback	#追溯错误

class ExceptionMW(MiddlewareMixin):
	def process_exception(self,request,exception):
		print(exception)
		print(traceback.format_exc())	#取得异常的详细信息
		
		mail.send_mail(subject = '出现错误',message = traceback.format_exc(),from_email = 'XXXX@qq.com',recipient_list=['xxx@qq.com'],)
		
		return HttpResponse('---当前网页繁忙---')
```

> 记得在settings.py中注册
>
> 例子使用了中间件，追溯错误，邮件发送
>
> from_email 的参数填 setting.py 中 EMAIL_HOS_USER的邮箱
>
> recipient_list可以在 settings.py 中配置一个参数使用，具体操作如下
>
> ```
> #settings.py中
> EX_EMAIL = ['xxxx@qq.com','xxxx@qq.com',...,]
> 
> #需要调用时引入
> #from django.conf import setting
> 
> #recipient_list = settings.EX_EMAIL
> 
> ```

------

