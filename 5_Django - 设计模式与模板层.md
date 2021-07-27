# Django - 设计模式与模板层

### 1、创建模板文件夹

创建模板文件夹 ：<项目名>/templates

setting.py中的TEMPLATES配置项中

> 1、BACKEND：指定模板引擎
>
> 2、DIRS：模板的搜索目录
>
> 3、APP_DIRS：是否要在应用中的templates文件夹中搜索模板文件
>
> 4、OPTIONS：有关模板的选项

修改

```
'DIRS':[os.path.join(BASE_DIR,'template')],
```

### 2、模板加载

方法一：

views.py

```
from django.template import loader
from django.html import HttpResponse
def test_html (request):
	t =  loader.get_template('test_html.html')	#通过loader加载模板
	html = t.render()	#将t转换成HTML字符串
	return HttpResponse(html)	#用响应对象将转换的字符串返回给浏览器
```

方法二：

方法：

```
from django.shortcuts import render
eturn render(request, 'xxx.html',字典数据)
```

例子1：

views.py

```
from django.shortcuts import render
from django.html import HttpResponse
def test_html (request):
	return render(request, 'test_html.html')
```

例子二：

views.py

```
from django.shortcuts import render
from django.html import HttpResponse
def test_html (request):
	dic = {
		"变量1"："值1",
		"变量1"："值1",
	}
	return render(request, 'xxx.html',dic)
```

> 模板中，我们可以使用***{{ 变量名 }}***的语法调用视图传进来的变量

### 3、模板的变量

| 数据类型   | 用法                                     |
| ---------- | ---------------------------------------- |
| str-字符串 | dic['str'] = 'houzi'                     |
| int-整形   | dic['int'] = 88                          |
| list-数组  | dic['lis'] = ['houzi1','houzi2','houzi3] |
| tuple-元组 | dic['dict'] = {'a':9,'b':8}              |
| dict-字典  | dic['dict'] = {'a':9,'b':8}              |
| func-方法  | dic['func'] = test_def                   |
| obj-对象   | dic['class_obj'] = dog()                 |

### 4、模板标签

标签模板

```
{% 标签 %}
...
{% 结束标签 %}
```

###### 1）if标签

语法：

```
{% if 条件语句1 %}
...
{% elif 条件语句2 %}
...
{% elif 条件语句3 %}
...
{% else %}
...
{% endif %}
```

###### 2）for标签

```
{% for 变量 in可迭代对象 %}
...
{% empty %}		#当循环值为空时运行下面代码
...
{% endfor %}
```

内置变量 - forloop

| 变量                | 描述                               |
| ------------------- | ---------------------------------- |
| forloop.counter     | 循环的当前迭代（从1开始）          |
| forloop.counter0    | 循环的当前迭代（从0开始）          |
| forloop.revcounter  | counter值的倒序                    |
| forloop.revcounter0 | revcounter值的倒序                 |
| forloop.first       | 如果这是第一次通过循环，则为真     |
| forloop.last        | 如果这是最后一次循环，则为真       |
| forloop.parentloop  | 当嵌套循环，parentloop表示外层循环 |

### 5、模板过滤器

通过使用过滤器改变变量的输出显示

语法

```
{{  变量 | 过滤器1:'参数值1'| 过滤器2:'参数值2'...}}
```

| 过滤器            | 说明                                         |
| ----------------- | -------------------------------------------- |
| lower             | 将字符串转换为全部小写                       |
| upper             | 将字符串转换为全部大写                       |
| safe              | 默认不对变量内的字符串进行html转义           |
| add:'n'           | 将value的值增加n                             |
| truncatechars:'n' | 如果字符串字符多余指定数量。那么就会被截断。 |

### 6、模板的继承

语法 - 父模板中：

block标签——在父模板中定义，可以在子模版中覆盖

语法 - 子模板中：

继承模板 extends 标签（写在模板文件的第一行）

> 如{% extends 'base.html' %}

子模版重写父模板中的内容块

```
{% block block_name %}
子模版用来覆盖父模板中block_name块的内容
{% endblock block_name %}
```

例子：

父模板.html

```
<head>
	<meta charset="UTF-8">
	{%block title%}
		<title>父模板</title>
	{%endblock%}
</head>
<body>
	<a href="/chird1">子模版1</a>
	<a href="/chird2">子模版2</a>
	<br>
	
	{%block info%}
	这是父模板
	{%endblock%}
	<h3>网站尾部</h3>
</body>
</html>

```

子模版1.html

```
{%extends '父模板.html'%}

{%block title%}
	<title>父模板</title>
{%endblock%}

{%block info%}
	这是子模版1
{%endblock%}
```

子模版2.html

```
{%extends '父模板.html'%}

{%block title%}
	<title>父模板</title>
{%endblock%}

{%block info%}
	这是子模版1
{%endblock%}
```

> 注：继承时，服务器端的动态内容无法继承
