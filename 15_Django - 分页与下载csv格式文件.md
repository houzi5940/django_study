# Django - 分页与下载csv格式文件

### 1、分页

分页只在web页面有大量数据需要显示时，为了阅读方便在每个分页中只显示部分数据

Django中提供了paginator类可以方便实现分页功能

#### 1）paginator

负责分页数据整体的管理

```
paginator = Paginator(object_list,per_page)
	-参数
		-object_list 需要分类数据的对象列表
		-per_page 每页数据的个数
	-返回值
		-Paginator对象
	-属性
		-count: 需要分页数据的对象总数
		-num_pages: 分页后的页面总数
		-page_range: 从1开始的range对象，用于记录当前页码数
		-per_page: 每页数据的个数
c
	-参数 number为页码信息（从1开始）
	-返回当前number页对应的信息
	-如果提供的页码不存在，抛出InvalidPage异常
```



#### 2）page对象

负责具体某一页的数据管理

```
page = paginator.page(number)
	-对象属性（page.xxx）
		-object_list: 当前页面上所有数据对象的列表
		-number: 当前页码（从1开始）
		-paginator: 当前page对象相关的Paginator对象
	-对象方法（page.xxx()）
		-has_next(): 如果有下一页，返回True
		-has_previous(): 如果有上一页，返回True
		-has_other_pages(): 如果有上一页或下一页，返回True
		-next_page-number():返回下一页的页码，如果下一页不存在，抛出InvalidPage异常
		-previous_page_number():返回上一页的页码，如果上一页不存在，抛出InvalidPage异常
```



例子：

做一个简短的分页

views.py

```
def test_page(request):
	#/test_page?page=1
	page_num = request.GET.get('page', 1)	#取得page的值，默认值为1
	all_data = ['a','b','c','d','e']
	
	#初始化paginator
	paginator = Paginator(all_data,2)
	
	#初始化 具体页码的page对象
	c_page = paginator.page(int(page_num))
	return render(request,'test_page.html',locals())
```

test_page.html

```
...
<body>
	{% for p in c_page %}
		<p>
			{{ p }}
		<p>
	{% endfor %}
	
	<--上一页-->
	{% if c_page.has_previous %}
		<a href="/test_page?page={{ c_page.previous_page_number }}">上一页</a>
	{% else %}
		上一页
	{% endif %}
	
	<--数字页码-->
	{% for p_num in paginator.page_range %}
		{% if p_num==c_page.number %}
			{{p_num}}
		{% else %}
			<a href="test_page?page={{ p_num }}">{{ p_num }}</a>
		{% endif %}
	{% endfor %}
	
    <--下一页-->
	{% if c_page.has_next %}
		<a href="/test_page?page={{ c_page.next_page_number }}">下一页</a>
	{% else %}
		下一页
	{% endif %}
</body>
```

urls.py

```
urlpatterns = [
	...
	path('test_page',views.test_page),
]
```



### 2、常规使用python中的csv

```
import csv
with open('eggs.csv','w',newline = '') as csvfile:
	writer = csv.writer(csvfile)
	writer = writerow(['a','b','c'])
```



#### 1）csv文件的下载

例子

```
import csv
from django.http import HttpResponse
from .models import Book

def make_csv_view(request):
	reuqest = HttpResponse(content_type = 'text/csv')
	response['Content-Dispostiton'] = 'attachment;filename ="mybook.csv"'
	#上面两行基本不改
	all_book = Book.objects.all()
	writer = csv.writer(response)
	writer.writerow(['id','title'])
	for b in all_book:
		writer.writerow([b.id,b.title])
		
	return response
```



#### 2）与分页结合例子

views,py

```
def make_page_csv(reuqest):
	#/test_page?page=1
	page_num = request.GET.get('page', 1)	#取得page的值，默认值为1
	all_data = ['a','b','c','d','e']
	
	#初始化paginator
	paginator = Paginator(all_data,2)
	
	#初始化 具体页码的page对象
	c_page = paginator.page(int(page_num))
	
	#
	reuqest = HttpResponse(content_type = 'text/csv')
	response['Content-Dispostiton'] = 'attachment;filename ="page-%s.csv"'%(page_num)
	writer = csv.writer(response)
	for b in c_page:
		writer.writerow([b])
	return response
```

urls.py

```
urlpatterns = [
	...
	path('make_page_csv',views.make_page_csv),
]
```

test_page.html

```
...
<body>
	<--下载csv文件-->
	<a href="/make_page_csv?page={{ c_page.number }}">生成csv</a>
	
	{% for p in c_page %}
		<p>
			{{ p }}
		<p>
	{% endfor %}
	
	<--上一页-->
	{% if c_page.has_previous %}
		<a href="/test_page?page={{ c_page.previous_page_number }}">上一页</a>
	{% else %}
		上一页
	{% endif %}
	
	<--数字页码-->
	{% for p_num in paginator.page_range %}
		{% if p_num==c_page.number %}
			{{p_num}}
		{% else %}
			<a href="test_page?page={{ p_num }}">{{ p_num }}</a>
		{% endif %}
	{% endfor %}
	
    <--下一页-->
	{% if c_page.has_next %}
		<a href="/test_page?page={{ c_page.next_page_number }}">下一页</a>
	{% else %}
		下一页
	{% endif %}
</body>
```

