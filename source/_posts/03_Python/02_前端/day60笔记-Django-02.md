---
title: Day59-Django-01
date: 2022-09-02 14:44:22
categories:
- Python
- 前端基础
tags:
---

“第60天Django-02学习笔记”

# 1 静态文件配置

## 1.1 静态文件介绍

什么是静态文件？

​	前端已经写好了的 能够直接调用使用的文件，如

- 网站写好的js文件
- 网站写好的css文件
- 网站用到的图片文件
- 第三方前端框架

我们将网站使用的所有`静态文件`默认都放在`static`文件夹中

<font color=red>**static文件夹默认没有，需要手动创建**</font>

```python
一般情况下我们在static文件夹内还会做进一步的划分处理
	-static
  	--js
    --css
    --img
    其他第三方文件
    
"""
在浏览器中输入url能够看到对应的资源
是因为后端提前开设了该资源的接口
如果访问不到资源 说明后端没有开设该资源的接口

http://127.0.0.1:8000/static/bootstrap-3.3.7-dist/css/bootstrap.min.css
"""
```

## 1.2 静态文件配置

### 1.2.1 回顾创建一个普通Django项目

如何在Django项目中设置静态文件？

我们先创建一个普通的Django项目

1、先在Pycharm创建Django项目-`day60_Django02`

2、打开terminal窗口，创建app01

```shell
python manage.py startapp app01
```

3、注册app01应用

编辑setting.py文件

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 注册应用
    'app01',
]
```

4、配置`urls.py`文件和`app01/views.py`文件

```python
from django.contrib import admin
from django.urls import path
from app01 import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('login/', views.login),
]
```

```python
# app01/views.py内容
from django.shortcuts import render

# Create your views here.
def login(request):
    return render(request, 'test.html')
```

5、创建HTML文件`templates/test.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>这是测试文件</h1>
</body>
</html>
```

6、运行项目

访问`http://127.0.0.1:8000/login`可见到`test.html`页面的内容

### 1.2.2 给这个项目配置静态文件

使用静态文件需要先创建`static`文件夹，并将静态文件都放到里面

1、创建`src01`文件夹，把下载并解压的`bootstrap-5.2.0-dist`文件夹放进去

```shell
$ ls static/src01/
bootstrap-5.2.0-dist
```

2、设置静态文件的`访问路径`和`存储路径`

编辑settings.py文件

```shell
# 配置静态文件的访问路径
STATIC_URL = '/bbc/'
# 配置静态文件的存储路径，会在三个文件夹依次寻找
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'src01'),
    os.path.join(BASE_DIR, 'src02'),
    os.path.join(BASE_DIR, 'src03'),
]
```

3、静态文件动态解析

给html文件添加动态解析代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>这是测试文件</h1>
    <button type="button" class="btn btn-primary">Primary</button>

    {% load static %}
    <link rel="stylesheet" href="{% static 'bootstrap-5.2.0-dist/css/bootstrap.min.css' %}">
    <script src="{% static 'bootstrap-5.2.0-dist/js/bootstrap.min.js' %}"></script>
</body>
</html>
```

4、查看效果

可见Bootstrap已生效

![image-20220905185920419](../../../img/image-20220905185920419.png)

### 1.3 访问路径和存储路径

有个容易迷惑的问题，什么是访问路径？什么又是存储路径？

**访问路径**：

​	通过浏览器访问静态文件的路径，如图：

![image-20220905190223177](../../../img/image-20220905190223177.png)

**存储路径：**

​	静态文件实际存放的路径，如图：

![image-20220905190333652](../../../img/image-20220905190333652.png)



# 2 request对象方法认识

将第一章中的`test.html`修改为一个form表单页面

> 如需获取数据，需要`name`属性

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1 class="text-center">欢迎来到的德莱联盟，请注册！</h1>
    <form action="" method="post">
        <div class="container">
            <div class="row">
                <div class="col-md-8 offset-md-2">
                    <div>用户名
                        <input  class="form-control" name="username" type="text">
                    </div>
                    <div>密码
                        <input  class="form-control" name="password" type="password">
                    </div>
                    <div>性别
                        <select name="gender">
                            <option value="1">男</option>
                            <option value="0">女</option>
                        </select>
                    </div>
                    <input type="submit" class="btn btn-primary form-control" value="注册">
                 </div>
            </div>
        </div>
    </form>


    {% load static %}
    <link rel="stylesheet" href="{% static 'bootstrap-5.2.0-dist/css/bootstrap.min.css' %}">
    <script src="{% static 'bootstrap-5.2.0-dist/js/bootstrap.min.js' %}"></script>
</body>
</html>
```

界面效果如下：

![image-20220905200230475](../../../img/image-20220905200230475.png)

## 2.1 GET和POST的区别

Form表单主要有两个参数，`action`和`method`

action参数：

​	1.不写 默认朝当前所在的url提交数据

​	2.全写 指名道姓

​	3.只写后缀 /login/

method参数：

​	1、get

​	2、post

本节主要介绍`GET`和`POST`的区别：

设置GET跟POST是通过html代码中的`method`指定，如:

```html
<form action="" method="GET">
...
</form>
```

### 2.1.1 GET

GET请求会在submit时，明文传递数据，浏览器上会显示，如图

![image-20220905222449310](../../../img/image-20220905222449310.png)

### 2.1.2 POST

而POST请求在submit的时候，传递的是密文数据，浏览器上不会显示，如图

![image-20220905223427713](../../../img/image-20220905223427713.png)



<font color=red>注意：</font>

在前期我们使用django提交post请求的时候 需要取配置文件中注释掉一行代码

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

否则，submit提交后会出现`403`界面

![image-20220905203914884](../../../img/image-20220905203914884.png)

具体报错信息是：

```python
Quit the server with CTRL-BREAK.
[05/Sep/2022 20:37:32] "GET /login/? HTTP/1.1" 200 1175
Forbidden (CSRF cookie not set.): /login/
[05/Sep/2022 20:37:35] "POST /login/? HTTP/1.1" 403 2870
```

### 2.1.3 通过request获取请求信息

可以获取如下信息:

```python
request.method # 返回请求方式 并且是全大写的字符串形式  <class 'str'>
request.POST  # 获取用户post请求提交的普通数据不包含文件
	request.POST.get()  # 只获取列表最后一个元素
  request.POST.getlist()  # 直接将列表取出
request.GET  # 获取用户提交的get请求数据
	request.GET.get()  # 只获取列表最后一个元素
  request.GET.getlist()  # 直接将列表取出
"""
get请求携带的数据是有大小限制的 大概好像只有4KB左右
而post请求则没有限制
"""
```

使用方法如下：

```python
# app01/views.py文件
from django.shortcuts import render


# Create your views here.
def login(request):
    print(request.POST)
    return render(request, 'test.html')

# 显示
<QueryDict: {'username': ['test'], 'password': ['qwer123'], 'gender': ['1']}>
```

也可以用于GET和POST的判断：

```python
def login(request):
    # 返回一个登陆界面
    """
    get请求和post请求应该有不同的处理机制
    :param request: 请求相关的数据对象 里面有很多简易的方法
    :return:
    """
    # print(type(request.method))  # 返回请求方式 并且是全大写的字符串形式  <class 'str'>
    # if request.method == 'GET':
    #     print('来了 老弟')
    #     return render(request,'login.html')
    # elif request.method == 'POST':
    #     return HttpResponse("收到了 宝贝")
    
    if request.method == 'POST':
        return HttpResponse("收到了 宝贝")
    return render(request, 'login.html')
```



# 3 Pycharm和Django连接数据库的方法