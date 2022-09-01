---
title: Day57-JQuery-02
date: 2022-08-25 14:44:22
categories:
- Python
- 前端基础
tags:
---

“第57天JQuery（02）学习笔记”

今天主要内容

* jQuery操作标签
* jQuery绑定事件
* jQuery补充知识点
* jQuery动画效果(了解)
* Bootstrap(前端框架)开头



# 1 JQuery练习题

[练习题页面的源码](https://github.com/gsproj/gsproj.github.io/blob/source/source/_posts/03_Python/02_%E5%89%8D%E7%AB%AF/sources/day57/jQuery%E7%BB%83%E4%B9%A0%E9%A2%98.html)

找到本页面中id是`i1`的标签

```javascript
$('#i1')
S.fn.init [div#i1.container]
$('#i1')[0]
<div id=​"i1" class=​"container">​…​</div>​
```

找到本页面中所有的`h2`标签

```javascript
$('h2')
S.fn.init [h2, prevObject: S.fn.init(1)]
$('h2')[0]
<h2>​练习题：​</h2>​
```

找到本页面中所有的`input`标签

```javascript
$('input')
S.fn.init(9) [input#exampleInputEmail1.form-control, input#exampleInputPassword1.form-control, input#exampleInputFile, input, input, input, input, input#optionsRadios1, input#optionsRadios2, prevObject: S.fn.init(1)]
$('input')[0]
<input type=​"email" class=​"form-control" id=​"exampleInputEmail1" placeholder=​"Email">​
$('input')[1]
<input type=​"password" class=​"form-control" id=​"exampleInputPassword1" placeholder=​"Password">​
```

找到本页面所有样式类中有`c1`的标签

```javascript
$('.c1')
S.fn.init(2) [h1.c1, h1.c1, prevObject: S.fn.init(1)]
$('.c1')[0]
<h1 class=​"c1">​Jason​</h1>​
```

找到本页面所有样式类中有`btn-default`的标签

```javascript
$('.btn-default')
S.fn.init [button#btnSubmit.btn.btn-default, prevObject: S.fn.init(1)]
$('.btn-default')[0]
<button id=​"btnSubmit" type=​"submit" class=​"btn btn-default">​提交​</button>​
```

找到本页面所有样式类中有`c1`的标签和所有`h2`标签

```javascript
$('.c1, h2')
S.fn.init(3) [h1.c1, h1.c1, h2, prevObject: S.fn.init(1)]
```

找到本页面所有样式类中有`c1`的标签和id是`p3`的标签

```javascript
$('.c1, #p3')
S.fn.init(3) [h1.c1, h1.c1, p#p3.divider, prevObject: S.fn.init(1)]
$('.c1, #p3')[2]
<p id=​"p3" class=​"divider">​</p>​
```

找到本页面所有样式类中有`c1`的标签和所有样式类中有`btn`的标签

```javascript
$('.c1, .btn')
S.fn.init(11) [h1.c1, h1.c1, a.btn.btn-primary.btn-lg, button.btn.btn-warning, button.btn.btn-danger, button.btn.btn-warning, button.btn.btn-danger, button.btn.btn-warning, button.btn.btn-danger, button#btnSubmit.btn.btn-default, a.btn.btn-success, prevObject: S.fn.init(1)]
```

找到本页面中`form`标签中的所有`input`标签

```javascript
$('form').find('input')
S.fn.init(3) [input#exampleInputEmail1.form-control, input#exampleInputPassword1.form-control, input#exampleInputFile, prevObject: S.fn.init(1)]
```

找到本页面中被包裹在`label`标签内的`input`标签

```javascript
$('label input')
S.fn.init(6) [input, input, input, input, input#optionsRadios1, input#optionsRadios2, prevObject: S.fn.init(1)]
```

找到本页面中紧挨在`label`标签后面的`input`标签

```javascript
$('label+input')
S.fn.init(3) [input#exampleInputEmail1.form-control, input#exampleInputPassword1.form-control, input#exampleInputFile, prevObject: S.fn.init(1)]
```

找到本页面中id为`p2`的标签后面所有和它同级的`li`标签

```javascript
$('#p2~li')
S.fn.init(8) [li, li, li, li, li, li, li, li, prevObject: S.fn.init(1)]
```

找到id值为`f1`的标签内部的第一个input标签

```javascript
$('#f1 input:first')
S.fn.init [input#exampleInputEmail1.form-control, prevObject: S.fn.init(1)]
$('#f1 input:first')[0]
<input type=​"email" class=​"form-control" id=​"exampleInputEmail1" placeholder=​"Email">​
```

找到id值为`my-checkbox`的标签内部最后一个input标签

```javascript
$('#my-checkbox input:last')
S.fn.init [input, prevObject: S.fn.init(1)]
```

找到id值为`my-checkbox`的标签内部没有被选中的那个input标签

```javascript
$('#my-checkbox input[checked!="checked"]')
S.fn.init(3) [input, input, input, prevObject: S.fn.init(1)]
```

找到所有含有`input`标签的`label`标签

```javascript
$('label:has("input")')
S.fn.init(6) [label, label, label, label, label, label, prevObject: S.fn.init(1)]
```

# 2 JQuery操作标签

## 2.1 操作Class

JQuery操作类主要有四个方法：

| JQuery方法    | JS方法               | 作用               |
| ------------- | -------------------- | ------------------ |
| addClass()    | classList.add()      | 添加类             |
| removeClass() | classList.remove()   | 移除类             |
| hasClass()    | classList.contains() | 查询是否包含某个类 |
| toggleClass() | classList.toggle()   | 有则移除，无则添加 |

操作案例如下：

页面代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .c1 {
            background-color: coral;
            height: 200px;
            width: 300px;
        }

        .c2 {
            border: 20px solid red;
        }

        .c3 {
            margin-top: 20%;
            margin-left: 50%;
        }
    </style>
    <script src="jquery-3.6.0.min.js"></script>
</head>
<body>
<div class="c1 c2 c3"></div>
</body>
</html>
```

操作类：

```javascript
// 移除类
$('div').removeClass('c2')
S.fn.init [div.c1.c3, prevObject: S.fn.init(1)]
// 添加类
$('div').addClass('c2')
S.fn.init [div.c1.c3.c2, prevObject: S.fn.init(1)]
// 查询是否包含类
$('div').hasClass('c1')
true
// 有则移除，无则添加
$('div').toggleClass('c1')
S.fn.init [div.c2.c3, prevObject: S.fn.init(1)]
$('div').toggleClass('c1')
S.fn.init [div.c2.c3.c1, prevObject: S.fn.init(1)]
$('div').toggleClass('c1')
S.fn.init [div.c2.c3, prevObject: S.fn.init(1)]
```

## 2.2 操作CSS

JQuery操作CSS的方式：

```javascript
// 选择的标签.css('属性','值')
// 即：
$('div').css('color','red')
```

案例如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcss.com/jquery/3.6.0/jquery.min.js"></script>
</head>
<body>
<p>111</p>
<p>222</p>
<p>333</p>
</body>
</html>
```

操作CSS：

```javascript
// 使第一个p标签，变为红色
$('p').first().css('color','red')
w.fn.init [p, prevObject: w.fn.init(3)]
```

也可以连起来操作

```javascript
// 让第2个p标签变为绿色，让最后一个p标签变为黄色
$('p').last().css('color','green').prev().css('color','yellow')
w.fn.init [p, prevObject: w.fn.init(1)]
```

>补充：
>
>为什么JQuery可以连起来操作？
>
>因为操作完之后返回的还是JQuery对象
>
>实际上是一种链式操作，用Python的方式还原，如下代码：
>
>```python
>class MyClass(object):
>    def func1(self):
>        print('func1')
>        return self
>
>    def func2(self):
>        print('func2')
>        return self
>
>
>obj = MyClass()
>obj.func1().func2()
>```

## 2.3 操作位置

操作位置需要了解

| 方法         | 作用                          |
| ------------ | ----------------------------- |
| offset()     | 相较于浏览器窗口，移动位置    |
| position()   | 相对于父标签，移动位置        |
| scrollTop()  | 获取/设置滚动条的位置（左右） |
| scrollLeft() | 获取/设置滚动条的位置（上下） |

### 2.3.1 offset()和position()案例

页面代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
    <style>
        body {
            margin: 0;
        }
        p {
            position: relative;
            top: 100px;
            left: 100px;
        }
    </style>
</head>
<body>
<p>ppp</p>
</body>
</html>
```

操作代码：

```javascript
// offset移动p标签
$('p').offset({top:10,left:10})
w.fn.init [p, prevObject: w.fn.init(1)]

// position获取标签的位置
$('p').position()
{top: -6, left: 10}
```

### 2.3.2 scrollTop()案例

页面代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
    <style>
        .c1 {
            width: 100%;
            height: 500px;
            background-color: red;
        }

        .c2 {
            width: 100%;
            height: 500px;
            background-color: blue;
        }

        .c3 {
            width: 100%;
            height: 500px;
            background-color: pink;
        }
    </style>
</head>
<body>
    <div class="c1"></div>
    <div class="c2"></div>
    <div class="c3"></div>
</body>
</html>
```

获取当前窗口下拉框的位置

```javascript
$(window).scrollTop()
400
```

设置当前窗口下拉框的位置

```javascript
$(window).scrollTop(20)
w.fn.init [Window]
```

## 2.4 操作尺寸

操作标签尺寸的方法有如下：

| 方法          | 作用               |
| ------------- | ------------------ |
| height()      | 设置标签内容的宽度 |
| width()       | 设置标签内容的高度 |
| innerHeight() | 设置标签内高       |
| innerWidth()  | 设置标签内宽       |
| outerHeight() | 设置标签外高       |
| outerWidth()  | 设置标签外宽       |

关于这些方法控制的位置，图示图下：

![image-20220825165519019](../../../img/image-20220825165519019.png)

案例代码如下：

```javascript
// 不加参数，获取标签宽度/高度
$('p').last().width()
942
$('p').last().innerHeight()
21

// 指定参数，设置标签宽度/高度
$('p').last().innerHeight(300)
w.fn.init [p, prevObject: w.fn.init(3)]
```

## 2.5 文本操作

主要方法如下

| 方法   | 作用                 |
| ------ | -------------------- |
| text() | 获取标签内的文本     |
| html() | 获取标签内的HTML代码 |

案例代码：

HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<div id="d1">童稚
    <p class="c1">余忆童稚时</p>
    <p class="c1">能张目对日</p>
    <p class="c1">明察秋毫</p>
</div>
<p id="d2">222</p>
<p id="d3">333</p>
</body>
</html>
```

JS:

```javascript
// 获取标签内部文本
$('#d1').text()
'童稚\n    余忆童稚时\n    能张目对日\n    明察秋毫\n'
// 获取标签内部HTML代码
$('#d1').html()
'童稚\n    <p class="c1">余忆童稚时</p>\n    <p class="c1">能张目对日</p>\n    <p class="c1">明察秋毫</p>\n'
```

## 2.6 操作Value

JQuery中使用`val()`方法获取`input`标签的`value`属性值

案例HTML代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
    <div><input id="d1" type="text" value="测试"></div>
    <div><input id="d2" type="file"></div>
</body>
</html>
```

普通输入框-值操作

```javascript
// 获取标签d1的value值
$('#d1').val()
'测试'

// 设置标签d1的value值
$('#d1').val('我换值了')
w.fn.init [input#d1]
```

文件选择框的值操作

```javascript
// 获取值
$('#d2').val()
'C:\\fakepath\\TFile.txt'

// 转换为文件对象
$('#d2')[0].files[0]
File {name: 'TFile.txt', lastModified: 1661233436430, lastModifiedDate: Tue Aug 23 2022 13:43:56 GMT+0800 (中国标准时间), webkitRelativePath: '', size: 797, …}
```

## 2.7 操作属性

有如下方法：

| 方法              | 作用                |
| ----------------- | ------------------- |
| attr(name, value) | 设置属性name的value |
| attr(name)        | 获取属性name的值    |
| removeAttr(name)  | 移除name属性        |

仍用2.6的HTML代码，操作案例如下：

```javascript
// 获取d1标签的id值
$('#d1').attr('id')
'd1'
// 将id值改为d3
$('#d1').attr('id', 'd3')
w.fn.init [input#d3]
// 再次获取标签d1的id值，获取失败，说明修改成功
$('#d1').attr('id')
undefined
// 获取标签d3的值，获取成功
$('#d3').attr('id')
'd3'
// 移除d3属性
$('#d3').removeAttr('d3')
w.fn.init [input#d3]
// 获取不到，说明移除成功           
$('#d3').attr('d3')
undefined
```

**<font color=red>特殊情况说明</font>**：

- 对于标签上有的能够看到的属性和自定义属性用attr
- 对于返回布尔值比如checkbox radio option是否被选中用prop

案例代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
    <div><input id="d1" type="text" value="测试"></div>
    <div><input id="d2" type="file"></div>
    <div>
        <input type="checkbox">sel01
        <input id="d3" type="checkbox" checked="checked">sel02
        <input id="d4" type="checkbox" checked="checked">sel03
    </div>
    <div>
        <input id="d5" type="radio" >Rad01
        <input id="d6" type="radio" checked="checked">Rad02
        <input type="radio" >Rad03
    </div>
    <select name="" id="">
        <option id="d7" value="">选择一</option>
        <option id="d8" value="" selected="selected">选择二</option>
    </select>
</body>
</html>
```

什么问题？

```javascript
// 用attr获取checkbox得值，能获取
$("#d3").attr('checked')
'checked'
// 设置值
$("#d3").attr('checked', 'unchecked')
w.fn.init [input#d3]
// 再次获取，设置是失败的，这就是问题
$("#d3").attr('checked')
'checked'
```

使用`prop()`尝试

```javascript
// 1、设置checkbox
// 获取值
$("#d3").prop('checked')
true
// 设置值
$("#d3").prop('checked', false)
w.fn.init [input#d3]
$("#d3").prop('checked')
false

// 2、设置radio
$("#d5").prop('checked')
false
$("#d5").prop('checked', true)
w.fn.init [input#d5]
$("#d5").prop('checked')
true

// 3、设置select的option
$("#d8").prop('selected')
true
$("#d8").prop('selected', false)
w.fn.init [option#d8]
$("#d8").prop('selected')
false
```

## 2.8 文档处理

文档处理常用操作

| 方法                  | 作用                                   |
| --------------------- | -------------------------------------- |
| $('<p>')              | 创建标签，相当于JS的createElement('p') |
| append($pEle)         | 内部尾部追加元素，相当于appendChild()  |
| prepend($pEle)        | 内部头部追加元素                       |
| after($pEle)          | 放在某个标签后面                       |
| insertAfter($('#d1')) |                                        |
| before($pEle)         | 放在某个标签前面                       |
| $('#d1').remove()     | 将标签从DOM树中删除                    |
| $('#d1').empty()      | 清空标签内部所有的内容                 |

案例代码如下：

```javascript
let $pEle = $('<p>')
$pEle.text('你好啊 草莓要不要来几个?')
$pEle.attr('id','d1')          
$('#d1').append($pEle)  # 内部尾部追加
$pEle.appendTo($('#d1')) 
           
$('#d1').prepend($pEle)  # 内部头部追加
w.fn.init [div#d1]
$pEle.prependTo($('#d1'))
w.fn.init [p#d1, prevObject: w.fn.init(1)]
         
$('#d2').after($pEle)  # 放在某个标签后面
w.fn.init [p#d2]
$pEle.insertAfter($('#d1'))
        
$('#d1').before($pEle)
w.fn.init [div#d1]
$pEle.insertBefore($('#d2'))

$('#d1').remove()  # 将标签从DOM树中删除
w.fn.init [div#d1]
           
$('#d1').empty()  # 清空标签内部所有的内容
w.fn.init [div#d1]
```



# 3 JQuery事件

## 3.1 事件触发

触发事件有两种方式

第一种

```javascript
$('#d1').click(function () {
    alert('别说话 吻我')
});
```

第二种(功能更加强大 还支持事件委托)

```python
$('#d2').on('click',function () {
    alert('借我钱买草莓 后面还你')
})
```

## 3.2 克隆事件

```html
<button id="d1">屠龙宝刀，点击就送</button>

<script>
    $('#d1').on('click',function () {
        // console.log(this)  // this指代是当前被操作的标签对象
        // $(this).clone().insertAfter($('body'))  // clone默认情况下只克隆html和css 不克隆事件
        $(this).clone(true).insertAfter($('body'))  // 括号内加true即可克隆事件

    })
</script>
```

## 3.3 自定义模态框

```python
"""
模态框内部本质就是给标签移除或者添加上hide属性
"""
```

* 左侧菜单

  ```html
  <script>
      $('.title').click(function () {
          // 先给所有的items加hide
          $('.items').addClass('hide')
          // 然后将被点击标签内部的hide移除
          $(this).children().removeClass('hide')
      })
  </script>
  <!--尝试用一行代码搞定上面的操作-->
  ```

* 返回顶部

  ```python
  <script>
      $(window).scroll(function () {
          if($(window).scrollTop() > 300){
              $('#d1').removeClass('hide')
          }else{
              $('#d1').addClass('hide')
          }
      })
  </script>
  ```

* 自定义登陆校验

  ```python
  # 在获取用户的用户名和密码的时候 用户如果没有填写 应该给用户展示提示信息
  <p>username:
      <input type="text" id="username">
      <span style="color: red"></span>
  </p>
  <p>password:
      <input type="text" id="password">
      <span style="color: red"></span>
  </p>
  <button id="d1">提交</button>
  
  <script>
      let $userName = $('#username')
      let $passWord = $('#password')
      $('#d1').click(function () {
          // 获取用户输入的用户名和密码 做校验
          let userName = $userName.val()
          let passWord = $passWord.val()
          if (!userName){
              $userName.next().text("用户名不能为空")
          }
          if (!passWord){
              $passWord.next().text('密码不能为空')
          }
      })
      $('input').focus(function () {
          $(this).next().text('')
      })
  </script>
  ```

* input实时监控

  ```python
  <input type="text" id="d1">
  
  <script>
      $('#d1').on('input',function () {
          console.log(this.value)  
      })
  </script>
  ```

* hover事件

  ```python
  <script>
      // $("#d1").hover(function () {  // 鼠标悬浮 + 鼠标移开
      //     alert(123)
      // })
  
      $('#d1').hover(
          function () {
              alert('我来了')  // 悬浮
      },
          function () {
              alert('我溜了')  // 移开
          }
      )
  </script>
  ```

* 键盘按键按下事件

  ```html
  <script>
      $(window).keydown(function (event) {
          console.log(event.keyCode)
          if (event.keyCode === 16){
              alert('你按了shift键')
          }
      })
  </script>
  ```

## 3.4 阻止后续事件执行

以案例说明，点击按钮后，本应将span(d1)的文本设置成'宝贝 你能看到我吗?'，但由于后续submit的提交事件执行，页面将刷新，导致设置的文本消失：

提交前：

![image-20220831151405792](../../../img/image-20220831151405792.png)

提交后：

![image-20220831151419650](../../../img/image-20220831151419650.png)

半秒后：

![image-20220831151405792](../../../img/image-20220831151405792.png)

如何解决这个问题，停止后续的submit操作？

- return false
- e.preventDefault()

案例代码如下：

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<form action="">
    <span id="d1" style="color: red"></span>
    <input type="submit" id="d2">
</form>

<script>
    $('#d2').click(function (e) {
        $('#d1').text('宝贝 你能看到我吗?')
        // 阻止标签后续事件的执行 方式1
        // return false
        // 阻止标签后续事件的执行 方式2
        // e.preventDefault()
    })
</script>

</body>
</html>
```

## 3.5 阻止事件冒泡

以案例说明，由于d1、d2、 d3是嵌套关系，当点击d3触发click事件后，页面显示alert('span')，然后alert('p')，最后alert('div')，将alert三次。

怎么解决这个问题，实现只触发#d3的alert？

- return false
- e.stopPropagation()

案例代码如下：

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
    <div id="d1">div
        <p id="d2">div>p
            <span id="d3">span</span>
        </p>
    </div>


    <script>
        $('#d1').click(function () {
            alert('div')
        })
        $('#d2').click(function () {
            alert('p')
        })
        $('#d3').click(function (e) {
            alert('span')
            // 阻止事件冒泡的方式1
            // return false
            // 阻止事件冒泡的方式2
            // e.stopPropagation()
        })
    </script>
</body>
</html>
```

## 3.6 事件委托

正常情况下，我们帮一个按钮添加点击事件，只需给按钮绑定事件即可，即：

```javascript
$('button').click(function () {  // 无法影响到动态创建的标签
     alert(123)
})
```

但是，假如我们动态创建了button按钮，这种绑定方式对`后续动态生成的button`是`无效的`

需要使用事件委托:

```javascript
// 事件委托
$('body').on('click','button',function () {
    alert(123)  // 在指定的范围内 将事件委托给某个标签 无论该标签是事先写好的还是后面动态创建的
})
```

# 4 补充

## 4.1 页面加载

等页面元素加载完毕后，再调用JS代码。

在原生JS中，我们使用：

```javascript
// 方法一
window.onload = function(){
  // js代码
}

// 方法二
直接写在body最下方
```

在JQuery中有三种方式

```javascript
// 第一种
$(document).ready(function(){
  // js代码
})
// 第二种
$(function(){
  // js代码
})
// 第三种
直接写在body内部最下方
```

## 4.2 动画效果

给元素添加`淡入淡出`、`隐藏/出现`等效果

```javascript
$('#d1').hide(5000)
w.fn.init [div#d1]
$('#d1').show(5000)
w.fn.init [div#d1]
$('#d1').slideUp(5000)
w.fn.init [div#d1]
$('#d1').slideDown(5000)
w.fn.init [div#d1]
$('#d1').fadeOut(5000)
w.fn.init [div#d1]
$('#d1').fadeIn(5000)
w.fn.init [div#d1]
$('#d1').fadeTo(5000,0.4)
w.fn.init [div#d1]   
```

案例代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
    <style>
        #d1 {
            height: 1000px;
            width: 400px;
            background-color: red;
        }
    </style>
</head>
<body>
    <div id="d1"></div>
    <div><button id="d2">别按我！</button><button id="d3">再按就生气了~！</button></div>

    <script>
        $("#d2").click(
            function() {
                $("#d1").hide(5000)	// 5秒动画-消失
            }
        )

        $("#d3").click(
            function() {
                $("#d1").show(5000) // 5秒动画-出现
                
            }
        )
    </script>
</body>
</html>
```

## 4.3 遍历元素

案例代码如下：

```javascript
# each()
# 第一种方式
$('div')
w.fn.init(10) [div, div, div, div, div, div, div, div, div, div, prevObject: w.fn.init(1)]
$('div').each(function(index){console.log(index)})
VM181:1 0
VM181:1 1
VM181:1 2
VM181:1 3
VM181:1 4
VM181:1 5
VM181:1 6
VM181:1 7
VM181:1 8
VM181:1 9

$('div').each(function(index,obj){console.log(index,obj)})
VM243:1 0 <div>​1​</div>​
VM243:1 1 <div>​2​</div>​
VM243:1 2 <div>​3​</div>​
VM243:1 3 <div>​4​</div>​
VM243:1 4 <div>​5​</div>​
VM243:1 5 <div>​6​</div>​
VM243:1 6 <div>​7​</div>​
VM243:1 7 <div>​8​</div>​
VM243:1 8 <div>​9​</div>​
VM243:1 9 <div>​10​</div>​

# 第二种方式
$.each([111,222,333],function(index,obj){console.log(index,obj)})
VM484:1 0 111
VM484:1 1 222
VM484:1 2 333
(3) [111, 222, 333]
"""
有了each之后 就无需自己写for循环了 用它更加的方便
"""
# data()
"""
能够让标签帮我们存储数据 并且用户肉眼看不见
"""
$('div').data('info','回来吧，我原谅你了!')
w.fn.init(10) [div#d1, div, div, div, div, div, div, div, div, div, prevObject: w.fn.init(1)]
               
$('div').first().data('info')
"回来吧，我原谅你了!"
$('div').last().data('info')
"回来吧，我原谅你了!"
               
$('div').first().data('xxx')
undefined
$('div').first().removeData('info')
w.fn.init [div#d1, prevObject: w.fn.init(10)]
           
$('div').first().data('info')
undefined
$('div').last().data('info')
"回来吧，我原谅你了!"
```



