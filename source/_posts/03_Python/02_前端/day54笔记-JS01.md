---
title: Day54-JS-01
date: 2022-08-08 16:44:22
categories:
- Python
- Python入门
tags:
---

“第54天JS-01学习笔记”

# 1 JavaScript简介

**<font color=blue>JavaScript(简称JS)</font>** 听起来是不是感觉跟Java的关系很大?

其实跟Java什么关系都没有, 只是为了<font color="red">**蹭jiava的热度**</font>,才起的这么个名字!

JS到底是什么?

- JavaScript 是脚本语言
- JavaScript 是一种轻量级的编程语言。
- JavaScript 是可插入 HTML 页面的编程代码。
- JavaScript 插入 HTML 页面后，可由所有的现代浏览器执行。
- JavaScript 很容易学习

第一个js程序

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script type="text/javascript">
        // 单行注释
        
        /*
        多行注释
        */
        
        document.write("<h1>Hello World!</h1>")
    </script>
</head>
<body>
</body>
</html>
```

# 2 JS变量

定义js变量两种方法

- var 
- let  (es6推出的新语法, 如果你的编辑器支持的版本是5.1那么无法使用let)

案例代码如下:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script type="text/javascript">
        // var定义变量
        var num1 = 100
        alert(num1)

        // let定义变量
        let num2 = 300
        alert(num2)
    </script>
</head>
<body>
</body>
</html>
```

let和var的区别是什么?  

- var是函数作用域，let是块作用域\

  ```javascript
  for (var num = 0; num < 100; num++){}
  console.log(num) // 输出100	(说明函数内有效)
  
  for (let num2 = 0; num < 100; num++){}
  console.log(num2) // 输出错误, num2未定义	(说明离开for循环块,就无效了)
  ```

- let不能在定义之前访问该变量, 但是var可以\

  var正常

  ```javascript
  console.log(num1)
  var num1 = 300  // 无报错，不输出东西
  ```

  let报错

  ```javascript
  console.log(num1)
  let num1 = 300  // 无报错，不输出东西
  // 报错:02-变量.html:14 Uncaught ReferenceError: Cannot access 'num1' before initialization
      at 02-变量.html:14:21
  ```

- let不能被重新定义, 但是var可以

  var正常

  ```javascript
  var num1 = 100
  console.log(num1)    // 正常输出100
  var num1 = 200
  console.log(num1)    // 正常输出200
  ```

  let报错

  ```javascript
  let num1 = 100
  console.log(100)
  let num1 = 200
  console.log(200)
  // 报错: Uncaught SyntaxError: Identifier 'num1' has already been declared
  ```



# 3 JS常量

js中使用`const`定义常量, 不可被修改的变量

>补充: Python中没有真正意义上的常量,默认全大小代表常量,而JS中有真正意义上的常量

案例代码

```javascript
const pi = 3.14
console.log(pi)
pi = 80 // 报错：Uncaught TypeError: Assignment to constant variable.
```







