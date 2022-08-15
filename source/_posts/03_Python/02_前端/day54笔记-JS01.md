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

// 自动识别成了字符类型
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

字符类型常用方法

| 方法名                    | 作用               |
| ------------------------- | ------------------ |
| length                    | 返回字符串长度     |
| trim()                    | 移除空白           |
| trimLeft()                | 移除左边的空白     |
| trimRight()               | 移除右边的空白     |
| charAt(n)                 | 返回第n个字符      |
| concat(value, ....)       | 拼接字符串         |
| indexof(substring, start) | 子序列位置         |
| substring(from, to)       | 根据索引获取子序列 |
| slice(start, end)         | 切片               |
| toLowerCase()             | 小写               |
| toUpperCase()             | 大写               |
| split(delimiter, limit)   | 分割               |

`length`方法案例：

```javascript
var str = 'you are beautiful'
undefined
console.log(str.length);
17
```

trim()方法案例：

```javascript
let str2 = " my left and right have blank "

// 输出 "my left and right have blank"
console.log(str2.trim())


// 输出 " my left and right have blank"
console.log(str2.trimRight())


// 输出 "my left and right have blank "
console.log(str2.trimLeft())


// 不能像Python一样指定去除的内容
var str = "$$Test string$$"
str.trim('$')
'$$Test string$$' // 去除$失败了
```

`charAt()`案例：

```javascript
var str = "$$Test string$$"

// 获取位置0和8的字符
str.charAt(0)
'$'
str.charAt(8)
't'
```

`indexOf()`案例

```javascript
var str = "$$Test string$$"

// 获取'st'和'str'第一次出现的位置
str.indexOf('st')
4
str.indexOf('str')
7
```

`substring()`案例：

```javascript
var str = "$$Test string$$"

// 获取位置1-5的字串
str.substr(1,5)
'$Test'

// 也可以用负数，表示倒数
var str = "12345678"
str.substr(-3, 20)	// 20超出范围，选到最后的位置
'678'
```

`slice()`案例（功能跟`substring()`类似，推荐使用这个）：

```javascript
var str = "12345678"
// 切片(1，5)
str.substr(1, 5)
'23456'
```

`toLowerCase()`和`toUpperCase()`案例：

```javascript
var str = 'ToDay is MONDAY'
str.toLowerCase()
'today is monday'
str.toUpperCase()
'TODAY IS MONDAY'
```

`split()`案例：

```javascript
var name = "Jerry | Tom | Jemmy | Marry"
undefined

// 指定分割符分割
name.split('|')
(4) ['Jerry ', ' Tom ', ' Jemmy ', ' Marry']

// 第二个参数可以指定获取的元素个数
name.split('|', 2)
(2) ['Jerry ', ' Tom ']
name.split('|', 20)
(4) ['Jerry ', ' Tom ', ' Jemmy ', ' Marry']
```

`concat()`案例:

```javascript
var str1 = "Hello everyone!"
undefined
var str2 = "My name is Tom!"
undefined

// 拼接str1和str2
str1.concat(str2)
'Hello everyone!My name is Tom!'

// 非字符串类型会自动转换后拼接
var phoneNumber = 12345678
undefined
str1.concat(phoneNumber)
'Hello everyone!12345678'
```



## 4.3 布尔值(boolean)

JS布尔值的定义：

- 在python中布尔值是首字母大写的
  - True / False
- 但是在js中布尔值是全小写的
  - true / false
- 布尔值是false的有哪些
  - 空字符串
  - 0
  - null
  - undefined
  - NaN

案例：

```javascript
var isFalse = false
undefined
var isTrue = true
undefined
```



## 4.4 null与undefined的区别

null

​	表示值为空 一般都是指定或者清空一个变量时使用

如:

```javascript
name = 'jason'
name = null
```

undefined

​	表示声明了一个变量 但是没有做初始化操作(没有给值)

​	函数没有指定返回值的时候 返回的也是undefined



## 4.5 对象(数组)

JS数组类似于Python中的列表`[]`，在JS中`一切皆为对象`

### 4.5.1 数组的定义

见案例：

```javascript
// 定义数组
var list = [11, "abc", 33.44, false]
undefined

// 获取数组元素
list[3]
false

// 类型是`对象`
typeof(list)
'object'
```

### 4.5.2 数组的常用方法

`length`获取数组长度

```javascript
var list = [11, "abc", 33.44, false]
list.length
4
```

`push()`追加元素到尾部

```javascript
// 追加元素，返回追加后的数组长度
list.push("Hello")
5
// 查看数组元素
list
(5) [11, 'abc', 33.44, false, 'Hello']
```

`pop()`弹出尾部元素

```javascript
list
(5) [11, 'abc', 33.44, false, 'Hello']
// 弹出元素（从最后一个开始）
list.pop()
'Hello'
list.pop()
false
list.pop()
33.44
list
(2) [11, 'abc']
```

`unshift()`数组头部插入元素

```javascript
// 查看数组元素
list
(3) [11, 11, 'abc']

// 头部插入元素
list.unshift("Good")
4

// 再次查看
list
(4) ['Good', 11, 11, 'abc']
```

`shift()`数组弹出头部元素

```javascript
// 查看数组元素
list
(4) ['Good', 11, 11, 'abc']

// 删除头部元素
list.shift()
'Good'

// 查看头部元素
list
(3) [11, 11, 'abc']
```

`slice()`数组切割

```javascript
// 查看数组元素
list
(4) [11, 11, 'abc', 555]

// 切割
list.slice(1,3)
(2) [11, 'abc']
```

`reverse()`数组反转

```javascript
// 查看数组元素
list
(4) [11, 11, 'abc', 555]

// 数组反转
list.reverse()
(4) [555, 'abc', 11, 11]
```

`join()`将`数组元素`转换为`字符串`

```javascript
// 查看数组元素
list
(4) [555, 'abc', 11, 11]

// 转换为字符串（参数为`拼接符号`）
list.join('$')
'555$abc$11$11'
```

`concat()`拼接数组

```javascript
// 查看数组元素
list
(4) [555, 'abc', 11, 11]

// 拼接另一个数组
list.concat(['Duo', 888, "hello"])
(7) [555, 'abc', 11, 11, 'Duo', 888, 'hello']

// 可以拼接多个数组
list.concat(['Duo', 888, "hello"], [666, 888])
(9) [555, 'abc', 11, 11, 'Duo', 888, 'hello', 666, 888]
```

`sort()`数组排序

```javascript
var list2 = [666, 111, 222, 888, 999]
undefined
// 数组排序
list2.sort()
(5) [111, 222, 666, 888, 999]
```



### 4.5.3 三个比较重要的方法

1） `foreach()`遍历数组元素

```javascript
// 定义数组
var list1 = ["Changsha", "Beijing", "Shanghai", "Taiwan"]
undefined

// 遍历数组（一个参数：显示值）
var list1 = ["Changsha", "Beijing", "Shanghai", "Taiwan"]
undefined
list1.forEach(function(value){console.log(value)})
VM942:1 Changsha
VM942:1 Beijing
VM942:1 Shanghai
VM942:1 Taiwan


// 遍历数组（两个参数：显示值和下标）
list1.forEach(function(value, index){console.log(value, index)}, list1)
VM541:1 Changsha 0
VM541:1 Beijing 1
VM541:1 Shanghai 2
VM541:1 Taiwan 3
undefine

// 遍历数组（三个参数：显示值、下标、数据来源）
list1.forEach(function(value, index, arr){console.log(value, index, arr)})
VM1237:1 Changsha 0 (4) ['Changsha', 'Beijing', 'Shanghai', 'Taiwan']
VM1237:1 Beijing 1 (4) ['Changsha', 'Beijing', 'Shanghai', 'Taiwan']
VM1237:1 Shanghai 2 (4) ['Changsha', 'Beijing', 'Shanghai', 'Taiwan']
VM1237:1 Taiwan 3 (4) ['Changsha', 'Beijing', 'Shanghai', 'Taiwan']
```

2） `splice()`替换数组元素（`先删除后添加`）

```javascript
// 定义数组
var ll = ["11", "22", "33"]
undefined

// 从下标0开始，删除2个元素
ll.splice(0, 2)
(2) ['11', '22']

// 查看效果
ll
['33']

// 插入元素
ll.push('66')
2
ll.push('88')
3
ll.push('99')
4

// 查看数组元素
ll
(4) ['33', '66', '88', '99']

// 从下标1开始，删除2个元素，替换为'9090'插入
ll.splice(1, 2, 9090)
(2) ['66', '88']
ll
(3) ['33', 9090, '99']

// 后续追加参数为替换后新增的元素
var ll2 = ['11', '22', '33', '44']
undefined
ll2.splice(0, 2, 'Hello', 'Man', 'HAHA')
(2) ['11', '22']

// 可见删除的两个元素被替换了
ll2
(5) ['Hello', 'Man', 'HAHA', '33', '44']

```

3）`map()`遍历数组元素

 ```javascript
// 定义数组
var ll = ["11", "22", "33"]
undefined

// 输出数组元素（方法和forEach类似，也可以最多三个参数）
ll.map(function(value){console.log(value)}, ll)
VM2621:1 11
VM2621:1 22
VM2621:1 33

// 三个参数输出数组
ll.map(function(value, index, aaa){console.log(value, index, aaa)}, ll)
VM3143:1 11 0 (3) ['11', '22', '33']
VM3143:1 22 1 (3) ['11', '22', '33']
VM3143:1 33 2 (3) ['11', '22', '33']

// 处理数组数据
ll.map(function(value){return value * 2}, ll)
(3) [22, 44, 66]
 ```



# 5 运算符

## 5.1 算术运算符

```javascript
var x = 10;
var res1 = x++;
var res2 = ++x;
res1 10
res2 12
++表示自增1 类似于 +=1
加号在前先加后赋值 加号在后先赋值后加
```

## 5.2 比较运算符

```javascript
1 == '1'  # 弱等于  内部自动转换成相同的数据类型比较了
true  

1 === '1'  # 强等于  内部不做类型转换

1 != '1'
false
1 !== '2'
true
```

## 5.3 逻辑运算符

”一定要注意到底什么时候返回的是布尔值 什么是返回的是数据“

```javascript
// 两个都为真，返回最后一个真的值
5 && '5'
'5'

// 左边为假，继续往后查找为真的值，并返回
0 || 1
1

// 遇到假直接返回假的值，!5 = false
!5 && '5'
false
```

**逻辑运算符的返回值问题**

```javascript
# && 且[运算符](https://so.csdn.net/so/search?q=运算符&spm=1001.2101.3001.7020)(从左到右只要找到无法通过的值[false]就结束，不再检查后面的值，可以通过的值[true]就继续向右检查直到最后一个)
console.log(5 && 4);//当结果为真时，返回最后一个为真的值4
console.log(4 && 5);//当结果为真时，返回最后一个为真的值5 
console.log(5 && 4 && 6);//当结果为真时，多个值时返回最后一个为真的值6

console.log(4 && 0);//当结果为假时，返回第一个为假的值0
console.log(0 && 4);//当结果为假时，返回第一个为假的值0
console.log(4 && 0 && false);//当结果为假时，多个值时返回第一个为假的值0 

console.log(false && 0);//当结果为假时，返回第一个为假的值false
console.log(0 && false);//当结果为假时，返回第一个为假的值0
console.log(false && 0 && false);//当结果为假时，多个值时返回第一个为假的值false

# || 或运算符(从左到右只要找到可以通过的值[true]就结束，不再检查后面的值)
console.log(0 || false);//当结果为假时，返回最后一个为假的值false
console.log(false || 0);//当结果为假时，返回最后一个为假的值0
console.log(false || 0 || false);//当结果为假时，返回最后一个为假的值false

console.log(2 || 0);//当结果为真时，返回真的值2
console.log(0 || 2);//当结果为真时，返回真的值2
console.log(0 || 4 || 3);//当结果为真时，多个值时返回第一个为真的值4

console.log(6 || 7);//当结果为真时，返回第一个为真的值6
console.log(7 || 6);//当结果为真时，返回第一个为真的值7
console.log(2 || 4 || 3);//当结果为真时，多个值时返回第一个为真的值2
```

## 5.4 赋值运算符

```javascript
= += -= *= ....
```



# 6 流程控制

## 6.1 if判断

```javascript
# if判断
var age = 28;
# if(条件){条件成立之后指向的代码块}
if (age>18){
  console.log('来啊 来啊')
}

# if-else
if (age>18){
  console.log('来啊 来啊')
}else{
  console.log('没钱 滚蛋')
}

# if-else if else
if (age<18){
  console.log("培养一下")
}else if(age<24){
  console.log('小姐姐你好 我是你的粉丝')
}else{
  console.log('你是个好人')
}
```

## 6.2 switch 语句

```javascript
var num = 2;
switch(num){
  case 0:
  	console.log('喝酒');
  	break;  # 不加break 匹配到一个之后 就一直往下执行
  case 1:
  	console.log('唱歌');
  	break;
  case 2:
  	console.log('洗脚');
  	break;
  case 3:
  	console.log('按摩');
  	break;
  case 4:
  	console.log('营养快线');
  	break;
  case 5:
  	console.log('老板慢走 欢迎下次光临');
  	break;
  default:
  	console.log('条件都没有匹配上 默认走的流程')
}
```

## 6.3 for循环

```javascript
# 打印0-9数字
for(let i=0;i<10;i++){
  console.log(i)
}
# 题目1  循环打印出数组里面的每一个元素
var l1 = [111,222,333,444,555,666]
for(let i=0;i<l1.length;i++){
  console.log(l1[i])
}
```

## 6.4 while循环

```javascript
var i = 0
while(i<100){
  console.log(i)
  i++;
}
```

## 6.5 三元运算符

三元运算符不要写的过于复杂 

```javascript
# python中三元运算符 res = 1 if 1>2 else 3
# JS中三元运算  res = 1 > 2 ? 1 : 3 
条件成立取问好后面的1 不成立取冒号后面的3
var res = 2 > 5 ? 8 : 10 # 10
var res = 2 > 5 ? 8 : ( 8 > 5 ? 666 : 444 )  # 666
```

