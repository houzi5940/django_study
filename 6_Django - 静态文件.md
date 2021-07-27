# Django - 静态文件

1、静态文件配置 

settings.py中

```
...
STATIC_URL = '/static'	#提示url中偶遇static时，访问静态文件
...
STATICFILES_DIRS = (		#静态文件路径
os.path.join(BASE_DIR,"static"),
)
```

使用：

方法一：直接使用

html

```
<img src = '/static/image/test.jpg'>
```

方法二：django标签

html

```
{% load static %}
<img src = '{% static 'image/test.jpg' %}'>
```

