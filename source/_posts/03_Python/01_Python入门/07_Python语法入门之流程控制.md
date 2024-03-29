---
title: 07-流程控制
date: 2022-07-18 9:19:22
categories:
- Python
- 01_Python入门
tags:
---

# 1 引子：

流程控制即控制流程，具体指控制程序的执行流程，而程序的执行流程分为三种结构：

- 顺序结构（之前我们写的代码都是顺序结构）
- 分支结构（用到if判断）
- 循环结构（用到while与for）

![img](https://pic3.zhimg.com/80/v2-1c94b67e61036b4b8029f1ff0e04021e_720w.jpg)

# 2 分支结构

## 2.1 什么是分支结构

分支结构就是根据条件判断的真假去执行不同分支对应的子代码

## 2.2 为什么要用分支结构

人类某些时候需要根据条件来决定做什么事情，比如：如果今天下雨，就带伞

所以程序中必须有相应的机制来控制计算机具备人的这种判断能力

## 2.3 如何使用分支结构

### 2.3.1 if语法

![img](https://pic3.zhimg.com/80/v2-a75171615cb658ae9bb3db7afabc6846_720w.jpg)

用if关键字来实现分支结构，完整语法如下

```python
if 条件1:   # 如果条件1的结果为True，就依次执行：代码1、代码2，......
  　代码1
    代码2
    ......
elif 条件2: # 如果条件2的结果为True，就依次执行：代码3、代码4，......
  　代码3
    代码4
    ......
elif 条件3: # 如果条件3的结果为True，就依次执行：代码5、代码6，......
  　代码5
    代码6
    ......
else:　　   # 其它情况，就依次执行：代码7、代码8，......
    代码7
    代码8
    ......
# 注意：
# 1、python用相同缩进(4个空格表示一个缩进)来标识一组代码块，同一组代码会自上而下依次运行
# 2、条件可以是任意表达式，但执行结果必须为布尔类型
     # 在if判断中所有的数据类型也都会自动转换成布尔类型
       # 2.1、None，0，空（空字符串，空列表，空字典等）三种情况下转换成的布尔值为False
       # 2.2、其余均为True
```

### 2.3.2 if应用案例

![img](https://pic4.zhimg.com/80/v2-0dff1efd05c82779f08db912b84ed5a3_720w.jpg)

案例1：

如果：女人的年龄>30岁，那么：叫阿姨

```python
age_of_girl = 31
if age_of_girl > 30:
    print("阿姨")
```

案例2：

如果：女人的年龄>30岁，那么：叫阿姨，否则：叫小姐姐

```text
age_of_girl = 29
if age_of_girl > 30:
    print("阿姨")
else:
    print("小姐姐")
```

案例3：

如果：女人的年龄>=18并且<22岁并且身高>170并且体重<100并且是漂亮的，那么：表白，否则：叫阿姨**

```text
age_of_girl=18
height=171
weight=99
is_pretty=True
if age_of_girl >= 18 and age_of_girl < 22 and height > 170 and weight < 100 and is_pretty == True:
    print('表白...')
else:
    print('阿姨好')
```

案例4：

如果：成绩>=90，那么：优秀

如果成绩>=80且<90,那么：良好

如果成绩>=70且<80,那么：普通

其他情况：很差

```python
score=input('>>: ')
score=int(score)

if score >= 90:
    print('优秀')
elif score >= 80:
    print('良好')
elif score >= 70:
    print('普通')
else:
    print('很差')
```

案例5：if嵌套

```text
#在表白的基础上继续：
#如果表白成功，那么：在一起
#否则：打印。。。

age_of_girl=18
height=171
weight=99
is_pretty=True
success=False

if age_of_girl >= 18 and age_of_girl < 22 and height > 170 and weight < 100 and is_pretty == True:
    if success:
        print('表白成功,在一起')
    else:
        print('什么爱情不爱情的,爱nmlgb的爱情,爱nmlg啊...')
else:
    print('阿姨好')
```

![img](https://pic3.zhimg.com/80/v2-3a90716a58a4458640aee154894b0bd2_720w.jpg)

练习1: 登陆功能

```python
username = input("请输入用户名：").strip()
password = input("请输入用户密码：").strip()

if username == 'admin' and password == 'local666':
    print("老铁，登录成功")
else:
    print("滚蛋，登录失败！")
```

练习2：

```python
#!/usr/bin/env python
#根据用户输入内容打印其权限

'''
egon --> 超级管理员
tom  --> 普通管理员
jack,rain --> 业务主管
其他 --> 普通用户
'''
name = input('请输入用户名字：')

if name == 'egon':
    print('超级管理员')
elif name == 'tom':
    print('普通管理员')
elif name == 'jack' or name == 'rain':
    print('业务主管')
else:
    print('普通用户')
```

# 3 循环结构

## 3.1 什么是循环结构

循环结构就是重复执行某段代码块

## 3.2 为什么要用循环结构

![img](https://pic4.zhimg.com/80/v2-f174b482ecf2583633bd31e3a13f8e8b_720w.jpg)

人类某些时候需要重复做某件事情

所以程序中必须有相应的机制来控制计算机具备人的这种循环做事的能力

## 3.3 如何使用循环结构

### 3.3.1 while循环语法

![img](https://pic4.zhimg.com/80/v2-43c2d731786fa1b75f5e45b55f0cf833_720w.jpg)

python中有while与for两种循环机制，其中while循环称之为条件循环，语法如下

```python
while 条件:
     代码1     
     代码2     
     代码3
while的运行步骤：
步骤1：如果条件为真，那么依次执行：代码1、代码2、代码3、......
步骤2：执行完毕后再次判断条件,如果条件为True则再次执行：代码1、代码2、代码3、......，如果条件为False,则循环终止
```

![img](https://pic3.zhimg.com/80/v2-6f56fdbd7d92dd2a7ce02573d950136a_720w.jpg)

### 3.3.2 while循环应用案例

![img](https://pic4.zhimg.com/80/v2-0dff1efd05c82779f08db912b84ed5a3_720w.jpg)

案例一：while循环的基本使用

用户认证程序

![img](https://pic2.zhimg.com/80/v2-72163eb478e754fbfd478839c5cbf845_720w.jpg)

```python
#用户认证程序的基本逻辑就是接收用户输入的用户名密码然后与程序中存放的用户名密码进行判断，判断成功则登陆成功，判断失败则输出账号或密码错误
username = "jason"
password = "123"

inp_name =  input("请输入用户名：")
inp_pwd =  input("请输入密码：")
if inp_name == username and inp_pwd == password:
    print("登陆成功")
else:
    print("输入的用户名或密码错误！")
#通常认证失败的情况下，会要求用户重新输入用户名和密码进行验证，如果我们想给用户三次试错机会，本质就是将上述代码重复运行三遍，你总不会想着把代码复制3次吧。。。。
username = "jason"
password = "123"

# 第一次验证
inp_name =  input("请输入用户名：")
inp_pwd =  input("请输入密码：")
if inp_name == username and inp_pwd == password:
    print("登陆成功")
else:
    print("输入的用户名或密码错误！")

# 第二次验证
inp_name =  input("请输入用户名：")
inp_pwd =  input("请输入密码：")
if inp_name == username and inp_pwd == password:
    print("登陆成功")
else:
    print("输入的用户名或密码错误！")

# 第三次验证
inp_name =  input("请输入用户名：")
inp_pwd =  input("请输入密码：")
if inp_name == username and inp_pwd == password:
    print("登陆成功")
else:
    print("输入的用户名或密码错误！")

#即使是小白的你，也觉得的太low了是不是，以后要修改功能还得修改3次，因此记住，写重复的代码是程序员最不耻的行为。
#那么如何做到不用写重复代码又能让程序重复一段代码多次呢？ 循环语句就派上用场啦（使用while循环实现）

username = "jason"
password = "123"
# 记录错误验证的次数
count = 0
while count < 3:
    inp_name = input("请输入用户名：")
    inp_pwd = input("请输入密码：")
    if inp_name == username and inp_pwd == password:
        print("登陆成功")
    else:
        print("输入的用户名或密码错误！")
        count += 1
```

案例二：while+break的使用

使用了while循环后，代码确实精简多了，但问题是用户输入正确的用户名密码以后无法结束循环，那如何结束掉一个循环呢？这就需要用到break了！

```python
username = "jason"
password = "123"
# 记录错误验证的次数
count = 0
while count < 3:
    inp_name = input("请输入用户名：")
    inp_pwd = input("请输入密码：")
    if inp_name == username and inp_pwd == password:
        print("登陆成功")
        break # 用于结束本层循环
    else:
        print("输入的用户名或密码错误！")
        count += 1
```

案例三：while循环嵌套+break

如果while循环嵌套了很多层，要想退出每一层循环则需要在每一层循环都有一个break

```python
username = "jason"
password = "123"
count = 0
while count < 3:  # 第一层循环
    inp_name = input("请输入用户名：")
    inp_pwd = input("请输入密码：")
    if inp_name == username and inp_pwd == password:
        print("登陆成功")
        while True:  # 第二层循环
            cmd = input('>>: ')
            if cmd == 'quit':
                break  # 用于结束本层循环，即第二层循环
            print('run <%s>' % cmd)
        break  # 用于结束本层循环，即第一层循环
    else:
        print("输入的用户名或密码错误！")
        count += 1
```

案例四：while循环嵌套+tag的使用

针对嵌套多层的while循环，如果我们的目的很明确就是要在某一层直接退出所有层的循环，其实有一个窍门，就让所有while循环的条件都用同一个变量，该变量的初始值为True，一旦在某一层将该变量的值改成False，则所有层的循环都结束

```python
username = "jason"
password = "123"
count = 0

tag = True
while tag: 
    inp_name = input("请输入用户名：")
    inp_pwd = input("请输入密码：")
    if inp_name == username and inp_pwd == password:
        print("登陆成功")
        while tag:  
            cmd = input('>>: ')
            if cmd == 'quit':
                tag = False  # tag变为False， 所有while循环的条件都变为False 
                break
            print('run <%s>' % cmd)
        break  # 用于结束本层循环，即第一层循环
    else:
        print("输入的用户名或密码错误！")
        count += 1
```

案例五：while+continue的使用

break代表结束本层循环，而continue则用于结束本次循环，直接进入下一次循环

```python
# 打印1到10之间，除7以外的所有数字
number=11
while number>1:
    number -= 1
    if number==7:
        continue # 结束掉本次循环，即本次循环continue之后的代码都不会运行了，而是直接进入下一次循环
    print(number)
```

案例五：while+else的使用

在while循环的后面，我们可以跟else语句，当while 循环正常执行完并且中间没有被break 中止的话，就会执行else后面的语句，所以我们可以用else来验证，循环是否正常结束

```python
count = 0
while count <= 5 :
    count += 1
    print("Loop",count)
else:
    print("循环正常执行完啦")
print("-----out of while loop ------")
输出
Loop 1
Loop 2
Loop 3
Loop 4
Loop 5
Loop 6
循环正常执行完啦   #没有被break打断，所以执行了该行代码
-----out of while loop ------
```

如果执行过程中被break，就不会执行else的语句

```python
count = 0
while count <= 5 :
    count += 1
    if count == 3:
        break
    print("Loop",count)
else:
    print("循环正常执行完啦")
print("-----out of while loop ------")
输出
Loop 1
Loop 2
-----out of while loop ------ #由于循环被break打断了，所以不执行else后的输出语句
```

![img](https://pic3.zhimg.com/80/v2-3a90716a58a4458640aee154894b0bd2_720w.jpg)

练习1：

寻找1到100之间数字7最大的倍数（结果是98）

```python
number=100
while number>0:
    if number%7==0:
        print(number)
        break
    number-=1
```

练习2：

```python
age=18
count=0
while count<3:
    count+=1
    guess = int(input(">>:"))
    if guess > age :
        print("猜的太大了，往小里试试...")
    elif guess < age :
        print("猜的太小了，往大里试试...")
    else:
        print("恭喜你，猜对了...")
```

### 3.3.3 for循环语法

![img](https://pic3.zhimg.com/80/v2-3a90716a58a4458640aee154894b0bd2_720w.jpg)

循环结构的第二种实现方式是for循环，for循环可以做的事情while循环都可以实现，之所以用for循环是因为在循环取值（即遍历值）时for循环比while循环的使用更为简洁，

**for循环语法如下**

```python
for 变量名 in 可迭代对象: # 此时只需知道可迭代对象可以是字符串\列表\字典，我们之后会专门讲解可迭代对象
    代码一
    代码二
    ...

#例1
for item in ['a','b','c']:
    print(item)
# 运行结果
a
b
c

# 参照例1来介绍for循环的运行步骤
# 步骤1：从列表['a','b','c']中读出第一个值赋值给item（item=‘a’），然后执行循环体代码
# 步骤2：从列表['a','b','c']中读出第二个值赋值给item（item=‘b’），然后执行循环体代码
# 步骤3: 重复以上过程直到列表中的值读尽
```

![img](https://pic1.zhimg.com/80/v2-7d7dc5ae41a09a7cd6e043d75c63b81c_720w.jpg)

### 3.3.4 for循环应用案例

![img](https://pic4.zhimg.com/80/v2-0dff1efd05c82779f08db912b84ed5a3_720w.jpg)

案例一：打印数字0-5

```python
# 简单版：for循环的实现方式
for count in range(6):  # range(6)会产生从0-5这6个数
    print(count)

# 复杂版：while循环的实现方式
count = 0
while count < 6:
    print(count)
    count += 1
```

案例二：遍历字典

```python
# 简单版：for循环的实现方式
for k in {'name':'jason','age':18,'gender':'male'}:  # for 循环默认取的是字典的key赋值给变量名k
    print(k)

# 复杂版：while循环确实可以遍历字典，后续将会迭代器部分详细介绍
```

案例三：for循环嵌套

```python
#请用for循环嵌套的方式打印如下图形：
*****
*****
*****

for i in range(3):
    for j in range(5):
        print("*",end='')
    print()  # print()表示换行
```

注意：break 与 continue也可以用于for循环，使用语法同while循环

![img](https://pic3.zhimg.com/80/v2-3a90716a58a4458640aee154894b0bd2_720w.jpg)

练习一：

打印九九乘法表

```python
for i in range(1,10):
    for j in range(1,i+1):
        print('%s*%s=%s' %(i,j,i*j),end=' ')
    print()
```

练习二：

打印金字塔

![img](https://pic3.zhimg.com/80/v2-53598748f490542a9700e63433b79496_720w.jpg)

```python
# 分析
'''
#max_level=5
     *        # current_level=1，空格数=4，*号数=1
    ***       # current_level=2,空格数=3,*号数=3
   *****      # current_level=3,空格数=2,*号数=5
  *******     # current_level=4,空格数=1,*号数=7
 *********    # current_level=5,空格数=0,*号数=9

# 数学表达式
空格数=max_level-current_level
*号数=2*current_level-1
'''
# 实现：
max_level=5
for current_level in range(1,max_level+1):
    for i in range(max_level-current_level):
        print(' ',end='') #在一行中连续打印多个空格
    for j in range(2*current_level-1):
        print('*',end='') #在一行中连续打印多个空格
    print()
```

## 视频链接：

1、if判断

[https://www.bilibili.com/video/av73342471/?p=10www.bilibili.com/video/av73342471/?p=10](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/av73342471/%3Fp%3D10)

2、while循环

[https://www.bilibili.com/video/av73342471?p=11www.bilibili.com/video/av73342471?p=11](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/av73342471%3Fp%3D11)