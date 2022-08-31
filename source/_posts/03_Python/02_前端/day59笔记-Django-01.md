---
title: Day59-Django-01
date: 2022-08-25 14:44:22
categories:
- Python
- 前端基础
tags:
---

“第59天Django-01学习笔记”

# 1 手写Web框架

在学Web框架之前，首先得知道什么是Web框架

```python
# HTTP协议
"""
网络协议
HTTP协议				数据传输是明文
HTTPS协议				数据传输是密文
websocket协议		数据传输是密文


四大特性
	1.基于请求响应
	2.基于TCP、IP作用于应用层之上的协议
	3.无状态
	4.短/无链接

数据格式
	请求首行
	请求头
	
	请求体

响应状态码
	1XX
	2XX			200
	3XX			
	4XX			403 404
	5XX			500
"""
```

## 1.1 纯手写的Web框架

案例代码如下：

```python
# 你可以将web框架理解成服务端
import socket


server = socket.socket()  # TCP  三次握手四次挥手  osi七层
server.bind(('127.0.0.1',8080))  # IP协议 以太网协议 arp协议...
server.listen(5)  # 池 ...

"""
b'GET / HTTP/1.1\r\n
Host: 127.0.0.1:8082\r\n
Connection: keep-alive\r\n
Upgrade-Insecure-Requests: 1\r\n
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36\r\n
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n
Sec-Fetch-Site: none\r\n
Sec-Fetch-Mode: navigate\r\n
Sec-Fetch-User: ?1\r\n
Sec-Fetch-Dest: document\r\n
Accept-Encoding: gzip, deflate, br\r\n
Accept-Language: zh-CN,zh;q=0.9\r\n
Cookie: csrftoken=KYJnVBLPpJxwt09TOmTXzpb5qkFJwHVxVGpi0NxEGIg4z5VUuazZ1O2RMwSisu14\r\n
\r\n'


"""
while True:
    conn, addr = server.accept()
    data = conn.recv(1024)
    # print(data)  # 二进制数据
    data = data.decode('utf-8')  # 字符串
    # 获取字符串中特定的内容       正则  如果字符串有规律也可以考虑用切割
    conn.send(b'HTTP/1.1 200 OK\r\n\r\n')
    current_path = data.split(' ')[1]
    # print(current_path)
    if current_path == '/index':
        # conn.send(b'index heiheihei')
        with open(r'templates/01 myhtml.html', 'rb') as f:
            conn.send(f.read())
    elif current_path == '/login':
        conn.send(b'login')
    else:
        # 你直接忽略favicon.ico
        conn.send(b'hello web')
    conn.close()
```

纯手撸的web框架有以下不足：

1. 代码重复(服务端代码所有人都要重复写)
2.  手动处理http格式的数据 并且只能拿到url后缀 其他数据获取繁琐(数据格式一样处理的代码其实也大致一样 重复写)
3.  并发的问题

## 1.2 使用wsgiref模块

既然纯手写的Web框架存在褚多问题，可以使用wsgiref模块来改善

案例代码如下:

```python
from wsgiref.simple_server import make_server
from urls import urls
from views import *


def run(env, response):
    """
    :param env:请求相关的所有数据
    :param response:响应相关的所有数据
    :return: 返回给浏览器的数据
    """
    # print(env)  # 大字典  wsgiref模块帮你处理好http格式的数据 封装成了字典让你更加方便的操作
    # 从env中取
    response('200 OK', [])  # 响应首行 响应头
    current_path = env.get('PATH_INFO')
    # if current_path == '/index':
    #     return [b'index']
    # elif current_path == '/login':
    #     return [b'login']
    # return [b'404 error']
    # 定义一个变量 存储匹配到的函数名
    func = None
    for url in urls:  # url (),()
        if current_path == url[0]:
            # 将url对应的函数名赋值给func
            func = url[1]
            break  # 匹配到一个之后 应该立刻结束for循环
    # 判断func是否有值
    if func:
        res = func(env)
    else:
        res = error(env)

    return [res.encode('utf-8')]


if __name__ == '__main__':
    server = make_server('127.0.0.1',8080,run)
    """
    会实时监听127.0.0.1:8080地址 只要有客户端来了
    都会交给run函数处理(加括号触发run函数的运行)
    
    flask启动源码
        make_server('127.0.0.1',8080,obj)
        __call__
    """
    server.serve_forever()  # 启动服务端
```

使用wsgi需要特定的文件夹格式：

```python
"""
urls.py						路由与视图函数对应关系
views.py					视图函数(后端业务逻辑)
templates文件夹		专门用来存储html文件
"""
# 按照功能的不同拆分之后 后续添加功能只需要在urls.py书写对应关系然后取views.py书写业务逻辑即可
```

其中urls.py的内容如下：

```python
from views import *

# url与函数的对应关系
urls = [
    ('/index',index),
    ('/login',login),
    ('/xxx',xxx),
    ('/get_time',get_time),
    ('/get_dict',get_dict),
    ('/get_user',get_user)
]
```

views.py的内容如下：

```python
def index(env):
    return 'index'


def login(env):
    return "login"


def error(env):
    return '404 error'


def xxx(env):
    with open(r'templates/02 myxxx.html','r',encoding='utf-8') as f:
        return f.read()

import datetime
def get_time(env):
    current_time = datetime.datetime.now().strftime('%Y-%m-%d %X')
    # 如何将后端获取到的数据"传递"给html文件？
    with open(r'templates/03 mytime.html','r',encoding='utf-8') as f:
        data = f.read()
        # data就是一堆字符串
    data = data.replace('dwadasdsadsadasdas',current_time)   # 在后端将html页面处理好之后再返回给前端
    return data


from jinja2 import Template
def get_dict(env):
    user_dic = {'username':'jason','age':18,'hobby':'read'}
    with open(r'templates/04 get_dict.html','r',encoding='utf-8') as f:
        data = f.read()
    tmp = Template(data)
    res = tmp.render(user=user_dic)
    # 给get_dict.html传递了一个值 页面上通过变量名user就能够拿到user_dict
    return res


import pymysql
def get_user(env):
    # 去数据库中获取数据 传递给html页面 借助于模版语法 发送给浏览器
    conn = pymysql.connect(
        host = '127.0.0.1',
        port = 3306,
        user = 'root',
        password = 'admin123',
        db='day59',
        charset = 'utf8',
        autocommit = True
    )
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    sql = 'select * from userinfo'
    affect_rows = cursor.execute(sql)
    data_list = cursor.fetchall()  # [{},{},{}]
    # 将获取到的数据传递给html文件
    with open(r'templates/05 get_data.html','r',encoding='utf-8') as f:
        data = f.read()
    tmp = Template(data)
    res = tmp.render(user_list=data_list)
    # 给get_dict.html传递了一个值 页面上通过变量名user就能够拿到user_dict
    return res


if __name__ == '__main__':
    get_user(111)
```

## 1.3 动态网页和静态网页

什么是动态网页？什么又是静态网页？它们的区别是什么？

- 静态网页
  - 页面上的数据是直接写死的(万年不变)
- 动态网页
  - 数据是实时获取的
  - 例如：
    - 后端获取当前时间展示到html页面上
    - 数据是从数据库中获取的展示到html页面上

动态网页的案例如下：

```python
# 动态网页制作
import datetime
def get_time(env):
    current_time = datetime.datetime.now().strftime('%Y-%m-%d %X')
    # 如何将后端获取到的数据"传递"给html文件？
    with open(r'templates/03 mytime.html','r',encoding='utf-8') as f:
        data = f.read()
        # data就是一堆字符串
    data = data.replace('dwadasdsadsadasdas',current_time)   # 在后端将html页面处理好之后再返回给前端
    return data

# 将一个字典传递给html文件 并且可以在文件上方便快捷的操作字典数据
from jinja2 import Template
def get_dict(env):
    user_dic = {'username':'jason','age':18,'hobby':'read'}
    with open(r'templates/04 get_dict.html','r',encoding='utf-8') as f:
        data = f.read()
    tmp = Template(data)
    res = tmp.render(user=user_dic)
    # 给get_dict.html传递了一个值 页面上通过变量名user就能够拿到user_dict
    return res

# 后端获取数据库中数据展示到前端页面
```





-----------------



### 自定义简易版本web框架请求流程图

```python
"""
wsgiref模块
	1.请求来的时候解析http格式的数据 封装成大字典
	2.响应走的时候给数据打包成符合http格式 再返回给浏览器

"""
```

### python三大主流web框架

```python
"""
django
	特点:大而全 自带的功能特别特别特别的多 类似于航空母舰
	不足之处:
		有时候过于笨重

flask
	特点:小而精  自带的功能特别特别特别的少 类似于游骑兵
	第三方的模块特别特别特别的多，如果将flask第三方的模块加起来完全可以盖过django
	并且也越来越像django
	不足之处:
		比较依赖于第三方的开发者
		
tornado
	特点:异步非阻塞 支持高并发
		牛逼到甚至可以开发游戏服务器
	不足之处:
		暂时你不会
"""
A:socket部分
B:路由与视图函数对应关系(路由匹配)
C:模版语法

django
	A用的是别人的		wsgiref模块
  B用的是自己的
  C用的是自己的(没有jinja2好用 但是也很方便)

flask
	A用的是别人的		werkzeug(内部还是wsgiref模块)
  B自己写的
  C用的别人的(jinja2)

tornado
	A，B，C都是自己写的
```

### 注意事项

```python
# 如何让你的计算机能够正常的启动django项目
	1.计算机的名称不能有中文
  2.一个pycharm窗口只开一个项目
  3.项目里面所有的文件也尽量不要出现中文
  4.python解释器尽量使用3.4~3.6之间的版本
  	(如果你的项目报错 你点击最后一个报错信息
    去源码中把逗号删掉)
    
# django版本问题
	1.X 2.X 3.X(直接忽略)
  1.X和2.X本身差距也不大 我们讲解主要以1.X为例 会讲解2.X区别
  公司之前用的1.8 满满过渡到了1.11版本 有一些项目用的2.0
 
# django安装
	pip3 install django==1.11.11
  如果已经安装了其他版本 无需自己卸载
  直接重新装 会自动卸载安装新的
  
  如果报错 看看是不是timeout 如果是 那么只是网速波动
  重新安装即可
  
  验证是否安装成功的方式1
  	终端输入django-admin看看有没有反应
```

### django基本操作

```python
# 命令行操作
	# 1.创建django项目
  	"""
  	你可以先切换到对应的D盘 然后再创建
  	"""
  	django-admin startproject mysite
    
    	mysite文件夹
      	manage.py
      	mysite文件夹
        	__init__.py
        	settings.py
          urls.py
          wsgi.py
 # 2.启动django项目
	"""
		一定要先切换到项目目录下	
		cd /mysite
	"""
  python3 manage.py runserver
  # http://127.0.0.1:8000/
 
# 3.创建应用
"""
Next, start your first app by running python manage.py startapp [app_label].
"""
	python manage.py startapp app01
    应用名应该做到见名知意
      user
      order
      web
      ...
      但是我们教学统一就用app01/02/03/04
      
	有很多文件
  
# pycharm操作
	# 1 new project 选择左侧第二个django即可
  
  # 2 启动
  		1.还是用命令行启动
    	2.点击绿色小箭头即可

  # 3 创建应用
  		1.pycharm提供的终端直接输入完整命令
    	2.pycharm 
      		tools 
        		run manage.py task提示(前期不要用 给我背完整命令)
 # 4 修改端口号以及创建server	
		edit confi....
  
 
```

### 应用

```python
"""
django是一款专门用来开发app的web框架

django框架就类似于是一所大学(空壳子)
app就类似于大学里面各个学院(具体功能的app)
	比如开发淘宝
		订单相关
		用户相关
		投诉相关
		创建不同的app对应不同的功能
	
	选课系统
		学生功能
		老师功能

一个app就是一个独立的功能模块
"""
***********************创建的应用一定要去配置文件中注册**********************
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app01.apps.App01Config',  # 全写
  	'app01',			 # 简写
]
# 创建出来的的应用第一步先去配置文件中注册 其他的先不要给我干
ps:你在用pycharm创建项目的时候 pycharm可以帮你创建一个app并且自动注册
***********************************************************************
```

### 主要文件介绍

```python
-mysite项目文件夹
	--mysite文件夹
  	---settings.py	配置文件
    ---urls.py			路由与视图函数对应关系(路由层)
    ---wsgi.py			wsgiref模块(不考虑)
  --manage.py				django的入口文件
  --db.sqlite3			django自带的sqlite3数据库(小型数据库 功能不是很多还有bug)
  --app01文件夹
  	---admin.py			django后台管理
    ---apps.py			注册使用
    ---migrations文件夹		数据库迁移记录
    ---models.py		数据库相关的 模型类(orm)
  	---tests.py			测试文件
    ---views.py			视图函数(视图层)
```

### 命令行与pycharm创建的区别

```python
# 1 命令行创建不会自动有templatew文件夹 需要你自己手动创建而pycharm会自动帮你创建并且还会自动在配置文件中配置对应的路径
# pycharm创建
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
]
# 命令行创建
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
]
"""
也就意味着你在用命令创建django项目的时候不单单需要创建templates文件夹还需要去配置文件中配置路径
'DIRS': [os.path.join(BASE_DIR, 'templates')]
"""
```

### django小白必会三板斧

```python
"""
HttpResponse
	返回字符串类型的数据

render
	返回html文件的

redirect
	重定向
	  return redirect('https://www.mzitu.com/')
    return redirect('/home/')
"""
```

### 作业

```python
"""
1.整理web框架推导思路
2.安装django并正常启动访问，测试三板斧
3.整理今日日考题，django内容
选做题
1.结合前端，django，MySQL，pymysql模块实现数据库数据动态展示到前端
2.尝试着摸索django模版语法
"""
```

