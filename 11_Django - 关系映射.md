# Django - 关系映射

### 1、关系映射

常见的关系映射有：

一对一映射

一对多映射

多对多映射

------

### 2、一对一

语法：

```
OneToOneField(类名, on_delete = xxx)

#例子
class A (model.Model):
	...
	
class B (model.Model):
	属性 = models.OneToOnefield(A, on_delete = xxx)
```

on_delete - 级联删除

可选项有

| 可选项         | 意义                                                         |
| -------------- | ------------------------------------------------------------ |
| models.CASCADE | 模拟SQL约束ON DELETE CASCADE的行为，并删除包含ForeignKey的对象 |
| models.PROTECT | 有数据关联时不允许删除                                       |
| SET_NULL       | 设置ForeignKey null，使用时需要指定null = True               |
| SET_DEFAULT    | 将ForeignKey 设置为其默认值，使用时必须设置ForeignKey 的默认值 |

例子

#### 1）创建表 

 models.py

```
class Author(models.Model):
	name = models.CharField('姓名', maxlength = 11)

class Wife(models.Modle):
	name = models.CharField('姓名', maxlength = 11)
	author = model.OneToOneField(Author, on_delete = models.CASCADE)
```

#### 2）创建数据 

在python脚本或python编译器环境下或shell下

当前项目的根目录下

```
In[]:from <应用名>.models imporl *

#无外键的模型类，正常创建
In[]:a1 = Author.objects.create(name = 'wang')

#有外键的模型类有两种方法
In[]:w1 = Wife.objects.create(name = 'wangfuren',author = a1)
In[]:w2 =  Wife.objects.create(name = 'wangfuren',author_id = 1)
```

#### 3）查询关联数据

正向查询 —— 直接通过外键属性查询

```
#通过wife找author
from .models import wife
wife = Wife.objects.get(name = 'wangfuren')
print(wife.name,'的老公是',wife.author.name)
```

反向查询 —— 没有外键属性的一方调用反向属性查询到关联的的一方

```
#通过author找wife
from .models import wife
author = Author.objects.get(name = 'wang')
print(author.wife.name)		#注意wife是小写
```

------

### 3、一对多

语法

```
#在多表上创建外键
class A(models.Model):
	...
	
class B (models.Model):
	属性 = models.ForeignKey("一"的模型类, on_delete = xx)
```

on_delete - 级联删除

可选项与一对一 一样

例子：

#### 1）创建表

models.py

```
class Publisher(models.Model):
	#出版社 [一]
	name = models.CharField('出版社名称'，max_length = 50)
	
class Book(models.Model):
	#书名 [多]
	title = models.CharField('书名'，max_length = 11)
	publisher = models.ForeignKey(Publisher,on_delete = models.CASCADE)
```

#### 2）创建数据

与一对一相似

```
#先创建'一' 再创建'多'
In[]:from <应用名>.models imporl *

#无外键的模型类，正常创建
In[]:p1 = Publisher.objects.create(name = '中信出版社')

#有外键的模型类有两种方法
In[]:b1 = Book.objects.create(name = 'Python1',publisher = p1)
In[]:b2 =  Book.objects.create(name = 'python2',publisher_id = 1)
```

#### 3）查询关联数据

正向查询

```
#有外键的模型类
abook = Book.objects.get(id = 1)
print(abook.title,'的出版社是:',abook.publisher.name
```

反向查询

```
#没有外键属性的一方调用反向属性查询到关联的的一方
pub1 = Publisher.objects.get(name = '中信出版社')
books = pub1.book_set.all()	#注意在'多'的模型类后面加上set

print('中信出版社的书有：')
for book in books:
	print(book.title)
```

### 4、多对多

mysql中创建多对多需要依赖第三张表来实现

Django中代码直接生成第三张表

语法

```
属性 = models.ManyToManyField(MyModel)
```

#### 1）创建表

```
class Author(models.Model):
	name = models.CharFiled('姓名',max_length = 11)
	
class Book(models.Model):
	title = models.CharField('书名'，max_length = 11)
	authors = models.ManyToManyField(Author)
```

> 在数据库中会创建三个表，Author,Book,Book_Author

#### 2）创建数据

```
#方法一，先创建 author 再创建 book
author1 = Author.objects.create(name = '吕老师')
author2 = Author.objects.create(name = '王老师')
	#吕老师和王老师同时写了一本书
#注意book创建表时有用语法，所以要book_set
book11 = author1.book_set.create(title = 'Python')	
author2.book_set.add(book11)


#方法二，先创建 book 再创建 author
book = Book.objects.create (title = 'python1')
	#吕老师和王老师同时写了一本书
author2 = Author.objects.create(name = '王老师')
author1 = book.authors.create(name = '吕老师')
book.authors.add(author2)
```

#### 3）查询关联数据

正向查询

```
#通过 Book 查询对应所有的 Author
book.author.all()
book.author.filter(<查询属性> = <查询值>)
```

反向查询

```
#通过 Author 查询对应所有的 Book
author.book_set.all()
author.book_set.filter(<查询属性> = <查询值>)
```

