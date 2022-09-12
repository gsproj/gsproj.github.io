---
title: 04-基本数据类型
date: 2022-07-18 9:16:22
categories:
- Python
- 01_Python入门
tags:
---

"Python中的数据类型笔记"



# 数据类型简介

为应对不同的业务需求，Python中也把数据分为不同类型

![image-20220912100558673](../../../img/image-20220912100558673.png)

检测数据类型的方法：`type()`，比如

```python
a = 1
print(type(a))  # <class 'int'> -- 整型

b = 1.1
print(type(b))  # <class 'float'> -- 浮点型

c = True
print(type(c))  # <class 'bool'> -- 布尔型

d = '12345'
print(type(d))  # <class 'str'> -- 字符串

e = [10, 20, 30]
print(type(e))  # <class 'list'> -- 列表

f = (10, 20, 30)
print(type(f))  # <class 'tuple'> -- 元组

h = {10, 20, 30}
print(type(h))  # <class 'set'> -- 集合

g = {'name': 'TOM', 'age': 20}
print(type(g))  # <class 'dict'> -- 字典
```



# 1 数字类型

## 1.1 int整型

用来记录人的年龄，出生年份，学生人数等整数相关的状态

案例如下：

```python
age = 18
birthday = 1990
student_count = 48
```

## 1.2 float浮点型

用来记录人的身高，体重，薪资等小数相关的状态

案例如下：

```python
height = 172.3
weight = 103.5
salary = 15000.89
```

## 1.3 数字类型的使用

1 、数学运算

```python
>>> a = 1
>>> b = 3
>>> c = a + b
>>> c
4
```

2、比较大小

```python
>>> x = 10
>>> y = 11
>>> x > y
False
```

# 2 字符串类型String

作用：用来记录人的名字，家庭住址，性别等描述性质的状态

字符串可以用`单引号`、`双引号`或者`多引号`定义，案例如下：

```python
name = 'Tony'
address = '上海市浦东新区'
sex = '男'
hobby = "篮球"
age = '''
	18岁
	狗年生
'''
```

PS：各种引号定义字符串的注意事项

```python
#1、需要考虑引号嵌套的配对问题
msg = "My name is Tony , I'm 18 years old!" #内层有单引号，外层就需要用双引号

#2、多引号可以写多行字符串
msg = '''
        天下只有两种人。比如一串葡萄到手，一种人挑最好的先吃，另一种人把最好的留到最后吃。
        照例第一种人应该乐观，因为他每吃一颗都是吃剩的葡萄里最好的；第二种人应该悲观，因为他每吃一颗都是吃剩的葡萄里最坏的。
        不过事实却适得其反，缘故是第二种人还有希望，第一种人只有回忆。
'''
```

## 2.1 字符串的特殊使用

```python
数字可以进行加减乘除等运算，字符串呢？也可以，但只能进行"相加"和"相乘"运算。
>>> name = 'tony'
>>> age = '18'
>>> name + age #相加其实就是简单的字符串拼接
'tony18'
>>> name * 5 #相乘就相当于将字符串相加了5次
'tonytonytonytonytony'
```



# 3 列表list[]

## 3.1 作用

用一个变量记录多个学生的姓名，用数字类型是无法实现，字符串类型确实可以记录下来，比如

stu_names='张三 李四 王五'，但存的目的是为了取，此时若想取出第二个学生的姓名实现起来相当麻烦，而列表

类型就是专门用来记录多个同种属性的值（比如同一个班级多个学生的姓名、同一个人的多个爱好等），并且存取

都十分方便

## 3.2 定义

```python
>>> stu_names=['张三','李四','王五']
```

## 3.3 使用

```python
# 1、列表类型是用索引来对应值，索引代表的是数据的位置，从0开始计数
>>> stu_names=['张三','李四','王五']
>>> stu_names[0] 
'张三'
>>> stu_names[1]
'李四'
>>> stu_names[2]
'王五'

# 2、列表可以嵌套，嵌套取值如下
>>> students_info=[['tony',18,['jack',]],['jason',18,['play','sleep']]]
>>> students_info[0][2][0] #取出第一个学生的第一个爱好
'play'
```



# 4 字典dict{}

## 4.1 作用

如果我们需要用一个变量记录多个值，但多个值是不同属性的，比如人的姓名、年龄、身高，用列表可以存，但列

表是用索引对应值的，而索引不能明确地表示值的含义，这就用到字典类型，字典类型是用key：value形式来存储

数据，其中key可以对value有描述性的功能

## 4.2 定义

```python
>>> person_info={'name':'tony', 'age':18, 'height':185.3}
```

## 4.3 使用

```python
# 1、字典类型是用key来对应值，key可以对值有描述性的功能，通常为字符串类型
>>> person_info={'name':'tony','age':18,'height':185.3}
>>> person_info['name']
'tony'
>>> person_info['age']
18
>>> person_info['height']
185.3

# 2、字典可以嵌套，嵌套取值如下(用列表套字典)
>>> students=[
... {'name':'tony','age':38,'hobbies':['play','sleep']},
... {'name':'jack','age':18,'hobbies':['read','sleep']},
... {'name':'rose','age':58,'hobbies':['music','read','sleep']},
... ]
>>> students[1]['hobbies'][1] #取第二个学生的第二个爱好
'sleep'
```



# 5 布尔bool

## 5.1 作用

用来记录真假这两种状态

## 5.2 定义

```python
>>> is_ok = True
>>> is_ok = False
```

## 5.3 使用

```python
通常用来当作判断的条件，我们将在if判断中用到它
```



# 6 集合set{}

## 6.1 作用

在Python中，集合（set）是无序的、元素不重复的集合

## 6.2 定义

```python
set01 = {'num1', 'num2'}
```

## 6.3 集合运算

```python
# 1、定义集合
>>> admins = {'Justin', 'caterpillar'}
>>> users = {'momor', 'hamini', 'Justin'}

# 2、使用in判断元素在集合
>>> 'Justin' in admins	
True

# 3、交集
>>> admins & users
{'Justin'}

# 4、并集
>>> admins | users
{'hamini', 'caterpillar', 'Justin', 'momor'}

# 差集
>>> admins - users
{'caterpillar'}

# 对称集
>>> admins ^ users
{'hamini', 'caterpillar', 'momor'}

# 集合比大小
>>> admins > users
False
>>> admins < users
False
>>>
```



# 7 元组tuple()

## 7.1 作用

`tuple` 可以做什么？有时候想返回一组值，但是这组值又不想定义某个类型，就可以使用元组

## 7.2 定义

`tuple` 定义后，里面的值就无法修改

```python
# 1、创建元组的方式-1
>>> 10,
(10,)
>>> 10, 20, 30,
(10, 20, 30)

# 2、创建元组的方式-2
>>> mytup=(1, 2, 3)
>>> mytup
(1, 2, 3)

>>> type(mytup)
<class 'tuple'>

# 尝试修改元组的值
>>> mytup[0] = 3
## 会报错
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

