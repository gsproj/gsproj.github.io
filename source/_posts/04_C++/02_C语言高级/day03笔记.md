---
title: Day03-C高级编程
date: 2022-12-14 13:36:22
categories:
- C++
- 02_C高级编程
tags:
---

“C高级编程第3天学习笔记”

# 1 calloc和realloc

用于内存分配的两个API方法

## 1.1 calloc使用

calloc的功能和使用方法：

```c
#include <stdlib.h>
void *calloc(size_t nmemb, size_t size);
功能：
	在内存动态存储区中分配nmemb块长度为size字节的连续区域。calloc自动将分配的内存 置0。
参数：
	nmemb：所需内存单元数量
	size：每个内存单元的大小（单位：字节）
返回值：
	成功：分配空间的起始地址
	失败：NULL
```

calloc案例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
        char *buf = (char *)calloc(1, 200); // 分配200个1字节的内存空间
        strcpy(buf, "Hello Size");
        printf("buf = %s, sizeof = %d\n", buf, sizeof(buf));
        // 输出: buf = Hello Size, sizeof = 8
	
    	free(buf);
    	buf = NULL;

        return 0;
}
```

## 1.2 realloc使用

realloc的功能和使用方法：

```c
#include <stdlib.h>
void *realloc(void *ptr, size_t size);
功能：
	重新分配用malloc或者calloc函数在堆中分配内存空间的大小。
	realloc不会自动清理增加的内存，需要手动清理，如果指定的地址后面有连续的空间，那么就会在已有地址基础上增加	内存，如果指定的地址后面没有空间，那么realloc会重新分配新的连续内存，把旧内存的值拷贝到新内存，同时释放旧	内存。
参数：
	ptr：为之前用malloc或者calloc分配的内存地址，如果此参数等于NULL，那么和realloc与malloc功能一致
	size：为重新分配内存的大小, 单位：字节
返回值：
	成功：新分配的堆内存地址
	失败：NULL
```

realloc案例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
        // 用malloc分配10字节的空间
        char *buf = (char *)malloc(10);
        strcpy(buf, "012345678966666");
        printf("buf = %s\n", buf); // 输出: buf = 012345678966666 为什么超了也能显示？？
        free(buf);
        buf = NULL;

        // 用calloc分配10字节的内存空间
        buf = (char *)calloc(1, 10); // 分配10个1字节的内存空间
        strcpy(buf, "012345678988888");
        printf("buf = %s\n", buf); // 输出: buf = 012345678988888

        // 重新调整内存大小
        realloc(buf, 300);
        printf("buf = %s\n", buf); // 输出: buf = 012345678988888
        strcpy(buf, "Hello Size 22222");
        printf("buf = %s\n", buf); // 输出：Hello Size 22222

        return 0;
}
```

# 2 二级指针

## 2.1 二级指针的概念

二级指针，通俗的讲就是指针的指针，以下面的例子来说

```c
int a = 12;
int *b = &a;	// 一级指针
int **c = &p;	// 二级指针，存储一级指针的地址
```

在内存中的表示如下图：

![image-20221214134854067](../../../img/image-20221214134854067.png)

## 2.2 二级指针做形参的输出特性

特性：由被调函数分配内存

```c
#include <stdio.h>
#include <stdlib.h>

// 分配内存，赋值, 从函数中输出二级指针
void allocate_space(int **arr, int n) {
        // 分配内存
        int *tmp = (int *)malloc(sizeof(int) * n);
        if (NULL == tmp) {
                return;
        }

        // 内存初始化值
        int *pTmp = tmp;
        for (int i = 0; i < n; i++) {
                // pTmp[i] = i + 100;
                *pTmp = i + 100;
                pTmp++;
        }

        // 指针间接赋值
        *arr = tmp;
}

// 输出值
void printarr(int *arr, int n) {
        for (int i = 0; i < n; i++) {
                printf("%d ", arr[i]);
        }
        printf("\n");
}

int main() {
        int *myarr = NULL;
        int n = 8;
        allocate_space(&myarr, n);
        printarr(myarr, n);
        return 0;
}
```

## 2.3  二级指针做形参的输入特性

输入特性：由主调函数分配内存

```c
#include <stdio.h>
#include <stdlib.h>

// 输入二级指针到函数中
void print_array(int **arr,int n){
        for (int i = 0; i < n;i ++){
                printf("%d ",*(arr[i]));
        }
        printf("\n");
}

//二级指针输入特性(由主调函数分配内存)
void test(){

        int a1 = 10;
        int a2 = 20;
        int a3 = 30;
        int a4 = 40;
        int a5 = 50;

        int n = 5;

        int** arr = (int **)malloc(sizeof(int *) * n);
        arr[0] = &a1;
        arr[1] = &a2;
        arr[2] = &a3;
        arr[3] = &a4;
        arr[4] = &a5;

        print_array(arr,n);

        free(arr);
        arr = NULL;
}

// 赋值
int main() {
        /*
        TODO：我在linux中尝试用下面的代码赋值，结果输出（段错误），但是test01()赋值是正常的
        int n = 8;
        int **arr = (int**)malloc(sizeof(int*) * n);

        // 分配内存
        int *tmp = (int *)malloc(sizeof(int) * n);
        if (NULL == tmp) {
                return 0;
        }

        // 内存初始化值
        int *pTmp = tmp;
        for (int i = 0; i < n; i++) {
                // pTmp[i] = i + 100;
                *pTmp = i + 100;
                pTmp++;
        }

        // 指针间接赋值
        *arr = tmp;

        print_array(arr, n);
        */ 
        test();
        return 0;
}
```

## 2.4  多级指针

不仅有二级指针，也有三级、四级等多级指针，以下是三级指针案例：

```c
#include <stdio.h>
#include <stdlib.h>

// 分配空间，赋值
void allocSpace(char ***arr, int n) {
        char **tmp = (char **)malloc(sizeof(char *) * n);

        if (tmp == NULL) {
                return;
        }

        // 给每个指针malloc内存
        for (int i = 0; i < n; i++) {
                tmp[i] = (char *)malloc(sizeof(char) * n); // 分配内存
                sprintf(tmp[i], "%2d_hello world!", i + 1); // 赋值
        }

        // 间接赋值
        *arr = tmp;
}

// 打印值
void printArray(char **arr, int n) {
        for (int i = 0; i < n; i++) {
                printf("%s\n", arr[i]);
        }
}

// 释放空间
void freeSpace(char ***buf, int n) {
        if (buf == NULL) {
                return;
        }

        char **tmp = *buf;

        for (int i = 0 ; i < n; i++) {
                free(tmp[i]);
                tmp[i] = NULL;
        }

        free(tmp);
}

int main() {
        int n = 10;
        char **arr = NULL;
        allocSpace(&arr, n);
        printArray(arr, n);
        freeSpace(&arr, n);

        return 0;
}
```

输出

```c
 1_hello world!
 2_hello world!
 3_hello world!
 4_hello world!
 5_hello world!
 6_hello world!
 7_hello world!
 8_hello world!
 9_hello world!
10_hello world!
```

# 3 位运算

## 3.1 位逻辑运算符

位逻辑运算符总共有4个，分别是：

- 按位取反`~`
- 按位与`&`
- 按位或`|`|
- 按位异或``

不要与逻辑判断的`&&`和`||`弄混淆。

### 3.1.1 按位取反`~`

一元运算符`~`能将每个1变成0，每个0变成1，如下面的例子:

```c
#include <stdio.h>

int main() {
        // 一个字节8位
        unsigned char a = 2; // 二进制: 00000010
        unsigned char b = ~a; // 取反结果: 11111101
        printf("b = %d\n", b); // 253
    	printf("a = %d\n", b); // 2  不影响原变量
}
```

### 3.1.2 按位与`&`

两个都为1，结果才为1，否则为0

```c
#include <stdio.h>

int main() {
        // 一个字节8位
        unsigned char a = 2; // 二进制: 00000010
        unsigned char b = 3; // 二进制: 00000011
        unsigned char c = a & b; // 与后: 00000010
        printf("c = %d\n", c); // 2
}
```

按位与符号组合运算

```c
#include <stdio.h>

int main() {
        // 一个字节8位
        unsigned char a = 2; // 二进制: 00000010
        unsigned char b = 3; // 二进制: 00000011
        a &= b;  // 等价于 a = a & b;
        printf("a = %d\n", a); // 2
}
```

### 3.1.3 按位或`|`

有一个为1就是1，只有两个0才是0

```c
#include <stdio.h>

int main() {
        // 一个字节8位
        unsigned char a = 2; // 二进制: 00000010
        unsigned char b = 3; // 二进制: 00000011
        a |= b;  // a = 00000011
        printf("a = %d\n", a); // 3
}
```

### 3.1.4 按位异或`^`

不同为1，相同为0

```c
#include <stdio.h>

int main() {
        // 一个字节8位
        unsigned char a = 2; // 二进制: 00000010
        unsigned char b = 3; // 二进制: 00000011
        a ^= b;  // a = 00000001
        printf("a = %d\n", a); // 1
}
```

### 3.1.5 位运算符的使用场景

学习了位运算符的使用方法，那么这四个位运算符能够用来做什么呢？主要有四个用途：

- 打开位
- 关闭位
- 转置位
- 交换两个数不需要中间值（面试题）

#### 3.1.5.1 打开位

案例：将标记`11111001`的第2位打开

```c
#include <stdio.h>

int main() {
        // 11111001 需要打开成 11111101
        unsigned char a = 249; // 11111001
        unsigned char b = 4; // 00000100
        a |= b; // 11111101 = 253(十进制)
        printf("a = %d\n", a);

        return 0;
}
```

特殊：将所有位打开`flag | ~flag`

```c
#include <stdio.h>

int main() {
        unsigned char a = (char)(00001000);
        a |= ~a;
        printf("a = %d\n", a); // 255

        return 0;
}
```

#### 3.1.5.2 关闭位

将所有位关闭`flag & ~flag`

```c
(10011010)
&(01100101)
=(00000000)
```

#### 3.1.5.3 转置位

转置(toggling)一个位表示如果该位打开，则关闭该位；如果该位关闭，则打开`flag ^ 0xff`

```c
(10010011)
^(11111111)
=(01101100)
```

#### 3.1.5.4 交换两个数不需要临时变量（重要）

数值交换最方便的方法

```c
#include <stdio.h>

int main() {
        int num1 = 10;
        int num2 = 20;
        num1 ^= num2;
        num2 ^= num1;
        num1 ^= num2;

        printf("num1 = %d, num2 = %d\n", num1, num2); // num1 = 20, num2 = 10
        return 0;
}

```

## 3.2 移位运算符

以为运算符可以将位向左或向右移动，我们仍然以二进制的形式来说明该机制的工作原理。

### 3.2.1 左移`<<`

左移运算符将每位向左移动，移动的位数由`操作数`指定，空出来的位用0填充，并丢弃移出左侧操作数末端的位。

比如：

```c
#include <stdio.h>

int main() {
        unsigned char a = 138; // 10001010
        printf("a = %d\n", a);
        a  = a << 2; // 00101000 即十进制的40，最左侧的1被丢弃
        printf("a = %d\n", a); // 40

        return 0;
}
```

### 3.2.2 右移`>>`

右移运算符>>将其左侧的操作数的值每位向右移动，移动的位数由其右侧的操作数指定。丢弃移出左侧操作数有段的位。**<font color=green>对于unsigned类型，使用0填充左端空出的位。对于有符号类型，结果依赖于机器。空出的位可能用0填充，或者使用符号(最左端)位的副本填充</font>**

将3.3.1的案例改成右移看看：

```c
#include <stdio.h>

int main() {
        unsigned char a = 138; // 10001010
        printf("a = %d\n", a);
        a  = a >> 2; // 00100010 即十进制的34
        printf("a = %d\n", a); // 34

        return 0;
}
```

对于`绿色字`部分的解释

```c
//有符号值
(10001010) >> 2
(00100010)     //在某些系统上的结果值

(10001010) >> 2
(11100010)     //在另一些系统上的结果

//无符号值
(10001010) >> 2
(00100010)    //所有系统上的结果值
```

### 3.2.3 移位运算符的使用场景

移位运算符能提供快捷、高效对2的幂的乘法和除法

| 使用方法     | 作用                                   |
| ------------ | -------------------------------------- |
| number  >> n | 如果number非负，则用number除以2的n次幂 |
| number  << n | number乘以2的n次幂                     |

