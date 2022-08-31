---
title: Day58-Bootstrap
date: 2022-08-25 14:44:22
categories:
- Python
- 前端基础
tags:
---

“第58天Bootstrap学习笔记”

# 1 什么是Bootstrap

<部分没有练习，但已知道用途，后续可按实际需要使用>

该框架已经帮你写好了很多页面样式，你如果需要使用，只需要下载它对应文件，之后直接cv拷贝即可

在使用Bootstrap的时候所有的页面样式都只需要你通过class来调节即可

版本选择建议使用v3版本：<https://v3.bootcss.com/>

><font color=red>--注意--</font>
>
>**bootstrap的js代码是依赖于jQuery的，也就意味着你在使用Bootstrap动态效果的时候，一定要导入jQuery**

# 2 使用Bootstrap

## 2.1 布局容器

```python
<div class="container">
    	左右两侧有留白
</div>

<div class="container-fluid">
			左右两侧没有留白
</div>
# 后续在使用bootstrap做页面的时候 上来先写一个div class=container,之后在div内部书写页面
```

##  2.2 栅格系统

```python
<div class="row"></div>
写一个row就是将所在的区域划分成12份

<div class="col-md-6 ">  获取你所要的份数
# 在使用bootstrap的时候 脑子里面一定要做12的加减法
```

## 2.3 栅格参数

```python
.col-xs-	.col-sm-	.col-md-	.col-lg-
# 针对不同的显示器 bootstrap会自动选择对应的参数
# 如果你想要兼容所有的显示器 你就全部加上即可


# 在一行如何移动位置
<div class="col-md-8 c1 col-md-offset-2"></div>
```

## 2.4 排版

bootstrap将所有原生的HTML标签的文本字体统一设置成了肉眼可以接受的样式

效果一样，但是标签表达的意思不一样（语义）

## 2.5 表格

```python
<table class="table table-hover table-striped table-bordered">
		
<tr class="success">
            <td>1</td>
            <td>jason</td>
            <td>123</td>
            <td>study</td>
</tr>

<tr class="active">...</tr>
<tr class="success">...</tr>
<tr class="warning">...</tr>
<tr class="danger">...</tr>
<tr class="info">...</tr>
```

## 2.6 表单

```python
<div class="container">
    <div class="col-md-8 col-md-offset-2">
        <h2 class="text-center">登陆页面</h2>
        <form action="">
            <p>username:<input type="text" class="form-control"></p>
            <p>password:<input type="text" class="form-control"></p>
            <p>
                <select name="" id="" class="form-control">
                    <option value="">111</option>
                    <option value="">222</option>
                    <option value="">333</option>
                </select>
            </p>
            <textarea name="" id="" cols="30" rows="10" class="form-control"></textarea>
            <input type="submit">
        </form>
    </div>
</div>

# 针对表单标签 加样式就用form-control
	class="form-control"
"""
<input type="checkbox">222
<input type="radio">333
checkbox和radio我们一般不会给它加form-control，直接使用原生的即可
"""

# 针对报错信息 可以加has-error（input的父标签加）
<p class="has-error">
	username:
  <input type="text" class="form-control">
</p>
```

## 2.7 按钮

```python
<a href="https://www.mzitu.com/" class="btn btn-primary">点我</a>
<button class="btn btn-danger">按我</button>
<button class="btn btn-default">按我</button>
<button class="btn btn-success">按我</button>
<button class="btn btn-info">按我</button>
<button class="btn btn-warning">按我</button>


<button class="btn btn-warning btn-lg">按我</button>
<button class="btn btn-warning btn-sm">按我</button>
<button class="btn btn-warning btn-xs">按我</button>
<input type="submit" class="btn btn-primary btn-block">  
通过给按钮添加 .btn-block 类可以将其拉伸至父元素100%的宽度，而且按钮也变为了块级（block）元素。
```

## 2.8 图表

```python
<h2 class="text-center">登陆页面 <span class="glyphicon glyphicon-user"></span></h2>


    <style>
        span {
            color: greenyellow;
        }
    </style>

# 扩展
```

## 2.9 导航条

```python
<nav class="navbar navbar-inverse">
<nav class="navbar navbar-default">
```

## 2.10 分页器

```python
<nav aria-label="Page navigation">
  <ul class="pagination">
    <li>
      <a href="#" aria-label="Previous">
        <span aria-hidden="true">&laquo;</span>
      </a>
    </li>
    <li class="active"><a href="#">1</a></li>
    <li><a href="#">2</a></li>
    <li><a href="#">3</a></li>
    <li><a href="#">4</a></li>
    <li><a href="#">5</a></li>
    <li>
      <a href="#" aria-label="Next">
        <span aria-hidden="true">&raquo;</span>
      </a>
    </li>
  </ul>
</nav>
```

## 2.11 弹框

```python
https://lipis.github.io/bootstrap-sweetalert/
  
  
swal('你还好吗?')
undefined
swal('你还好吗?')
undefined
swal('你还好吗?','我不好，想你了!')
undefined
swal('你还好吗?','我不好，想你了!','success')
undefined
swal('你还好吗?','我不好，想你了!','warning')
undefined
swal('你还好吗?','我不好，想你了!','error')
undefined
swal('你还好吗?','我不好，想你了!','info')
undefined
# 我们在后面的课程中 还会涉及到该部分内容
```

