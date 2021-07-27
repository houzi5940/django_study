# Django - 模型层

模型层 - 负责跟数据库之间进行通信

### 1、安装Mysql

Linux

终端

```
sudo apt list --installed|grep -E 'libmysqlclient-dev|python3-dev'
#若无输出，输入
sudo apt-get install python3-dev default-libmysqlclient-dev

sudo pip3 install mysqlclient	#安装mysqlclient
```

windows

cmd

```
pip install mysqlclient
```

 

### 2、创建数据库

终端，cmd

```
mysql -uroot -p
create database <数据库名> default charset utf8;
```

> 数据库名一般与项目名一致

配置数据库

settings.py

```
...
DATABASES = {
	'default': {
		'ENGINE':'django.db.backends.mysql',
		'NAME':'<数据库名>'，
		'USER':'root',
		'PASSWORD':'123456',
		'HOST':'127.0.0.1',
		'PORT': '3306'
	}
}
...
```



### 3、ORM框架

作用：

建立模型类和表之间的对应关系，允许我们通过面向对象的方式来操作数据库

根据设计的模型类生成数据库中的表格

通过简单的配置就可以进行数据库的切换

#### 1）使用

创建与注册应用——python manage.py startapp <应用名>

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

