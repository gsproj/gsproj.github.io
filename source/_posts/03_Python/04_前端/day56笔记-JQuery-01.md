---
title: Day56-JQuery-01
date: 2022-08-22 15:44:22
categories:
- Python
- 07_JQuery
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

## 2.3 基本筛选器

以ul li为例子，演示基本筛选器的使用

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
</head>
<body>
    <ul>爸爸
        <li>儿子01</li>
        <li id='d1'>儿子02</li>
        <li>儿子03</li>
        <li>儿子04</li>
        <li>儿子05</li>
        <li>儿子06</li>
        <li>儿子07</li>
        <li>儿子08</li>
        <li>儿子09</li>
        <li>儿子10</li>
    </ul>
</body>
</html>
```

使用基本筛选器

```javascript
// 1、获取ul下的所有li
$('ul li')
S.fn.init(10) [li, li, li, li, li, li, li, li, li, li, prevObject: S.fn.init(1)]

// 2、获取ul下的第一个li
$('ul li:first')
S.fn.init [li, prevObject: S.fn.init(1)]
// 或者
$('ul li').first()
S.fn.init [li, prevObject: S.fn.init(10)]

// 3、获取ul下的最后一个li
$('ul li:last')
S.fn.init [li, prevObject: S.fn.init(1)]
// 或者
$('ul li').last()
S.fn.init [li, prevObject: S.fn.init(10)]

// 4、获取ul的下标为2的后代
$('ul li:eq(2)')
S.fn.init [li, prevObject: S.fn.init(1)]
// 或者
$('ul li').eq(2)
S.fn.init [li, prevObject: S.fn.init(10)]

// 5、获取ul中下标为偶数的后代（0包含在内）
$('ul li:even')
S.fn.init(5) [li, li, li, li, li, prevObject: S.fn.init(1)]

// 6、获取ul中下标为奇数的后代
$('ul li:odd')
S.fn.init(5) [li, li, li, li, li, prevObject: S.fn.init(1)]

// 7、获取索引大于2的后代
$('ul li:gt(2)')
S.fn.init(7) [li, li, li, li, li, li, li, prevObject: S.fn.init(1)]

// 8、获取索引小于2的后代
$('ul li:lt(2)')
S.fn.init(2) [li, li#d1, prevObject: S.fn.init(1)]

// 9、移除满足条件的后代标签
$('ul li:not(#d1)')
S.fn.init(9) [li, li, li, li, li, li, li, li, li, prevObject: S.fn.init(1)]
```

## 2.4 属性选择器

可以根据标签属性选择到标签

案例代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
</head>
<body>
    <form action="">
        <p>用户名：<input type="text" username='Tom'></p>
        <p>真实姓名：<input type="text" username='Jack'></p>
        <p>住址：<input type="text" site='HuNan ChangSha'></p>
        <p>密码：<input type="password"></p>
        <p><input type="button" value='点我好康'></p>
    </form>
</body>
</html>
```

属性选择器的使用：

```javascript
// 1、找到所有包含username属性的标签
$('[username]')
S.fn.init(2) [input, input, prevObject: S.fn.init(1)]

// 2、找到所有username='Tom'的标签
$('[username="Tom"]')
S.fn.init [input, prevObject: S.fn.init(1)]

// 3、找到所有username='Jack'的标签
$('[username="Jack"]')
S.fn.init [input, prevObject: S.fn.init(1)]

// 4、找到所有包含type属性的标签
$('[type]')
S.fn.init(5) [input, input, input, input, input, prevObject: S.fn.init(1)]

// 5、找到所有包含type属性且type='text'的标签
$('[type="text"]')
S.fn.init(3) [input, input, input, prevObject: S.fn.init(1)]
```

## 2.5 表单筛选器

案例如下：

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
</head>
<body>
    <form action="">
        <p>用户名：<input type="text" username='Tom'></p>
        <p>真实姓名：<input type="text" username='Jack'></p>
        <p>住址：<input type="text" site='HuNan ChangSha'></p>
        <p>密码：<input type="password"></p>
        <p><input type="button" value='点我好康'></p>
        <p>性别
            <input type="radio" name="" id="" checked>男
            <input type="radio" name="" id="" >女
            <input type="radio" name="" id="" >未知
        </p>
        <p>爱好
            <input type="checkbox" name="" id="" checked>游泳
            <input type="checkbox" name="" id="">唱歌
            <input type="checkbox" name="" id="">划船
        </p>
        <p>学历
            <select name="" id="">
                <option value="" selected>本科</option>
                <option value="">研究生</option>
                <option value="">博士</option>
            </select>
        </p>
    </form>
</body>
</html>
```

### 2.5.1 通过input标签的类型选择元素

```javascript
// 1、获取拥有属性type='text'的input标签
$('input[type="text"]')
S.fn.init(3) [input, input, input, prevObject: S.fn.init(1)]
// 可以简写为
$(':text')
S.fn.init(3) [input, input, input, prevObject: S.fn.init(1)]

// 2、获取拥有属性type='password'的input标签
$('input[type="password"]')
S.fn.init [input, prevObject: S.fn.init(1)]
// 可以简写为
$(':password')
S.fn.init [input, prevObject: S.fn.init(1)]
```

可简写的一些方法：

- :text
- :password
- :file
- :radio
- :checkbox
- :submit
- :reset
- :button

### 2.5.2 通过表单对象属性选择元素

```javascript
// 1、获取所有包含checked=“checked”的标签
$(':checked')
S.fn.init(3) [input, input, option, prevObject: S.fn.init(1)]
// 明明只有两个checked，为什么拿了3个，因为有BUG，会把selected也拿到
$(':checked')[2]
<option value selected>​本科​</option>​   slot 
// 怎么解决这个问题？自己加限制条件
$('input:checked')
S.fn.init(2) [input, input, prevObject: S.fn.init(1)]

// 2、获取所有包含selected=“selected”的标签
$(':selected')
S.fn.init [option, prevObject: S.fn.init(1)]
```

相同用法的还有：

- :enabled
- :disabled

## 2.6 筛选器方法

根据一个标签，获取它的`父亲`、`儿子`、`爷爷`等

往后找

```javascript
// 获取#d1同级别下一个
$('#d1').next()  
w.fn.init [span, prevObject: w.fn.init(1)]0: spanlength: 1prevObject: 
w.fn.init [span#d1]__proto__: Object(0)
           
// 获取#d1同级别下的所有     
$('#d1').nextAll()
w.fn.init(5) [span, div#d2, span, span, span.c1, prevObject: w.fn.init(1)]0: span1: div#d22: span3: span4: span.c1length: 5prevObject: w.fn.init [span#d1]__proto__: Object(0)

// 获取#d1同级别下的所有，直到遇到.c1停止查找
$('#d1').nextUntil('.c1')  # 不包括最后一个
w.fn.init(4) [span, div#d2, span, span, prevObject: w.fn.init(1)]0: span1: div#d22: span3: spanlengt: 4prevObject: w.fn.init [span#d1]__proto__: Object(0)
```

往前找

```javascript
// 获取#d1同级别上一个
$('.c1').prev() 
w.fn.init [span, prevObject: w.fn.init(1)]0: spanlength: 1prevObject: w.fn.init [span.c1, prevObject: w.fn.init(1)]__proto__: Object(0)

// 获取#d1同级别上的所有
$('.c1').prevAll()
w.fn.init(5) [span, span, div#d2, span, span#d1, prevObject: w.fn.init(1)]
              
// 获取#d1同级别下的所有，直到遇到#d2停止查找
$('.c1').prevUntil('#d2')
w.fn.init(2) [span, span, prevObject: w.fn.init(1)]
```

找爸爸

```javascript
$('#d3').parent()  # 第一级父标签
w.fn.init [p, prevObject: w.fn.init(1)]0: plength: 1prevObject: w.fn.init [span#d3]__proto__: Object(0)
$('#d3').parent().parent()
w.fn.init [div#d2, prevObject: w.fn.init(1)]
$('#d3').parent().parent().parent()
w.fn.init [body, prevObject: w.fn.init(1)]
$('#d3').parent().parent().parent().parent()
w.fn.init [html, prevObject: w.fn.init(1)]
$('#d3').parent().parent().parent().parent().parent()
w.fn.init [document, prevObject: w.fn.init(1)]
$('#d3').parent().parent().parent().parent().parent().parent()
w.fn.init [prevObject: w.fn.init(1)]
$('#d3').parents()
w.fn.init(4) [p, div#d2, body, html, prevObject: w.fn.init(1)]
$('#d3').parentsUntil('body')
w.fn.init(2) [p, div#d2, prevObject: w.fn.init(1)]
```

找儿子

```javascript
$('#d2').children()
```

找兄弟

```javascript
$('#d2').siblings()  # 同级别上下所有
```

## 2.7 补充说明

存在一些互相等价的用法

```javascript
// 案例1
$('div p')
// 等价于           
$('div').find('p')  # find从某个区域内筛选出想要的标签 
              
// 案例2
$('div span:first')
w.fn.init [span, prevObject: w.fn.init(1)]
// 等价于
$('div span').first()
w.fn.init [span, prevObject: w.fn.init(3)]0: spanlength: 1prevObject: w.fn.init(3) [span, span#d3, span, prevObject: w.fn.init(1)]__proto__: Object(0)

// 案例3
$('div span:last')
w.fn.init [span, prevObject: w.fn.init(1)]
$('div span').last()

// 案例4
w.fn.init [span, prevObject: w.fn.init(3)]
$('div span:not("#d3")')
w.fn.init(2) [span, span, prevObject: w.fn.init(1)]
// 等价于
$('div span').not('#d3')
w.fn.init(2) [span, span, prevObject: w.fn.init(3)]
```

