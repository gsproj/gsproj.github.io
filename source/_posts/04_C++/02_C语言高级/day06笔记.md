---
title: Day06-C高级编程
date: 2022-2-16 11:05:22
categories:
- C++
- 02_C高级编程
tags:
---

“C高级编程第6天学习笔记”

## 一、数类型和函数指针

### 1.1 函数类型

什么是函数类型？

这个得从函数的三要素说起，函数由返回值、参数、函数名组成

而函数类型主要由它的返回值和参数类型决定，与函数名无关

比如在以下案例中：

```c
int myfunction(int a, int b);
```

函数类型是：`int (int a, int b);`

### 1.2 函数指针

#### 1.2.1 概念

那么什么又是函数指针？（简单一句话：指向函数的指针）

和其他指针类型一样，函数指针指向某种特定类型的指针，不过它指向的是函数而非对象

比如在以下案例中：

```c
int myfunction(int a, int b);
```

函数指针pf是：`int (*pf)(int a, int b);`

#### 1.2.1 函数名会自动转换成指针

当把函数作为一个值使用时，该函数会自动转换成指针，比如

```c
pf = myfunction;
// 等价于
pf = &myfunction;   
```

案例如下：

```c
#include <stdio.h>

typedef int(p)(int, int);

int my_func(int a, int b) {
	printf("%d %d\n", a ,b);
	return 0;
}

void test() {
	p p1;
	// p1(10, 20); // Error, 不能指直接调用，只描述了函数类型，但是没有定义函数体，没有函数体没法调用
	p* p2 = my_func; // 正确，函数自动转换成指针
	p* p3 = &my_func; // 正确，效果同上
	p2(10, 20); // 输出"10 20"
	p3(20, 30); // 输出"20 30"
```

#### 1.2.2 函数指针的定义方式

函数指针的定义方式有两种

- 先定义函数类型，更具类型定义函数指针变量
- 直接定义函数指针变量

案例如下：

```c
#include <stdio.h>

int myfunc(int a, int b) {
	printf("ret: %d\n", a + b);
	return 0;
}

// 方式一：先定义函数类型，通过类型定义指针
void test01() {
	typedef int(FUNC_TYPE)(int, int);
	FUNC_TYPE* f = myfunc;
	// 两种调用都可以
	f(10, 20); // ret:30
	(*f)(10, 20); // ret:30
}

// 方式二：直接定义函数指针变量
void test02() {	
	typedef int(*FUNC_TYPE)(int, int);
	FUNC_TYPE f = myfunc;
    // 两种调用都可以
	f(10, 20); 
	(*f)(10, 20);
}

int main() {
	test01();
	test02();
}
```

#### 1.2.3 函数指针数组

函数指针数组，里面每个元素都是函数指针，案例如下：

```c
#include <stdio.h>

// 定义函数
void func01(int a) {
	printf("func01: %d\n", a);
}

void func02(int a) {
	printf("func02: %d\n", a);
}

void func03(int a) {
	printf("func03: %d\n", a);
}

void test() {
	// 定义函数指针数组有两种方式
#if 0
	// 方式一：
	void(*func_array[](int)) = { func01, func02, func03 };
#else
	void(*func_array[3])(int);
	func_array[0] = func01;
	func_array[1] = func02;
	func_array[2] = func03;
#endif

	for (int i = 0; i < 3; i++) {
		// 调用函数
		func_array[i](10 + i);
		// 或者
		(*func_array[i])(20 + i);
	}
}

int main() {
	test();
	return 0;
}
```

### 1.3 回调函数

#### 1.3.1 概念

什么是回调函数？

将函数指针作为函数参数，即称为回调函数

比如：

```c
// 普通函数，形参为普通变量
void func01(int a){}
// 回调函数，形参为函数指针
void func02(int(*p)(int a)){}
```

那么把函数指针作为变量(回调函数)的作用是什么？

最常见的用途之一是传递函数的地址

#### 1.3.2 回调函数案例

简易计算器的案例

```c
/*
	计算器案例，学习回调函数
*/

#define _CRT_SECURE_NO_WARNINGS

#include <stdio.h>

// 定义函数
int plus(int a, int b) {
	return a + b;
}

int sub(int a, int b) {
	return a - b;
}

int mul(int a, int b) {
	return a * b;
}

int division(int a, int b) {
	return a / b;
}

// 回调函数
void calculator(int(*mycal)(int, int), int a, int b) {
	int ret = mycal(a, b);
	printf("ret: %d\n", ret);
}

void subcall(int flag, int num1, int num2) {
	
}

int main() {
	int select = 0;
	int num1 = 0;
	int num2 = 0;
	int flag = 1;
	while (flag != 0) {
		printf("---简易计算器---\n");
		printf("1.加法+\n");
		printf("2.减法-\n");
		printf("3.乘法*\n");
		printf("4.除法/\n");
		printf("5.退出\n");
		printf("请输入选择项：");
		scanf("%d", &select);

		if (select != 5) {
			printf("请输入第1个数: ");
			scanf("%d", &num1);
			printf("请输入第2个数: ");
			scanf("%d", &num2);
		}

		switch (select)
		{
		case 1:
			calculator(plus, num1, num2);
			break;
		case 2:
			calculator(sub, num1, num2);
			break;
		case 3:
			calculator(mul, num1, num2);
			break;
		case 4:
			calculator(division, num1, num2);
			break;
		case 5:
			flag = 0;
			break;
		default:
			break;
		}
	}

	return 0;
}
```

### 1.4 函数指针和指针函数

两个概念容易混淆

函数指针：

​	指向函数的指针

指针函数：

​	返回类型是指针的函数

## 二、预处理

### 2.1 基本概念

编译C语言源代码包含四个步骤：

1. 预处理：预处理器根据`#`开头的命令，修改C源代码生成`.i`文件
2. 编译：将C代码`.i`文件翻译成汇编代码`.s`文件
3. 汇编：汇编编译`.s`文件，生成二进制文件`.o`
4. 链接：链接各`.o`文件以及库文件，生成可执行程序

![image-20230217141741953](../../../img/image-20230217141741953.png)

### 2.2 什么是`#include`

#### 2.2.1 文件包含处理

什么是文件包含处理？

将`#include`指定的文件内容和当前源文件整合到一起，图示如下：

![image-20230217142520988](../../../img/image-20230217142520988.png) 

#### 2.2.3 #include<>和#include""的区别

- #include<file.h>
  - 按系统指定的目录检索文件
  - 常用于包含库函数的头文件
- #include "file.h"：
  - 现在file.c所在的当前目录找file.h，如果找不到，再按系统指定的目录检索。
  - 常用于包含自定义的头文件

理论上#include可以包含任意格式的文件(.c、.h等)，但一般用于头文件的包含。

2.3 宏定义