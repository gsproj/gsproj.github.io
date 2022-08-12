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

其实跟Java什么关系都没有, 只是为了<font color="red">**蹭Java的热度**</font>,才起的这么个名字!

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

## 2.1 定义变量的两种方法

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



## 2.2 变量的命名规范

```shell
js变量的命名规范
	1.变量名只能是 
		数字 字母 下划线 $
	2.变量名命名规范(不遵循也可以)
		1.js中推荐使用驼峰式命名
			userName
			dataOfDb
		2.python推荐使用下划线的方式
			user_name
			data_of_db
	3.不能用关键字作为变量名
			不需要记忆 
```



## 2.3 代码的书写位置

```shell
1.可以单独开设js文件书写
2.还可以直接在浏览器提供的console界面书写
	在用浏览器书写js的时候 左上方的清空按钮只是清空当前页面 代码其实还在
	如果你想要重新来 最好重新开设一个 页面
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

 

# 4 数据类型

js也是一门面向对象 的编程语言 即一切皆对象!!!

js/python是一门拥有动态类型

案例代码如下：

```javascript
name = 'jason'
name = 123
name = [1,2,3,4]
# name可以指向任意的数据类型 
```



## 4.1 数值类型（Number）

常用方法：
	parseInt()		转换为Int类型	
	parseFloat()	转换为Float类型

案例代码如下：

```javascript
// 定义变量
var a = 11;
var b = 11.11;

// 如何查看当前数据类型
typeof a;

// 查看参数的类型
typeof a;
typeof b;
"number"

// 特殊的 NaN:数值类型 表示的意思是“不是一个数字” NOT A NUMBER

parseInt('12312312')	// 正常将字符串转为Int类型
12312312
parseFloat('11.11')		// 正常将字符串转为浮点类型
11.11
parseInt('11.11')	// 正常将字符串转为Int类型，将去掉小数点后面的数
11
parseInt('123sdasdajs2312dasd')	// 只取前面的数
123
parseInt('asdasdad123sdasdajs2312dasd')	// 完全无法识别的转换为NaN
NaN
```



## 4.2 字符类型（String）

定义字符串的案例代码如下:

```javascript
// 使用【单引号】定义变量
var s = 'jason'

// 自动识别成了字符类型
typeof s
'string'

// 也可以使用【双引号】定义字符串
var s2 = "jason"
undefined

typeof s2
'string'

// 不能使用python的三引号方式定义多行文本的字符串
var s3 = '''jason'''
VM894:1 Uncaught SyntaxError: Unexpected string

// JavaScript中使用【反引号】定义多行文本的字符串
var s3 = `
这是多行文字
的演示
这是多行文字
的演示
`
// 输出s3
s3
'\n这是多行文字\n的演示\n这是多行文字\n的演示\n'
```

输出字符串的案例如下：

```javascript
// 通过${}的方式输出字符串
var age = 18
var name = "GLF"
var sss = `My name is ${name} and my age is ${age}`

// 输出
sss
'My name is GLF and my age is 18'

// JS中推荐使用【+】号拼接字符串
var year = '2022'
var day = '12'
var month = '08'
date = `Today is ${year + '-' + month + day}`
'Today is 2022-0812'
date
'Today is 2022-0812'
```

