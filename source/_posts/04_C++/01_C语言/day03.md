---
title: C语言-Day03
date: 2024-03-07 15:36:22
categories:
- C++
- 01_C语言
tags:
---

“C语言部分-Day03”

# 一、类型转换

为什么需要类型转换？

- 计算机硬件只能对“相同类型”的数据进行运算

何时会发生类型转换？

- 当给定的数据类型和需要的数据类型不匹配时

如何进行类型转换？

- 隐式类型转换：编译器做的类型转换
- 显示类型转换：又叫强制类型转换，是程序员手动进行的转换



## 1.1 隐式类型转换

隐式类型转换默认存在两条规则

第一条：

```c
short,char --> int --> long --> long long --> float --> double
```

第二条:

```c
signed --> unsigned
```

案例：

```c
int i = -1;
unsigned int u = 100;

if (i < u) {
    printf("i  is less than u\n");
}
else {
    printf("i  is greater than u\n"); // 是它 why?
}
```

此案例的结果输出为"i  is greater than u"，因为当`变量i`去和`unsigned int类型的变量u`对比时，`变量i`会隐式转换为`unsigned int类型`：i十六进制表示为0xFFFFFFFF，u的十六进制表示为0x00000064，因此i比u大

**避免这样使用！**



## 1.2 强制类型转换

