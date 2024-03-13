---
title: C语言-Day04
date: 2024-03-011 15:36:22
categories:
- C++
- 01_C语言
tags:
---

“C语言部分-Day04”

# 一、语句

## 1.1 选择语句

### 1.1.1 if语句

if语句的案例：

```c
if (num == 0) {
    printf("num == 0");
}
```

if...else if...else语句的案例：

```c
	if (num == 0) {
		printf("num == 0");
	}
	else if (num == 1) {
		printf("num == 1");
	}
	else {
		printf("num is error");
	}
```

### 1.1.2 switch语句

switch语句的案例：

```c
switch (grade) {
	case 0:
		printf("0000000");
		break;
	case 1:
		printf("1111111");
		break;
	default:	// 如输入的参数不匹配，缺省值
		break;
	}
```

>注意事项：
>
>1、expr（即案例中的grade）的值必须是int类型或者char类型
>
>2、case后面的值也必须是int类型或者char类型的常量表达式
>
>3、不能使用重复的标签
>
>4、多个分支标号可以共用一条语句，例如：
>
>```c
>case 1: case 2: case 3:
>		printf("1111111");
>		break;
>```
>
>5、如果缺省`break`可能出现case穿透的现象，例如
>
>```c
>switch (grade) {
>	case 0:
>		printf("0000000");	// 省略break
>	case 1: case 2: case 3:
>		printf("1111111");	// 省略break
>	default:
>   printf("都执行到末尾了！！");
>		break;
>	}
>
>// 执行发生穿透现象，后面的case也继续执行完了
>请输入数字: 0
>00000001111111都执行到末尾了！！
>```



**switch语句和if...else语句的比较：**

1. if...else语句比switch语句更加通用
2. 但是switch语句比if....else语句可读性高
3. switch语句的执行效率优于if...else语句



## 1.2 循环语句

### 1.2.1 while语句

while语句的案例：

```c
int main(void) {
	int num = 10;
	while (num > 0) {	// while设置条件
		printf("%d ", num);
		num--;
	}
	printf("\n");
	return 0;
}

// 执行
10 9 8 7 6 5 4 3 2 1
```



### 1.2.2 do...while语句

do..while语句的案例，将while语句的案例重写：

```c
int num = 10;

do {
    printf("%d ", num);
    num--;
} while (num > 0);

printf("\n");

// 执行：
10 9 8 7 6 5 4 3 2 1
```

那么do...while和while的区别是什么？

- 唯一区别：当初始条件为假时，do..while语句循环会执行一次，而while语句一次都不会执行，例如：

do..while会执行一次：

```c
int num = 10;

do {
    printf("%d ", num);
    num--;
} while (num > 0);

printf("\n");

// 执行输出10，说明执行了一次
10
```

while一次也不执行：

```c
int num = 10;

while (num < 0) {
	printf("%d ", num);
	num--;
}
printf("\n");

// 执行输出空白

```

### 1.2.3 for语句

for语句的案例：

```c
int main(void) {

	for (int i = 0; i < 10; i++) {
		printf("%d ", i);
	}
	printf("\n");

	return 0;
}

// 执行
0 1 2 3 4 5 6 7 8 9
```

>注意事项：
>
>for语句的expr1、expr2、expr3都可以省略，若省略expr2，其默认为true
>
>[不建议单独省略]

for语句的惯用法：

```c
// 无限循环
for (;;) {
    printf("hello\n");
}

// 效果等同于
while(1) {
    printf("hello\n");
}
```



## 1.3 跳转语句

### 1.3.1 break语句

用于跳出switch、for、while、do...while、for等语句，例子

```c
int sum = 0;

for (;;) {
    if (sum == 5) {
        break;	// 达到条件，跳出循环
    }
    printf("hello\n");
    sum++;
}

// 执行：
hello
hello
hello
hello
hello
```

>注意事项：
>
>当switch、while、do..while、for**语句嵌套时**，break语句**只能跳出包含break的最内层嵌套**，例如：
>
>```c
>	int sum = 0;
>
>	while (1) {
>		for (;;) {
>			if (sum == 5) {
>				break;	// 达到条件只能跳出for的无限循环
>			}
>			printf("hello\n");
>			sum++;
>		}
>		printf("Don't Break!!\n");	// while的无限循环仍会继续。。
>	}
>```

### 1.3.2 continue语句

continue语句和break语句的区别：

- break可以用于循环语句和switch语句，而continue语句只能用于循环语句
- break**语句是跳出整个循环**（循环彻底结束），而continue**语句是跳转到循环体的末尾**(结束当前循环，开启下一次循环)

continue语句的案例：

```c
for (;;) {
    if (sum == 4) {
        continue;	// 当满足条件num == 4后，程序执行被continue截胡，每次都跳过后面的语句，直接回到这里，因此一直卡在这
    }

    if (sum == 5) {
        break;	// 如果只有break，应该输出5次hello
    }

    printf("hello %d\n", sum);
    sum++;
}

// 执行
hello 0
hello 1
hello 2
hello 3
卡住，程序没有结束
```



### 1.3.3 go to语句

break语句只能跳转到switch或循环语句的下一条语句，continue只能跳转到循环体的末尾，而go to语句没有上面的限制，**能在同一函数内随意跳转**：

使用场景一，跳出外层嵌套：

```c
int sum = 0;

while (1) {
    for (;;) {
        if (sum == 4) {
            goto loop_done;	// 跳出到标签处
        }
        printf("sum = %d\n", sum);
        sum++;
    }
}
loop_done:	// 自定义标签

// 执行，能跳出while的无限循环
sum = 0
sum = 1
sum = 2
sum = 3
```

使用场景二，用于错误处理

```c
	while (1) {
		for (;;) {
			if (sum == 4) {
				goto error_handle;	// 加入发生错误，直接跳转到错误处理
			}
			printf("sum = %d\n", sum);
			sum++;
		}
	}
error_handle:	// 处理错误
	printf("发生错误了！");


// 执行
sum = 0
sum = 1
sum = 2
sum = 3
发生错误了！
```

>注意事项：
>
>使用go to容易造成，因此需要**尽量少用**，只有当其它方式实现不了时，再来考虑它
>
>- 代码可读性差
>- 很容易出BUG



# 二、数组

数组的模型

- 数组是一片连续的内存空间，被划分为大小相等的小空间
- 可以随机访问数组元素（在O1的时间复杂度内访问数组任意一元素）

## 2.1 关于数组的两个问题

Q1：为什么大多数语言中，数组的索引都是从0开始？

如果从1开始，每次寻址会多一次减法运算，浪费CPU资源

```c
// 寻址从0开始
i_aadr = base_addr + i * sizeof(数据类型)
// 寻址从1开始
i_aadr = base_addr + (i-1) * sizeof(数据类型)	// 多一次减法运算
```

Q2：为什么数组的效率高于链表？

- 数组的内存空间是连续的，而链表是不连续的，数组可以更好的利用CPU高速缓存，有预读效果
- 数组只需要存储数据，而链表需要存储数据+指针域，链表的内存占用更高

## 2.2 数组的声明和初始化

声明数组

```c
int arr[10];
```

>注意事项：
>
>数组的SIZE必须是是整型的常量表达式，在编译期间能计算数组的大小

数组的初始化：

```c
int arr1[10] = { 1,2,3,4,5,6,7,8,9,10 };	// 标准初始化
int arr2[10] = { 1,2,3 };	// 其余元素会初始化为0
int arr3[10] = { 0 };	// 利用特性，将所有元素初始化为0
int arr4[] = { 1,2,3,4,5 };	// 可以不指定SIZE，长度由编译器决定，这里是5
printf("%d\n", sizeof(arr4));	// 20
```
>注意事项：
>
>数组的初始化长度，不能大于数组的长度
>
>```c
>int arr5[2] = { 1,2,3 };	// error, 初始值设置项值太多
>```



## 2.3 对于数组使用sizeof计算数组的size

sizeof使用案例：

当我们需要对一个数组做循环，普通的操作是：

```c
int arr[10] = {0};
for (int i = 0; i < 10; i++) {
    .....
}
```

但如果数组不只一个呢？for循环就要写很多个，修改起来比较麻烦

```c
int arr[10] = {0};
int arr[20] = {0};
int arr[30] = {0};
for (int i = 0; i < 10; i++) {
    .....
}
for (int i = 0; i < 20; i++) {
    .....
}
for (int i = 0; i < 30; i++) {
    .....
}

// 即使使用宏定义，一个个改也较为麻烦
#define N1 10
#define N2 20
#define N3 30
```

最佳的解决方法，使用sizeof计算数组大小

```c
#define GET_ARRAY_LEN(arr, len) (len = sizeof(arr) / sizeof(arr[0])) 

int main(void) {
	int arr1[2] = { 1,2 };
	int arr2[4] = { 1,2,3,4 };
	int arr3[8] = { 1,2,3,4,5,6,7,8 };

	int len = 0;

	GET_ARRAY_LEN(arr1, len);	// 明了
	for (int i = 0; i < len; i++) {
		printf("%d ", arr1[i]);
	}
	printf("\n");

	GET_ARRAY_LEN(arr2, len);
	for (int i = 0; i < len; i++) {
		printf("%d ", arr2[i]);
	}
	printf("\n");

	GET_ARRAY_LEN(arr3, len);
	for (int i = 0; i < len; i++) {
		printf("%d ", arr3[i]);
	}
	printf("\n");
}
```

>注意事项：
>
>记录踩的坑.......
>
>数组当参数传入的时候，sizeof会计算形参的大小，而不是计算实参的大小。
>
>```c
>void printArr(int *arr) {
>	int len = 0;
>	GET_ARRAY_LEN(arr, len);
>	printf("array size = %d\n", len);	// 无论实参传入什么数组，输出的array size都是2，因为它只计算这个指针的大小
>	for (int i = 0; i < len; i++) {
>		printf("%d ", arr[i]);
>	}
>	printf("\n");
>}
>```
>
>arry size = 2的来源。
>
>```c
>int *arr = NULL;
>sizeof(arr) = 8;
>sizeof(arr[0]) = 4;
>sizeof(arr) / sizeof(arr[0]) = 2;
>```

## 2.4 多维数组

二维、三维、四维....数组，都叫做多维数组，例如：

```c
// 定义一个3行4列的数组
int matrix[3][4];
```

### 2.4.1 二维数组

二维数组在内存中的状态，以`matrix[3][4]`为例

```c
1,2,3,4		5,6,7,8		9,10,11,12
  0行		  1行		 2行  
```



**二维数组的初始化：**

```c
int matrix[3][4] = {{1,2,3,4}, {5,6,7,8}, {9,10,11,12}}; 
```

可以省略行，其余的会默认补充为0

```c
int matrix[3][4] = {{1,2,3,4}, {5,6,7,8}}; 	// 省略一行
```

也可以省略列，其余元素也会默认补充为0

```c
int matrix[3][4] = {{1,2,3}, {5,6,7}}; 	// 省略一行 + 一列
```

甚至可以省略大括号【不建议】

```c
int matrix[3][4] = {1,2,3,4, 5,6,7,8, 9,10,11,12}; 
```

全部初始化0：

```c
int matrix[3][4] = {0};
```



**可以由编译器判断行的大小：**

```c
int matrix[][4] = {{1,2,3,4}, {5,6,7,8}, {9,10,11,12}}; 
```

>注意事项：
>
>**不能省略列的大小**



### 2.4.2 常量数组

常量数组的元素不会发生改变，案例如下：

```c
const int matrix[][4] = {{1,2,3,4}, {2,2,3,4}};
matrix[1][0] = 3; // Error!元素不能修改
```



## 2.5 数组实战：随机发牌小程序

用户指定发几张牌，程序打印出手牌（52张，去除大小鬼），使用效果：

```c
请输入需要发的牌数:22
h4 h9 cT hJ sT dK s7 d9 s6 dA h8 h7 cK cA hT c8 cQ c4 s4 s5 sQ h6
```

案例代码：

```c
#define _CRT_SECURE_NO_WARNINGS

#include <stdio.h>
#include <stdbool.h>	// 用于bool
#include <stdlib.h>	// 用于srand

// 定义花色数量
#define NUM_SUITS 4

// 定义牌号数量
#define NUM_RANKS 13

int main(void) {
	// 定义牌组和花色：黑桃（Spade）、红桃（Heart）、方块（Diamond）、梅花（Club）
	const char suits[NUM_SUITS] = { 's', 'h', 'd', 'c' };
	const char ranks[NUM_RANKS] = { 'A', '2', '3', '4', '5', '6', '7', '8', '9', 'T','J', 'Q', 'K'};

	
	// 手里已有的牌组，全部初始化为false，表示都没有
	bool inhand[NUM_SUITS][NUM_RANKS] = { false };

	// 提示输入
	printf("请输入需要发的牌数:");
	int n = 0;
	scanf("%d", &n);

	// 随机数种子
	srand((unsigned)time(NULL));

	// 发牌
	while (n > 0) {
		// 产生随机花色、牌号
		int suit = rand() % NUM_SUITS; // %控制范围0-3
		int rank = rand() % NUM_RANKS; // %控制范围0-12

		// 判断手牌是否存在
		if (inhand[suit][rank] == false) {
			// 不存在，则插入手牌
			inhand[suit][rank] = true;
			n--;
			// 打印手牌
			printf("%c%c ", suits[suit], ranks[rank]);
		}
	}
	printf("\n");
}
```



