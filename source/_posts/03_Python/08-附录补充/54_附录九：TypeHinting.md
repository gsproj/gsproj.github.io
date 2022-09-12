---
title: 54-附录九：Type Hinting
date: 2022-07-18 10:54:22
categories:
- Python
- 12_附录补充
tags:
---

类型提示 **Type hinting**（最低Python版本为3.5）

python3新增类型提示功能，例如我们可以为函数增加类型提示信息，而不影响函数本身的执行：

注释的一般规则是参数名后跟一个冒号（：），然后再跟一个expression，这个expression可以是任何形式。

```text
def func(a: 'spam', b: (1, 10), c: float) -> int:
    return a + b + c
 
>>> func(1, 2, 3)
6
```

返回值的形式是 -> int，annotation可被保存为函数的attributes。查看所有的annotation，可通过如下语句

```text
>>> func.__annotations__
{'c': <class 'float'>, 'a': 'spam', 'b': (1, 10), 'return': <class 'int'>}
```

如果为函数增加了注释，可不可以继续使用默认参数呢？答案是肯定的。

```text
>>> def func(a: 'spam' = 4, b: (1, 10) = 5, c: float = 6) -> int:
...   return a + b + c
... 
>>> func(1, 2, 3)
6
>>> func()
15
>>> func(1, c=10)
16
>>> func.__annotations__
{'c': <class 'float'>, 'a': 'spam', 'b': (1, 10), 'return': <class 'int'>}
```