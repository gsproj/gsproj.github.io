---
title: Day56-JQuery-01
date: 2022-08-22 15:44:22
categories:
- Python
- Python入门
tags:
---

“第56天JQuery（01）学习笔记”

# 1 JQuery介绍

## 1.1 简介

什么是JQuery？

- jQuery内部封装了原生的js代码(还额外添加了很多的功能)
- 能够让你通过书写更少的代码 完成js操作 
- 类似于python里面的模块  在前端模块不叫模块  叫 “类库”

JQuery的宗旨：

​	write less do more

JQuery选择：

​	jquery.js	未压缩的版本

​	jquery.min.js	压缩后的版本（推荐）

## 1.2 导入JQuery

导入JQuery有两种方式

### 1.2.1 本地方式导入：

到JQuery官网获取JQuery的JS文件

```javascript
https://jquery.com/
```

在HTML中导入文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="js/jquery-3.6.0.min.js"></script>
</head>
<body>
    
</body>
</html>
```

### 1.2.2 网络CDN导入(需要网络)

前端免费的cdn网站: 

```javascript
https://www.bootcdn.cn/
```

导入方法：

```html
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
```



## 1.3 JQuery的简单使用

### 1.3.1 基本语法

使用JQuery有两种方式：

第一种

```javascript
jQuery(选择器).action()
```

第二种

```javascript
$(选择器).action()
```

两种方法是等价的，推荐<font color=red>第二种</font>



### 1.3.2 JQuery与JavaScript的语法对比

通过案例可见，实现相同的功能，使用JQuery后，代码量确实可以减少

```javascript
// JS原生代码
d1Ele = document.getElementById('d1')
d1Ele.style.color = 'red'

// 使用JQuery
$('#d1').css('color','red')
```



# 2 JQuery的使用

## 2.1 标签查找

通过下面的案例，演示标签查找

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="jquery-3.6.0.min.js"></script>
</head>
<body>
    <div id="d1">d1div
        <span>d1div-span</span>
    </div>
    <div class="c1">c1div</div>
    <span>span</span>
</body>
</html>
```

id选择器

```javascript
$('#d1')
S.fn.init [div#d1]	// 找到一个JQuery对象
```

class选择器

```javascript
$('.c1')
S.fn.init [div.c1, prevObject: S.fn.init(1)]	// 找到一个JQuery对象
```

标签选择器

```javascript
$('span')
S.fn.init(2) [span, span, prevObject: S.fn.init(1)]		// 找到两个JQuery对象
```

**<font color=blue>发现我们查找到的并不是标签，而是`JQuery对象`，那么如何才能将`JQuery对象`转换成`标签对象`呢？</font>**

JQuery对象和标签对象的互相转换

```javascript
// JQuery对象转换成标签对象
$('#d1')[0]
<div id=​"d1">​…​</div>​"d1div "<span>​d1div-span​</span>​</div>​

// 标签对象转换成JQuery对象
$(document.getElementById('d1'))
S.fn.init [div#d1]
```



## 2.2 组合选择器/分组与嵌套

组合选择器的使用和CSS的选择器基本相同

组合选择器

| 选择器         | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| $('div')       | 基本选择器，选择div标签                                      |
| $('div.c1')    | 选择class名为c1的div标签                                     |
| $('div#d1')    | 选择id名为d1的div标签                                        |
| $('*')         | 通用选择器，选择所有标签                                     |
| $('#d1,.c1,p') | 组合选择器，选择id名为d1的标签、class名为c1的标签、所有p标签 |

分组与嵌套

| 选择器        | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| $('div span') | 后代选择器，选择div的所有后代span                            |
| $('div>span') | 儿子选择器，选择div的儿子span (孙子辈就不算了)               |
| $('div+span') | 毗邻选择器，选择`紧接`在`指定元素`后的元素，且二者有相同父元素。 |
| $('div~span') | 弟弟选择器，选择`指定元素`后的元素，且二者有相同父元素。     |

分组与嵌套通过一个案例演示

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="jquery-3.6.0.min.js"></script>
</head>
<body>
    <span class="c1">c1-span</span>
    <div id="d1">d1
        <div id="d2">d2
            <div id="d3">d3<span>d3-span</span></div>
        </div>
    </div>
    <div id="d4">d4</div>
    <span class="c2">c2span</span>
    <span class="c3">c3span</span>
    <div id="d5">d5
        <div id="d6">d6</div>
    </div>
    <div id="d7">d7
        <span>d7-span
            <span>
                d7-span-span
            </span>
        </span>
    </div>
</body>
</html>
```

查找div的后代span

```javascript
// 找到3个
$('div span')
S.fn.init(3) [span, span, span, prevObject: S.fn.init(1)]

// 分别是
$('div span')[0]
<span>​d3-span​</span>​
$('div span')[1]
<span>​"d7-span "<span>​ d7-span-span ​</span>​</span>​
$('div span')[2]
<span>​ d7-span-span ​</span>​
```

查找div的儿子span

```javascript
// 查到2个
$('div>span')
S.fn.init(2) [span, span, prevObject: S.fn.init(1)]

// 分别是
$('div>span')[0]
<span>​d3-span​</span>​
$('div>span')[1]
<span>​"d7-span "<span>​ d7-span-span ​</span>​</span>​

// 为什么比后代少1个？
因为d7-span-span已经是孙子辈了，不在"儿子"里面
```

查找div的毗邻span

```javascript
// 只能找到1个
$('div+span')
S.fn.init [span.c2, prevObject: S.fn.init(1)]0: span.c2length: 1prevObject: S.fn.init [document][[Prototype]]: Object(0)

// d4紧接着的c2span
$('div+span')[0]
<span class=​"c2">​c2span​</span>​
```

查找div的弟弟span

```javascript
// 可以找到2个
$('div~span')
S.fn.init(2) [span.c2, span.c3, prevObject: S.fn.init(1)]

// 分别是
$('div~span')[0]
<span class=​"c2">​c2span​</span>​
$('div~span')[1]
<span class=​"c3">​c3span​</span>​

// 为什么会比毗邻多一个？
因为毗邻只算紧接的第一个，而弟弟可以算全部
```

