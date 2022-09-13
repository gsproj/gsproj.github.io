---
title: Day62-Django-03
date: 2022-09-12 14:44:22
categories:
- Python
- 09_Django
tags:
---

“Day62 Diango 03 学习笔记”



# 一、路由层（接day61）

## 1.2 路由分发

### 1.2.1 什么是路由分发？

​	将所有的url都写在Django项目总的`urls.py`中，写多了杂乱，不好维护

​	但是Django有个特点：每一个应用都可以有自己的templates文件夹、urls.py、static文件夹

​	基于上述的特点 django能够非常好的做到分组开发(每个人只写自己的app)

利用路由分发来减轻总路由的压力，总路由统领各app里面的路由

### 1.2.2 路由分发案例

1）新项目，创建两个app

```shell
python .\manage.py startapp app01
python .\manage.py startapp app02
```

2）两个app配置子路由`urls.py`文件

```python
# app01
from django.urls import path
from app01 import views

urlpatterns = [
    path('reg/', views.reg),
]

# app02
from django.urls import path
from app02 import views

urlpatterns = [
    path('reg/', views.reg),
]
```

3）两个app的views文件修改

```python
# app01
from django.shortcuts import render, HttpResponse

# Create your views here.
def reg(request):
    return HttpResponse("app01:reg")

# app02
from django.shortcuts import render, HttpResponse

# Create your views here.
def reg(request):
    return HttpResponse("app01:reg")
```

4）总路由`urls.py`配置路由分发

```python
from django.contrib import admin
from django.conf.urls import url, include
# 需要导入模块
from django.urls import path, re_path
from app01 import urls as app01_urls
from app02 import urls as app02_urls

urlpatterns = [
    path('admin/', admin.site.urls),
    # 凡是app01开头的，都交给app01子路由处理
    re_path(r'^app01/', include(app01_urls)),
    # 凡是app02开头的，都交给app02子路由处理
    re_path(r'^app02/', include(app02_urls)),
]
```

更推荐的写发，不需要重命名：

```python
from django.contrib import admin
from django.conf.urls import url, include
# 需要导入模块
from django.urls import path, re_path

urlpatterns = [
    path('admin/', admin.site.urls),
    # 凡是app01开头的，都交给app01子路由处理
    re_path(r'^app01/', include('app01.urls')),
    # 凡是app02开头的，都交给app02子路由处理
    re_path(r'^app02/', include('app02.urls')),
]
```

测试访问成功：

![image-20220913193027730](../../../img/image-20220913193027730.png)



## 1.3 名称空间(了解)

```python
# 当多个应用出现了相同的别名 我们研究反向解析会不会自动识别应用前缀
"""
正常情况下的反向解析是没有办法自动识别前缀的
"""

# 名称空间
	# 总路由
    url(r'^app01/',include('app01.urls',namespace='app01')),
    url(r'^app02/',include('app02.urls',namespace='app02'))
  # 解析的时候
  	# app01
  	urlpatterns = [
    url(r'^reg/',views.reg,name='reg')
		]
    # app02
    urlpatterns = [
    url(r'^reg/',views.reg,name='reg')
		]
    
  	reverse('app01:reg')
    reverse('app02:reg')
    
    {% url 'app01:reg' %}
    {% url 'app02:reg' %}
# 其实只要保证名字不冲突 就没有必要使用名称空间
"""
一般情况下 有多个app的时候我们在起别名的时候会加上app的前缀
这样的话就能够确保多个app之间名字不冲突的问题
"""
urlpatterns = [
    url(r'^reg/',views.reg,name='app01_reg')
]
urlpatterns = [
    url(r'^reg/',views.reg,name='app02_reg')
]
```

## 1.4 伪静态(了解)

```python
"""
静态网页
	数据是写死的 万年不变
	
伪静态
	将一个动态网页伪装成静态网页
	
	为什么要伪装呢？
		https://www.cnblogs.com/Dominic-Ji/p/9234099.html
		伪装的目的在于增大本网站的seo查询力度
		并且增加搜索引擎收藏本网上的概率
	
	搜索引擎本质上就是一个巨大的爬虫程序
	
	总结:
		无论你怎么优化 怎么处理
		始终还是干不过RMB玩家
"""
urlpatterns = [
    url(r'^reg.html',views.reg,name='app02_reg')
]
```

## 1.5 虚拟环境(了解)

```python
"""
在正常开发中 我们会给每一个项目配备一个该项目独有的解释器环境
该环境内只有该项目用到的模块 用不到一概不装

linux:缺什么才装什么

虚拟环境
	你每创建一个虚拟环境就类似于重新下载了一个纯净的python解释器
	但是虚拟环境不要创建太多，是需要消耗硬盘空间的

扩展:
	每一个项目都需要用到很多模块 并且每个模块版本可能还不一样
	那我该如何安装呢？ 一个个看一个个装？？？
	
	开发当中我们会给每一个项目配备一个requirements.txt文件
	里面书写了该项目所有的模块即版本
	你只需要直接输入一条命令即可一键安装所有模块即版本
"""
```

## 1.6 django版本区别

```python
"""
1.django1.X路由层使用的是url方法
	而在django2.Xhe3.X版本中路由层使用的是path方法
	url()第一个参数支持正则
	path()第一个参数是不支持正则的 写什么就匹配什么
	
	
	如果你习惯使用path那么也给你提供了另外一个方法
		from django.urls import path, re_path
		from django.conf.urls import url
		
		re_path(r'^index/',index),
    url(r'^login/',login)
  2.X和3.X里面的re_path就等价于1.X里面的url
 
 
2.虽然path不支持正则 但是它的内部支持五种转换器
	path('index/<int:id>/',index)
	# 将第二个路由里面的内容先转成整型然后以关键字的形式传递给后面的视图函数

	def index(request,id):
    print(id,type(id))
    return HttpResponse('index')
    
  
  
  str,匹配除了路径分隔符（/）之外的非空字符串，这是默认的形式
	int,匹配正整数，包含0。
	slug,匹配字母、数字以及横杠、下划线组成的字符串。
	uuid,匹配格式化的uuid，如 075194d3-6885-417e-a8a8-6c931e272f00。
	path,匹配任何非空字符串，包含了路径分隔符（/）（不能用？）
	
3.除了有默认的五个转换器之外 还支持自定义转换器(了解)
	class MonthConverter:
    regex='\d{2}' # 属性名必须为regex

    def to_python(self, value):
        return int(value)

    def to_url(self, value):
        return value # 匹配的regex是两个数字，返回的结果也必须是两个数字
	
	
	from django.urls import path,register_converter
	from app01.path_converts import MonthConverter

	# 先注册转换器
	register_converter(MonthConverter,'mon')

	from app01 import views


	urlpatterns = [
    path('articles/<int:year>/<mon:month>/<slug:other>/', 	views.article_detail, name='aaa'),

]


4.模型层里面1.X外键默认都是级联更新删除的
但是到了2.X和3.X中需要你自己手动配置参数
	models.ForeignKey(to='Publish')
	
	models.ForeignKey(to='Publish',on_delete=models.CASCADE...)
"""
```

# 二、视图层

## 2.1 三板斧

```python
"""
HttpResponse
	返回字符串类型
render
	返回html页面 并且在返回给浏览器之前还可以给html文件传值
redirect
	重定向
"""
# 视图函数必须要返回一个HttpResponse对象  正确   研究三者的源码即可得处结论
The view app01.views.index didn't return an HttpResponse object. It returned None instead.

# render简单内部原理
		from django.template import Template,Context
    res = Template('<h1>{{ user }}</h1>')
    con = Context({'user':{'username':'jason','password':123}})
    ret = res.render(con)
    print(ret)
    return HttpResponse(ret)

```

## 2.2 JsonResponse对象

```python
"""
json格式的数据有什么用？
	前后端数据交互需要使用到json作为过渡 实现跨语言传输数据

前端序列化
	JSON.stringify()					json.dumps()
	JSON.parse()							json.loads()
"""
import json
from django.http import JsonResponse
def ab_json(request):
    user_dict = {'username':'jason好帅哦,我好喜欢!','password':'123','hobby':'girl'}

    l = [111,222,333,444,555]
    # 先转成json格式字符串
    # json_str = json.dumps(user_dict,ensure_ascii=False)
    # 将该字符串返回
    # return HttpResponse(json_str)
    # 读源码掌握用法
    # return JsonResponse(user_dict,json_dumps_params={'ensure_ascii':False})
    # In order to allow non-dict objects to be serialized set the safe parameter to False.
    # return JsonResponse(l,safe=False)  
    # 默认只能序列化字典 序列化其他需要加safe参数	
```

## 2.3 form表单上传文件及后端如何操作

form表单上传文件类型的数据

1. method必须指定成post
2. enctype必须换成formdata

### 2.3.1 案例：

HTML页面代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
	# 必须设置
    <form action="" method="post" enctype="multipart/form-data">
        <p>选择文件：<input type="file" name="file"></p>
        <input type="submit">
    </form>
</body>
</html>
```

views.py代码

```python
from django.shortcuts import render, HttpResponse


# Create your views here.
def reg(request):
    if request.method == "POST":
        # print(request.POST)  # 只能获取普通的简直对数据 文件不行
        print(request.FILES)
        file_obj = request.FILES.get('file')  # 文件对象
        print(file_obj.name)
        with open(file_obj.name, 'wb') as f:
            for line in file_obj.chunks():  # 推荐加上chunks方法 其实跟不加是一样的都是一行行的读取
                f.write(line)
    return render(request, 'index.html')

```

测试：页面上传文件后，会保存到项目文件夹中

## 2.4 request对象方法

```python
"""
request.method
request.POST
request.GET
request.FILES
request.body  # 原生的浏览器发过来的二进制数据  后面详细的讲
request.path 
request.path_info
request.get_full_path()  能过获取完整的url及问号后面的参数 
"""
print(request.path)  # /app01/ab_file/
print(request.path_info)  # /app01/ab_file/
print(request.get_full_path())  # /app01/ab_file/?username=jason
```

## 2.5 FBV与CBV

```python
# 视图函数既可以是函数也可以是类
def index(request):
  return HttpResponse('index')

# CBV
    # CBV路由
    url(r'^login/',views.MyLogin.as_view())


    from django.views import View


    class MyLogin(View):
        def get(self,request):
            return render(request,'form.html')

        def post(self,request):
            return HttpResponse('post方法')
      
"""
FBV和CBV各有千秋
CBV特点
	能够直接根据请求方式的不同直接匹配到对应的方法执行
	
	内部到底是怎么实现的？
		CBV内部源码(******)
"""
```

