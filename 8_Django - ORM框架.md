# Django - ORM框架



### 1、ORM框架

作用：

建立模型类和表之间的对应关系，允许我们通过面向对象的方式来操作数据库

根据设计的模型类生成数据库中的表格

通过简单的配置就可以进行数据库的切换

#### 1）使用

创建与注册**应用**——python manage.py startapp <应用名>

在应用的**models.py**中

```
from django.db import models

class Book(models.Model):

	# max_length字符串长度，default初始值
	title = models.CharField('书名', max_length = 50, default = '')
	
	# maxdigits位数，decimal_places小数点位置
	price = models.DecimalField('价格',maxdigits = 7, decimal_places = 2)
	
```

#### 2）数据库迁移

在数据库创建表

终端/cmd

```
python3 manage.py makemigrations	#生成迁移文件

python3 manage.py migrate			#执行脚本，同步迁移文件
```

> 若想在创建表后添加字段，则在models.py中的对应类中添加属性，然后再次执行数据库迁移的两条代码，同步迁移文件



#### 3）字段类型

| 字段类型        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| BooleanField()  | 数据库类型：tinyint(1)，编程中表示：True/False，数据库中表示：0/1 |
| CharField()     | 数据库类型：varchar，必须要指定max_length参数值              |
| DateField()     | 数据库类型：date，作用：表示日期，参数：auto_now，auto_now_add，default |
| DateTimeField() | 数据库类型：datetime(6)，作用：表示日期和时间，与DateField()参数相同 |
| FloatField()    | 数据库类型：double，编程与数据库都使用小数表示               |
| DecimalField()  | 数据库类型：decimal(x,y)，编程中：使用小数表示该列的值，数据库中：用小数 |
| EmailField()    | 数据库类型：varchar，编程与数据库都使用字符串                |
| IntegerField()  | 数据库类型：int，编程与数据库都使用整形                      |
| ImageFirld()    | 数据库类型：varchar(100)，编程与数据库都使用字符串           |
| TextField()     | 数据库类型：longtext，作用：表示不定长的字符数据             |

> 更多字段查找官网文档

------

### 2、字段选项，参数

| 字段选项     | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| primary_key  | 主键，True/False                                             |
| blank        | 字段能否为空，True/False                                     |
| null         | 字段能是为空，默认值为False，需要用default来设置默认值，建议不使用 |
| default      | 设置默认值                                                   |
| db_index     | True表示添加该列的索引                                       |
| unique       | True表示添加该列的唯一索引                                   |
| db_column    | 指定列的名称，如果不指定则采用属性名作为列名                 |
| verbose_name | 在admin界面上的显示名称                                      |

> 更多字段选项查找官方文档
>
> 请养成修改完后执行makemigrations 和 migrate 的习惯

------

### 3、Meta类

对**表**的相关操作

```
from django.db import models

class Book(models.Model):

	title = models.CharField('书名', max_length = 50, default = '')
	pub = models.CharField('出版社'，max_length = 100, default = '')
	price = models.DecimalField('价格',max_digits = 7, decimal_places = 2)
	market_price = models.DecimalField('零售价',max_digits=7,decimal_places = 2,default = 0.0)

	class Meta:
		db_table = 'book'
```

> 表名从bookstork_book变成book

| 方法名              | 作用                        |
| ------------------- | --------------------------- |
| db_table            | 更改数据表名称              |
| verbose_name        | 更改admin模型类的名称       |
| verbose_name_plural | 更改admin模型类的名称——复数 |



------



### 4、对数据的操作——增删改查

**QuerySet对象能用 <对象名>.query 的方法来查看数据库中的代码描述**

#### 1）增

方法一：

```
MyModel.objects.create(属性1 = 值1,属性2 = 值2,...)
```

方法二：

创建MyModel实例对象，并调用save()进行保存

```
obj = MyModel(属性1 = 值1,属性2 = 值2,...)
obj.属性 = 值
obj.save()
```

##### （1 Django Shell

利用Django Shell可以代替编写view的代码来进行直接操作

启动方式：

```
python3 manage.py shell

views.py
from <应用名>.models import <表名>
<表名>.objects.create(属性1 = 值1,属性2 = 值2,...)	#方法一

obj1 = <表名>(属性1 = 值1)	#方法二
obj1.save()
```

#### 2）查

##### （1 all()方法

用法：

```
MyModel.objects.all()
```

> 相当于 select * from MyModel

返回值为QuerySet容器对象（类数组）

例子：

```
from bookstore.models import Book
books = Book.object.all()
for book in books:
	print('书名',book.title,'出版社：',book.pub)
```

##### （2 values()方法

用法

```
MyModel.objects.values('<列1>'，'<列2>')
```

> 相当于 select 列1，列2  from MyModel

返回值为字典——{列1：值1}

例子

```
from bookstore.models import Book
a2 = Book.objects.values('title','pub')

Out[]:<QuertSet [{'title':'Python','pub':'出版社'}，{...},...]>

for book in a2:
	print(book['title'])
```

##### （3 values_list()方法

用法：

```
MyModel.objects.values_list('<列1>'，'<列2>')
```

> 相当于 select 列1，列2  from MyModel

返回值为QuerySet容器对象，内部存放‘**元组**’

例子：

```
from bookstore.models import Book
a2 = Book.objects.values_list('title','pub')

Out[]:<QuertSet [('Python','出版社')，(...),...]>

for book in a3:
	print(book[0])
```

##### （4 order_by()方法

用法：

```
MyModel.objects.order_by('-列','列')
```

> 对查询的结果进行根据某个字段选择性的进行排序
>
> 默认为升序排序，降序排序则需要在列前怎加‘-’表示

例子：

```
from bookstore.models import Book
a4 = Book.objects.order_by('-price')


#可以与上面的查询一起使用
a5 = Book.objects.values('title').order_by('-price')


```

##### （5 filter()方法

用法

```
MyModel.objects.filter(属性1=值1，属性2=值2)
```

> 返回包含次条件的全部的数据集
>
> 返回值是QuerySet对象
>
> 

例子：

```
from bookstore.models import Book
books = Book.objects.filter(pub = '清华大学出版社')
for book in books:
	print(书名：',book.title)
```

##### （6 exclude()方法

用法

```
MyModel.objects.exclude(<条件>)
```

> 返回不包含此条件的全部数据集

例子：

```
books = models.Book.objects.exclude(pub = '清华大学出版社',price = 50)
for book in books:
	print(book)
```

##### （7 get()方法

用法

```
MyModel.objects.get(<条件>)
```

> 返回满足条件的唯一一条数据
>
> 没有查询到或多于一条就报错

例子：

```
books = models.Book.objects.get(id = 1)
for book in books:
	print(book)
```

##### 以上是等值查询

------

##### 查询谓词

##### （1 __exact 等值匹配

用法

```
Author.objects.filter(id__exact = 1)
```

> 等同于select * from author where id = 1

##### （2 __contains 包含指定值

用法

```
Author.objects.filter(name__contains = 'w')
```

> 等同于select * from author where name like '%w%'
>
> __startswith:xxx 以什么开始
>
> __endswith:xxx 以什么结束

##### （3 __gt 大于指定值

用法

```
Author.objects.filter(age__gt = 50)
```

> 等同于select * from author where age > 50
>
> __gte : 大于等于
>
> __lt: 小于
>
> __lte: 小于等于

更多查询官网文档

------

#### 3）改

##### 修改单个实体

1.查——通过get()得到要修改的实体对象

2.改——通过 对象.属性 的方式修改数据

3.保存——通过 对象.save() 保存数据

例子：

```
from Bookstork.models import Book
b1  = Book.object.get(id=1)

In []:b1
Out []:<Book:Python_清华大学出版社_20.00_25.00>

b1.price = 22
b1 = save()

In []:b1
Out []:<Book:Python_清华大学出版社_22.00_25.00>
```

##### 批量数据更新

使用QuerySet的undate(属性=值)实现批量修改

例子：

```
#将所有 id大于3 的图书价格定为0元
books = Book.object.filter(id__gt=3)
books.update(price = 0)
```



------

#### 4）删

##### 删除单个数据

1. 查找查询结果对应的一个数据对象
2. 调用这个数据对象的delete()方法

```
try:
	auth = Author.objects.get(id = 1)
	auth.delete()
except:
	print('删除失败')
```

##### 批量删除数据

例子：

```
#删除全部作者中年龄大于65的全部信息
auths = Author.objects.filter(age__gt=65)
auths.delete()
```



##### 伪删除

在创建表时添加字段is_active

例子：

**models.py**中

```
from django.db import models

class Book(models.Model):

	title = models.CharField('书名', max_length = 50, default = '')
	pub = models.CharField('出版社'，max_length = 100, default = '')
	price = models.DecimalField('价格',max_digits = 7, decimal_places = 2)
	market_price = models.DecimalField('零售价',max_digits=7,decimal_places = 2,default = 0.0)
	is_active = models.BooleanField('是否活跃'，default = True)

	class Meta:
		db_table = 'book'
```

> 记得要
>
> python3 manage.py makemigrations
>
> python3 manage.py migrate

**删除时将is_active = false，查询时添加查询is_active =true**

------

### 5、F对象

一个F对象代表数据库中某条记录的字段的信息

作用：

> 通常是对数据库中的字段值在不获取的情况下进行操作
>
> 用于类属性（字段）之间的比较

语法：

```
from django.db.models import F 
F('列名')
```

例子：

```
#对图书价格加一的操作
b1  = Book.object.get(id=1)
b1.price = b1.price+1
b1.save()
#这为常规操作
#但当出现并行操作时，理应是b1.price+1运行两次结果为10->11->12,但并行操作导致10-#>11->11
#原因是在b1.price = b1.price+1中的第二个b1.price是取得值，因用类似指针的方法
#F对象可以理解为指针
Book.object.get(id = 1).update(price = F('price')+1)
```



------

### 6、Q对像

使用复杂的逻辑‘或 |’，逻辑‘非 ~’的操作

语法：

```
Q(条件1) | Q(条件2)		#'或'操作——条件1成立或条件2成立
Q(条件1) & Q(条件2)		#'与'操作——条件1成立且条件二成立
Q(条件1) &~ Q(条件2)	#'与非'操作——条件一成立且条件二 不 成立
```

例子：

```
#想找出定价低于20元 或 清华大学出版社的全部书
from django.db.models import Q
from BookStork.models import Book

Book.object.filter(Q(price_lt = 20)|Q(pub = '清华大学出版社'))
```

