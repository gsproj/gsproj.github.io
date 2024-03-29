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

让程序员更精确地控制类型转换

**案例一：计算浮点数的小数部分**

```c
double d = 3.14, fraction;
fraction = d - (int)d;	// 强转为int类型
printf("fration = %.2lf\n", fraction);	// 输出0.14
```

**案例二：使代码可读性更高**

```c
float f = 3.14f;
int i = (int)f;		// 一目了然
```

**案例三：对类型转换进行更精确的控制**

```c
int d1 = 4, d2 = 3;
double q;
q = d1 / d2; 
printf("q = %lf\n", q); // 输出1.000000
q = (double)d1 / d2;
printf("q = %lf\n", q); // 输出1.333333
```

**案例四：避免数据溢出)**	(计算一天多少毫秒，多少纳秒

```c
// 一天多少毫秒
long long millisPerday = 24 * 60 * 60 * 1000;
// 一天多少纳秒
long long nanosPersday = 24 * 60 * 60 * 1000 * 1000 * 1000;
printf("%lld\n", nanosPersday / millisPerday);	// 结果不是10000，居然是-21？？
```

为什么结果是-21？

因为24、60、1000都是int类型的，相乘后**也是int类型**，在nanosPerday的乘法计算后，已经超出了int的范围（-2147483648到2147483647）

解决方法：强转为long long类型

```c
// 一天多少纳秒
long long nanosPersday = (long long)24 * 60 * 60 * 1000 * 1000 * 1000;
```



# 二、运算符

## 2.1 typedef	（TODO可以补充）

typedef用于给类型起别名，例如

```c
#include <stdio.h>

// 给int起别名Bool
typedef int Bool; 

// 其实也能用宏定义来起别名
#define Bool2 int

int main(void) {
	Bool b = 1;
	printf("b = %d\n", b);

	Bool2 b2 = 200;
	printf("b2 = %d\n", b2);
	return 0;
}
```

>Q1：当定义别名时，使用typdef和使用宏定义的区别是什么？
>
>宏定义是在预处理阶段做**简单的文本替换**，因此当编译器识别宏定义，当错误发生时不能给出一些友好的提示，而typedef定义的别名，是在编译阶段生效，可以在错误发生时给出提示
>
>**因此：定义类型的时候，请使用typedef**

使用变量起别名的好处：

- 增强代码可读性
- 增强大码可移植性

## 2.2 sizeof运算符

sizeof运算符用于计算某个数据类型所占的空间大小（单位：字节）

注意事项：

1. sizeof是运算符，不是函数
2. sizeof在编译期间进行计算，他是一个常量表达式，可以表示数组的长度，例如

```c
int i = 3;
int arr[sizeof(i)];
```



## 2.3 算术运算符(TODO公式有问题)

加(+)减(-)乘(*)除(/)、以及取模(%)

注意事项：

1. 其中加减乘除可以用于浮点数，但是取模要求两个操作数都是整数
2. 两个整数相除，其结果也是整数
3. `i%j`的结果可能为负数，符号与`i`的符号相同，满足公式：`（缺失）`，比如：

```c
4 % 3 = 1;
-4 % 3 = -1;
4 % -3 = 1;
-4 % -3 = -1;
```

取余的案例：判断一个数字是否为奇数

```c
bool is_odd(int n) {
	// return n % 2 == 1;	// 不推荐，不严谨，当n为负数时会出问题
	return n % 2 != 0;	// 推荐
	return n & 1; // 推荐，代码量最少，可读性较差
}
```



## 2.4 赋值运算符

### 2.4.1 简单赋值

案例

```c
float f;
int i;
f = i = 3.14f;

输出: 
f = 3.0 
i = 3
```

>注意事项：
>
>1. 赋值过程中可能有隐式转换，例如`int i = 3.14;`，隐式转换为int类型
>2. 赋值是从右到左的，例如：`a = b = 30;`

### 2.4.2 复合赋值

+=	-=	*=	/=	

案例：

```c
a += b
// 等同于
a = a + b
```

## 2.5 自增和自减运算符

自增表示为++，自减表示为--：

- i++：表达式的值为i，副作用是i自增
- ++i：表达式的值为（i++），副作用是i自增

自增的案例：

```c
int i, j, k;
i = 1;
j = 2;
k = ++i + j++;

printf("i = %d, j = %d, k = %d\n", i, j, k);
// 2 3 4
```



## 2.6 关系运算符

包含：小于<，大于，大于等于>=，小于等于<=

>注意事项：
>
>`i < j < k`怎么计算？
>
>先计算（i < j），得到的结果（0/1）后，再计算（0/1）< k，因此，
>
>正确的写法是：`j < i && j< k;`



## 2.7 判等运算符

包含：==，!=

其运算结果要么为0，要么为1



## 2.8 逻辑运算符

包含：与&&，或||，非！

>注意事项：
>
>`&&`和`||`会发生短路现象，例如：
>
>1. `e1 && e2`：先计算e1表达式，若e1为false，则不再计算e2
>2. `e1 || e2`：先计算e1表达式，若e1为true，则不再计算e2



## 2.9 位运算符（TODO需要补充位运算的知识）

分为：

- 左移<<
- 右移>>
- 与
- 或|
- 非^
- 异或~：相同为0，不同为1

### 2.9.1 左移右移

1、左移案例：

` i << j`，表示将i左移j位，在右边补0

```c
short s = 13;
printf("%d\n", s << 2);	// 输出52
```

二进制数13，用位表示：00001101，左移2位之后，是00110100，转换后为4+16+32 = 52，若没有发生溢出，**左移j位，相当于乘以2^j**，如13左移2位 = 13 * 2 ^ 2 = 13 * 4 = 52



2、右移案例

`i >> j`，表示将i右移j位，若**i为无符号数，或者非负数**，则左补0，**若i为负数**，它的行为由实现定义，有的左补0，有的左补1

```c
short s = 13;
printf("%d\n", s >> 2);	// 输出3
```

### 2.9.2 按位运算

**1、按位取反(~)**

```c
short i = 3, j = 4;
printf("i = %d, j = %d\n", ~i, ~j);	// -4 -5
```

以i为例：0000 0000 0000 0000 0011，取反之后为 1111 1111 1111 1100，结果是-4	（TODO 我不知道怎么算的，而且怎么一下是8位，一下是16位！！）

**2、异或的几个特性**

```c
a ^ 0 = a;
a ^ a = a;
a ^ b = b ^ a;	// 交换性
a ^ ( b ^ c ) = ( a ^ b ) ^ c // 结合性
```

**3、使用案例：**

*案例1*：如何判断一个数是否是奇数：

```c
bool is_odd(int a) {
	return a & 0x1;	// 与1按位与
}
```

*案例2*：判断一个数是否为2的幂数【TODO 补充方法1】

方法1：

```c
bool isPowerOf2(unsigned int num) 
```

方法2：利用特性，2的幂次只有1一个1

```c
bool isPowerOf2_2(unsigned int num) {
	return (num & num - 1) == 0;
}
```

*案例3*：给定一个不为0的整数，找出值为1，且权重最低的位，如输入0011 0100，输出4

```c
// 两种方法都可以
((n ^ n - 1) + 1) >> 1
n & (-n)
```

*案例4*：给定一个整数数组，里面的数都是成对的，只有一个数例外，请找出它：

```c
int findSigleNum(int arr[], int n) {
	int sigleNum = 0;
	for (int i = 0; i < n; i++) {
		sigleNum ^= arr[i];
	}
	return sigleNum;
}

// 测试
int arr[10] = { 1,1,2,2,4,5,5,6,6 };
printf("%d\n", findSigleNum(arr, 10));	// 4
```





