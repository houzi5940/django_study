# Django - 项目部署

### 1、前言

目的：

在项目开发完成后，在自己的服务器上或其他服务器上部署项目，并实现发布

步骤：

1、需要在服务器上搭建与开发环境相同的环境（即 python 版本，mysql 环境等）

2、django 的项目转移

```
sudo scp <开发环境下的项目路径> root@<服务器IP>:<服务器项目路径>
请输入root密码：
```

3、使用 uWSGI 代替 python3 manage.py runserver 方法启动服务器

> runserver 只是开发时测试使用的方法，性能不佳，请求过大，效果不好，所以用uWSGI代替runserver进行启动服务器
>
> 但uWSGI目前**不支持windows环境**下安装，所有一般部署项目的服务器尽可能的选择Linux，且Linux更适合相关的项目研发与开发

4、配置 nginx 反向代理服务器

5、用 nginx 配置静态文件路径，解决静态路径问题

------



### 2、uWSGI

uWSGI 是 WSGI 的一种变种，它实现了 http 协议  WSGI  协议以及 uwsgi 协议

uWSGI 功能晚上，支持协议众多，在 python web 圈热度高

#### 1）安装

linux下执行

```
#安装
sudo pip3 install uwsgi==2.018 -i https://pypi.tuna.tsinghua.edu.cn/simple/

#检查是否成功
sudo pip3 freeze|grep -i 'uwsgi'
#输出 uWSGI==2.0.18 代表成功
```

> uWSGI目前**不支持windows环境**下安装，所有请在 Linux 下操作

#### 2）配置uWSGI

创建配置文件 <项目文件夹>/uwsgi.ini (与setting.py同级)

uwsgi.ini

```
[uwsgi]
#socket = 127.0.0.1:8000	#此模式需要有 nginx 
http=127.0.0.1:8000		#服务器启动时的IP地址与端口，与socket二选一

chdir=/home/.../my_project	#此路径为项目的绝对路径
wsgi-file=my_project.wsgi.py	#此路径为相对路径
process = 4	#进程个数（最多为你cpu个数）
threads = 2	#每个进程的线程个数

pidfile = uwsgi.pid	#进程id写在的位置，服务的pid记录文件
daemonize = uwsgi.log	#后台启动，并且日志写入uwsgi.log文件中
master = True	#启动主进程管理模式
```

> 特殊说明：Django 的 setting.py 也须做如下配置
>
> ```
> 1、修改 setting.py 将 DEBUG=True 改为 DEBUG=False
> 2、修改 setting.py 将 ALLOWED_HOSTS = [] 改为 ALLOWED_HOSTS = ['网站域名']或者['服务监听的IP地址'] 
> ```

#### 3）启动与停止

启动：cd 到 uWSGI 配置文件目录下的终端

```
uwsgi --ini uwsgi.ini
```



停止：cd 到 uWSGI 配置文件目录下的终端

```
uwsgi --stop uwsgi.pid	#注意启动是ini，停止是pid
```

> 无论启动还是关闭都要在执行相对应的代码后执行
>
> ```
> ps aux|grep 'uwsgi'
> ```
>
> 查看是否有对应预期

------



### 3、nginx

- Nginx 是轻量级的高性能web服务器，提供了诸如 HTTP 代理和反向代理，负载均衡等一系列重要特性
- 全c语言编写，执行效率高
- 作用：1负载均衡，多台服务器轮流处理请求，2反向代理
- 原理：客户端请求 nginx，再由 nginx 将请求转发 uWSGI 运行的 django

**nginx默认是80端口**

#### 1）安装

```
#安装
sudo apt install nginx	#若下载速度慢，自行进行国内换源

#检查是否安装成功
nginx -v
```

#### 2）配置

/etc/nginx/sites-enabled/default

```
#在server节点下添加新的location项，指向uwsgi的IP与端口
server{
	...
	location / {
		#try_files $uri $uri/ =404;	#若未注释，请手动注释
		
		uwsgi_pass 127.0.0.1:8000;	#重定向到127.0.0.1的8000端口
		include /etc/nginx/uwsgi-params;	#将所有的参数转到uwsgi下
	}
	...
}
```

> 更改后重启 nginx 服务
>
> 在安装 nginx 成功时，自动启动了 nginx



#### 3）启动和停止

```
sudo /etc/init.d/nginx start|stop|restart|status
#或
sudo service nginx start|stop|restart|status


启动 - sudo /sudo /etc/init.d/nginx start
停止 - sudo /sudo /etc/init.d/nginx stop
重启 - sudo /sudo /etc/init.d/nginx restart
```

> 使用 nginx 后，将 uwsgi.ini 中的 http 改成 socket
>
> ```
> [uwsgi]
> socket = 127.0.0.1:8000	#此模式需要有 nginx 
> #http=127.0.0.1:8000	#服务器启动时的IP地址与端口，与socket二选一
> ```

更改完后重新启动 uwsgi 

访问的网址为127.0.0.1:80

> nginx 默认端口为80，且 nginx 是反向代理，所以请求先经过 nginx 后才进入 uwsig 最后才进入 django



### 4、nginx 静态文件配置

1. 创建路径 - 主要存放Django所有静态文件，如：/home/.../<项目名>_static/

2. 在Django settings.py 中添加新配置

   ```
   STATIC_ROOT = '/home/.../<项目名>_static/static'
   #注意比原路径多一个static文件夹
   ```

   

3. 进入项目，执行 python3 manage.py collectstatic 后，Django 将项目重所有静态文件复制到STATIC_ROOT中，包括Django内建的静态文件

   

   nginx配置中添加新的配置

   ```
   #file：/etc/nginx/sites-enabled/default
   #新添加location / static 路由配置，重定向到指定的路径
   
   server{
   	...
   	location / static{
   		root /home/.../<项目名>_static/;	#第一步创建的目录
   	}
   }
   ```





### 5、常见问题排查

排查问题最重要的是——看日志，看日志，看日志

nginx 日志位置：

异常信息：/var/log/nginx/error.log

正常访问信息：/var/log/nginx/access.log



uwsgi 日志位置：

项目同名目录下，uwsgi.log



常见

1. 访问127.0.0.1:80 地址，502响应——代表nginx反向代理成功，但 uwsgi 未启动
2. 访问127.0.0.1:80/url   404响应——路由不在django中，nginx配置错误，未注释掉try_file



### 6、自定义报错页面

在模板文件夹中添加 404.html 模板，即可生效（在settings.py的DEBUG = false 时生效）



### 7、django自带的邮箱警告

settings.py

```
DEBUG = False	#关闭调试模式

ADMINS = [('houzi','xxx@qq.com'),('houzi2','xxx@qq.com')]	#错误接收方

SERVER_EMAIL = 'email配置中的邮箱'	#发送错误报告方，默认为 root@localhost 账户，多数邮件服务器会拒绝
```



