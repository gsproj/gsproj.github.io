---
title: 04-基本数据类型
date: 2022-07-18 9:16:22
categories:
- Python
- Python入门
tags:
---



“我们学习变量是为了让计算机能够像人一样去记忆事物的某种状态，而变量的值就是用来存储事物状态的，很明显事物的状态分成不同种类的（比如人的年龄，身高，职位，工资等等），所以变量值也应该有不同的类型”



## 1 数字类型

### 1.1 int整型

#### 

用来记录人的年龄，出生年份，学生人数等整数相关的状态

#### 1.1.2 定义

```python
age=18

birthday=1990

student_count=48
```

### 1.2 float浮点型

#### 1.2.1 作用

用来记录人的身高，体重，薪资等小数相关的状态

#### 1.2.2 定义

```python
height=172.3

weight=103.5

salary=15000.89
```

### 1.3 数字类型的使用

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

## 2 字符串类型str

### 2.1 作用

用来记录人的名字，家庭住址，性别等描述性质的状态

### 2.2 定义

```python
name = 'Tony'
address = '上海市浦东新区'
sex = '男'
```

用单引号、双引号、多引号，都可以定义字符串，本质上是没有区别的，但是

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

### 2.3 使用

```python
数字可以进行加减乘除等运算，字符串呢？也可以，但只能进行"相加"和"相乘"运算。
>>> name = 'tony'
>>> age = '18'
>>> name + age #相加其实就是简单的字符串拼接
'tony18'
>>> name * 5 #相乘就相当于将字符串相加了5次
'tonytonytonytonytony'
```



## 3 列表list

## 3.1 作用

如果我们需要用一个变量记录多个学生的姓名，用数字类型是无法实现，字符串类型确实可以记录下来，比如

stu_names='张三 李四 王五'，但存的目的是为了取，此时若想取出第二个学生的姓名实现起来相当麻烦，而列表类型就是专门用来记录多个同种属性的值（比如同一个班级多个学生的姓名、同一个人的多个爱好等），并且存取都十分方便

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



## 4 字典dict

### 4.1 作用

如果我们需要用一个变量记录多个值，但多个值是不同属性的，比如人的姓名、年龄、身高，用列表可以存，但列表是用索引对应值的，而索引不能明确地表示值的含义，这就用到字典类型，字典类型是用key：value形式来存储数据，其中key可以对value有描述性的功能

### 4.2 定义

```python
>>> person_info={'name':'tony','age':18,'height':185.3}
```

### 4.3 使用

```python
# 1、字典类型是用key来对应值，key可以对值有描述性的功能，通常为字符串类型
>>> person_info={'name':'tony','age':18,'height':185.3}
>>> person_info['name']
'tony'
>>> person_info['age']
18
>>> person_info['height']
185.3
# 2、字典可以嵌套，嵌套取值如下
>>> students=[
... {'name':'tony','age':38,'hobbies':['play','sleep']},
... {'name':'jack','age':18,'hobbies':['read','sleep']},
... {'name':'rose','age':58,'hobbies':['music','read','sleep']},
... ]
>>> students[1]['hobbies'][1] #取第二个学生的第二个爱好
'sleep'
```



## 5 布尔bool

### 5.1 作用

用来记录真假这两种状态

### 5.2 定义

```python
>>> is_ok = True
>>> is_ok = False
```

### 5.3 使用

```python
通常用来当作判断的条件，我们将在if判断中用到它
```



## 6 集合

### 6.1 作用

在Python中，集合（set）是无序的、元素不重复的集合

### 6.2 定义

```python
set01 = {'num1', 'num2'}
```

### 6.3 使用

```python
>>> admins = {'Justin', 'caterpillar'}
>>> users = {'momor', 'hamini', 'Justin'}
>>> 'Justin' in admins	# 使用in判断元素在集合
True
>>> admins & users
{'Justin'}
>>> admins | users
{'hamini', 'caterpillar', 'Justin', 'momor'}
>>> admins - users
{'caterpillar'}
>>> admins ^ users
{'hamini', 'caterpillar', 'momor'}
>>> admins > users
False
>>> admins < users
False
>>>
```



## 7 元组

### 7.1 作用

`tuple` 可以做什麼呢？有時想要傳回一組相關的值，又不想特地定義一個型態，就會使用 `tuple`，像是 `(1, 'Justin', True)`，也許就代表了從資料庫中臨時撈出來的一筆資料。

### 7.2 定义

`tuple` 就立后就无法变动，想要建立 `tuple`，只要在某个值后面加上「,」就可以。例如：

```python
>>> 10,
(10,)
>>> 10, 20, 30,
(10, 20, 30)
>>> acct = 1, 'Justin', True
>>> acct
(1, 'Justin', True)
>>> type(acct)
<class 'tuple'>
>>>
```

### 7.3 使用