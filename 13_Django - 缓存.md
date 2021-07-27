# Django - 缓存



## 一、整体缓存

定义：缓存时一类可以更快的读取数据的介质统称，使用户能**更快的访问网址**，一般用来存储临时数据，常用介质的时读取熟读很快的内存

意义：视图渲染有一定的成本，数据库的平凡查询过高；所以对于**低频变动**的页面可以考虑使用缓存技术，减少实际渲染次数；用户拿到响应的时间成本会更低。

### 1、数据库缓存

定义：将缓存的数据存储到数据库中，虽然存储介质不变，但减少了原语句的复杂程度，相当于在A表中进行复杂的查询，查询结果存储在B中，第二次便可直接从B中进行简单查询来取得数据，减少时间。

setting.py

```
CACHES = {
	'default':{
		'DACKEND':'django.core.cache.backends.db.DatabaseCache',	#引擎
		'LOCATION':'my_cache_table',	#表名
		'TIMEOUT':300,		#缓存时间，以秒为单位
		'OPTIONS':{
			'MAX_ENTRIES':300,	#缓存最大数据条数
			'CULL_FREQUENCY':2,	#缓存条数达到最大值时，删除1/x的缓存数据
		}
	}
}
```

> 若使用数据库缓存，则需要自己先创建my_cache_table表
>
> manage.py同级下，终端输入
>
> ```
> python manage.py creatcachtable
> ```

------

### 2、本地内存缓存（一般用于测试）

将缓存的数据存储在本机的内存中

setting.py

```
CACHES = {
	'default':{
		'BACKEND':'django.core.cache.backends.locmem.locMemCache',	#引擎
		'LOCATION':'unique-snowflake',	#内存寻址
	}
}
```

------

### 3、文件系统缓存

将缓存的数据存储在本地的文件中

setting.py

```
CACHES = {
	'default':{
		'BACKEND':'django.core.cache.backends.filebased.FileBasedCache',	#引擎
		'LOCATION':'c:\test\cache',	#文件路径
	}
}
```



------

### 4、使用缓存

在视图函数中使用：

views.py

```
from django.views.decorators.cache import cache_page


@cache_page(30) #单位秒
def my_view(request):
	pass
```



在路由中使用缓存：

urls.py

```
from django.views.decorators.cache import cache_page

urlpatterns = [
	path('foo/', cache_page(62)(my_view)),
]
```



------

## 二、局部缓存

整体缓存是缓存整个return给客户端的东西

局部缓存可以缓存指定数据，能使得这个数据被多个地方或整体使用，提高了自由度

### 1、使用局部缓存



```
#方法一：引入
from django.core.cache import caches

cache1 = caches['myalias']	#此处的myalias是上面整体缓存中1、2、3的setting.py配置的CACHES
cache2 = caches['myalias_2']


#方式二：
from django.core.cache import cache
```

| 方法二：cache的内置方法               | 描述                                                |
| ------------------------------------- | --------------------------------------------------- |
| cache.set(key, value ,timeout)        | key:缓存的key，value：存储的对象，timeout：存储时间 |
| cache.get(key)                        | key:缓存的key                                       |
| cache.add(key,value)                  | 存储缓存，只在key不存在的时候生效                   |
| cache.get_or_set(key, value ,timeout) | 如果未获取到数据则执行set操作                       |
| cache.set_many(dict,timeout)          | 批量存储                                            |
| cache.get_many(key_list）             | 批量获取                                            |
| cache.delete(key)                     | 删除key的缓存数据                                   |
| cache.delete_many(key_list)           | 批量删除                                            |





------

## 三、浏览器缓存

### 1、强缓存

不会向服务器发送请求，直接从缓存中读取资源

**响应头 - Expires**

定义：缓存过期时间，用来指定资源到期的时间，是服务器端的具体时间点



**响应头 - Cache-Control**

Cache-Control主要用于控制网页缓存，如当‘Cache-Control : max-age=120’代表请求创建时间后的120秒，缓存失败。

说明目前服务器都会带着这两个头同时响应给浏览器，浏览器优先使用Cache-Control

### 2、协商缓存

强缓存中的数据一旦过期，还需要跟服务器进行通信，但部分静态文件与图片的数据量过大，这是需要使用协商缓存，若静态文件或图片未改变，则继续使用之前缓存的，否则更新

**响应头 - Last-Modified 和 请求头 - If-Modified-Since**

响应头 - Last-Modified：Last-Modified为文件的最近修改时间，浏览器第一次请求静态文件时，服务器如果返回Last-Modified响应头，则表示该资源为需协商缓存

请求头 - If-Modified-Since：当缓存到期后，浏览器将获取到的Last-Modified值作为请求头If-Modified-Since的值，与服务器发送请求协商，服务器返回304响应码【响应体为空】，代表缓存继续使用，200响应码代表缓存不可用【响应体为最新资源】





**响应头 - ETag  和 请求头 - If-None-Match**

响应头 - ETag：应为响应头 - Last-Modified可能存在不可靠性，所有ETag是服务器响应请求时，返回当前资源文件的一个唯一标识[哈希值]，只要资源有变化，ETag就会重新生成

请求头 - If-None-Match：当缓存到期后，浏览器将获取到的ETag的值作为请求头If-None-Match的值，与服务器发送请求协商，服务器返回304响应码【响应体为空】，代表缓存继续使用，200响应码代表缓存不可用【响应体为最新资源】