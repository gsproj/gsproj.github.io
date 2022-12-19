---
title: Day04-C高级编程
date: 2022-12-19 09:36:22
categories:
- C++
- 02_C高级编程
tags:
---

“C高级编程第4天学习笔记”



# 1 数组

什么是数组？

- 元素类型角度：数组是相同类型的变量的有序集合
- 内存角度：连续的一大片内存空间

如图所示：

![img](../../../img/clip_image002.png)

## 1.1 一维数组

数组分为一维数组和多维数组，在学习多维数组之前，需要通过一维数组来了解一些概念。

### 1.1.1 数组名

考虑下面的声明：

```c
int a;
int b[10];
```

我们把`a`称为标量`，因为它是一个单一的值，这个变量的类型是整型；

我们把`b`称为`数组`，因为它是一些值的集合，下标和数组名一起使用，比如`b[0]表示数组第一个值`

那么`b`的类型是什么？在C中，几乎所有数组名的表达式中，数组名指的是一个`指针常量`，也就是

`数组的第一个元素的地址`，它的类型取决于数组元素的类型，比如数组元素是int型，那么数组名的

类型就是`指向int的常量指针`。

### 1.1.2 下标引用

通过代码代码分析：

```c
#include <stdio.h>

int main() {
        int arr[8] = {1,2,3,4,5,6,7,8};
	    printf("num1 = %d\n", arr[3]; // 4 
        printf("num1 = %d\n", *(arr + 3)); // 4
        printf("num2 = %d\n", *(arr - 1)); // 85，越界访问
        printf("num2 = %d\n", arr[-1]); // 85，越界访问
}
```

数组下标从0开始，<font color = red>**不能为负数**</font>，否则会越界访问，输入异常数据。

 `*(arr + 3)`实际上是`arr[3]`的的另一种写法，数组名在表达式中是一个指向整型的指针，所以此表达式表示

arr指针向后移动了3个元素的长度。然后通过间接访问操作符从这个新地址开始获取这个位置的值。

### 1.1.3 指针和数组等价吗

思考：指针和数组等价吗

答案是**<font color = red>否定</font>**的，思考如下两个声明

```c
int a[10];
int *b;
```

声明一个数组时，编译器<font color=red>根据声明所指定的元素数量为数组分配内存空间</font>，然后再创建数组名，指向这段空间的起

始位置。

声明一个指针变量的时候，编译器<font color=red>只为指针本身分配内存空间</font>，并不为任何整型值分配内存空间，指针并

未初始化指向任何现有的内存空间。

因此，表达式`*a`是完全合法的，但是表达式`*b`却是非法的。`*b`将访问内存中一个不确定的位置，将会导致程序

终止。另一方面b++可以通过编译，a++却不行，因为a是一个常量值。



**数组名在表达式中使用的时候，编译器才会产生一个指针常量。那么数组在什么情况下不能作为指针常量呢？**

在以下两种场景下：

- 当数组名作为sizeof操作符的操作数的时候，此时sizeof返回的是整个数组的长度，而不是指针数组指针的

  长度。

- 当数组名作为&操作符的操作数的时候，此时返回的是一个指向数组的指针，而不是指向某个数组元素的

  指针常量。

```c
#include <stdio.h>

// 指针和数组名的差异

int main() {
        int arr[10] = {0,1,2,3,4,5,6,7,8,9};
        int *p = NULL;
        p = arr;

        // 输出的值是一样的
        for (int i = 0; i < 10; i++) {
                printf("%d %d, ", arr[i], p[i]); 
        }

        // sizeof显示占用的内存大小不一样：40 8
        printf("\narr占的内存大小是: %d\np占的内存大小是: %d\n", sizeof(arr), sizeof(p)); 
    	
    	// int(*)[10]而不是int* (在gcc没有编译运行成功)
        printf("&arr type is %s\n", typeid(&arr).name()); 
}
```

### 1.1.4 数组名做函数参数

数组名做函数参数两种写法都可以

```c
int print_arr(int *arr);
int print_arr(int arr[]);
```

用那种更准确呢？**指针**，实参实际上是指针，而不是数组，同样`sizeof(arr)`是指针的长度，而不是数组的长度

现在我们清楚了，为什么一维数组名做形参中无须写明它的元素数目了`int arr[]`，因为形参只是一个指针，并

不需要为数组参数分配内存。

另一方面，这种方式使得函数无法知道数组的长度。如果函数需要知道数组的长度，它必须显式传递一个长度参数

给函数。

案例如下：

```c
#include <stdio.h>

void print_arr01(int *arr) {
        for (int i = 0; i < 8; i++) {
                printf("%d ", arr[i]);
        } // 1 2 3 4 5 6 7 8
        printf("\nsizeof = %d\n", sizeof(arr)); // 8	内存大小一样
}

void print_arr02(int arr[]) {
        for (int i = 0; i < 8; i++) {
                printf("%d ", arr[i]);
        } // 1 2 3 4 5 6 7 8
        printf("\nsizeof = %d\n", sizeof(arr)); // 8	内存大小一样
}

int main() {
        int arr[8] = {1,2,3,4,5,6,7,8};
        print_arr01(arr);
        print_arr02(arr);
}
```

## 1.2 多维数组

什么是多维数组？某个数组的维数不止1个，它就被称为多维数组。

以下是二维数组的案例：

```c
#include <stdio.h>

int main() {
        // 二维数组的三种初始化方式
        int arr1[2][3] = {
                {1,2,3},
                {4,5,6}
        };

        int arr2[][4] = {
                {1,2,3,4},
                {5,6,7,8}
        };

        int arr3[3][3] = {1,2,3,4,5,6,7,8,9};

        for (int i = 0; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                        printf("%d, ", arr3[i][j]);
                }
        }
        printf("\n");
}
```

### 1.2.1 多维数组的数组名

在一维数组中，数组名是“指向元素类型的指针”，指向数组的第一个元素，同理如

在二维数组中，数组名是“指向元素类型数组的指针”，指向数组的第一个元素，不过第一个元素是数组

例如：

```c
int arr[3][3] = {{1，2，3},{4，5，6},{7，8，9}};
```

图解为：

![image-20221219135726107](../../../img/image-20221219135726107.png)

可以理解为这是一个一维数组，包含了3个元素，只是每个元素恰好是包含了10个元素的数组。arr就表示指向它的

第1个元素的指针，所以arr是一个指向了包含了10个整型元素的数组的指针。

### 1.2.2 二维数组的线性存储特性

以下案例可以证明二维数组是线性存储的：

```c
#include <stdio.h>

void print_arr(int *arr, int len) {
        for (int i = 0 ; i < len; i++) {
                printf("%d ", arr[i]);
        }
        printf("\n");
}

void test01() {
        int arr1[][3] = {
                {1,2,3},
                {4,5,6},
                {7,8,9}
        };

        int arr2[][3] = {1,2,3,4,5,6,7,8,9};

        int len = sizeof(arr1) / sizeof(int);

        // arr1和arr2输出一致，证明是线性存储
    	// 通过将数组首地址指针转成int*类型，那么步长就变成了4，就可以遍历整个数组
        print_arr((int *)arr1, len); // 1 2 3 4 5 6 7 8 9
        print_arr((int *)arr2, len); // 1 2 3 4 5 6 7 8 9
}

int main() {
        test01();
        return 0;
}
```

### 1.2.3 二维数组的三种参数形式

```c
#include <stdio.h>

// 二维数组方式01
void printArr01(int arr[3][3]) {
        for (int i = 0 ; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                        printf("%d ", arr[i][j]);
                }
                printf("\n");
        }
}

// 二维数组方式02
void printArr02(int arr[][3]) {
        for (int i = 0 ; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                        printf("%d ", arr[i][j]);
                }
                printf("\n");
        }
}

// 数组指针方式
void printArr03(int(*arr)[3]) {
        for (int i = 0 ; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                        printf("%d ", arr[i][j]);
                }
                printf("\n");
        }
}
int main() {
        int arr[][3] = {
                {7,8,9},
                {22,33,44},
                {55,66,88}
        };

        printArr01(arr);
        printArr02(arr);
        printArr03(arr);
}
```

## 1.3 数组指针和指针数组

让人头大的两个知识点，非常容易混淆。

```c
// 数组指针
int arr[3] = {1,2,3};
int (*pArr)[3] = &arr;

// 指针数组
int a = 10;
int b = 20;
int c = 30;

int *pABC[3] = {&a, &b, &c}; 
```

### 1.3.1 数组指针

数组指针，即**指向数组的指针**，以下是它的定义方式和使用方法：

```c
#include <stdio.h>


// typedef定义数组指针
void test01() {
        int arr[3] = {0,1,2};
        typedef int(ArrayType)[3]; // 定义的方式比较独特

        ArrayType myarr; // 等价于 int(myarr)[3]
        ArrayType *pArr = &arr; // 等价于int(*pAarr)[3] = &arr;
        for(int i = 0; i < 3; i++) {
                printf("%d ", (*pArr)[i]); // 解引用的方式比较独特
        }
        printf("\n");

}

// 使用数组指针给数组赋值
void test02() {
        int arr[8];
        typedef int(ArrayType)[8];

        // 定义数组指针
        ArrayType *pArr = &arr;

        // 使用指针给数组赋值
        for (int i = 0; i < 8; i++) {
                (*pArr)[i] = i + 100;
        }

        // 输出
        for (int i = 0; i < 8; i++) {
                printf("%d ", (*pArr)[i]);
        }
        printf("\n");

}

// 不使用typedef直接定义
void test03() {
        int arr[3] = {4,5,6};
        int(*pArr)[3] = &arr;

        // 输出
        for (int i = 0; i < 3; i++) {
                printf("%d ", (*pArr)[i]);
        }
        printf("\n");
}

int main() {
        test01();
        test02();
        test03();
}
```

### 1.3.2 指针数组

指针数组：一个数组，里面的元素都是指针，容易和数组指针搞混，一定要分得清才行！

以下是指针数组的案例：

```c
#include <stdio.h>

void test01() {
        int a = 10;
        int b = 20;
        int c = 30;
		
    	// 定义指针数组
        int *arr[3] = {&a, &b, &c}; // 最大的不同在这，如果是int (*arr)[3]就是"数组指针"

        // 输出
        for (int i = 0; i < 3; i++) {
                printf("%d ", *(arr[i]));
        }
        printf("\n"); // 10 20 30
}

int main() {
        test01();
        return 0;
}
```

#### 1.3.2.1 栈区指针数组

案例：使用指针数组对数组进行排序（栈区版）

```c
#include <stdio.h>
#include <string.h>

// 数组排序（栈区版）

void arr_sort(char **arr, int len) {
        for (int i = 0; i < len - 1; i++) {
                for (int j = 0; j < len - i - 1; j++) {
                        // 冒泡排序比较两个字符串
                        if(strcmp(arr[j], arr[j+1]) > 0) {
                                char *temp = arr[j];
                                arr[j] = arr[j+1];
                                arr[j+1] = temp;
                        }
                }
        }
}

void arr_print(char **p, int len) {
        for (int i = 0; i < len; i++) {
                printf("%s ", p[i]);
        }
        printf("\n");
}

int main() {
        // 栈区，主调函数分配内存

        // 定义指针数组 (数组元素都是char*类型)
        char* p[] = {"bb", "aa", "dd", "ee", "cc"};

        // 获取数组长度
        int len = sizeof(p) / sizeof(char *);

        // 打印数组
        arr_print(p, len); // bb aa dd ee cc

        // 排序
        arr_sort(p, len); 

        // 打印数组
        arr_print(p, len); // aa bb cc dd ee

        return 0;
}
```

#### 1.3.2.2 堆区指针数组

案例：使用指针数组对数组进行排序（堆区版）

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 分配内存，赋值
char** alloc_arr(int len) {
        if (len < 0) {
                return NULL;
        }

        char** temp = (char**)malloc(sizeof(char*) * len);
        // 容错验证，一定是！！！双等号！！！
    	if (temp == NULL) {	
                printf("指针分配内存失败");
                return NULL;
        }

        // 分别给每个指针分配内存
        for (int i = 0; i < len; i++) {
                temp[i] = malloc(sizeof(char)* 30);
                sprintf(temp[i], "%2d_hello world", i+1);
        }

        return temp;
}

// 打印输出
void print_arr(char **p, int len) {
        if (p == NULL) {
                printf("指针中没数据\n");
                return;
        }
        for (int i = 0; i < len; i++) {
                printf("%s\n", p[i]);
        }
}

// 释放空间
void free_arr(char **p, int len) {
        if (p == NULL) {
                return;
        }

        // 释放指针数组中的元素
        for(int i= 0 ; i < len; i++) {
                free(p[i]);
                p[i] = NULL;
        }

        // 释放数组本身
        free(p);
        p = NULL;
}

int main() {
        // 堆区：被调函数分配内存
        int len = 10;
        char **arr = alloc_arr(len);

        print_arr(arr, len);

        free_arr(arr, len);
}
```

### 1.4 数组总结

编程提示：

- 源代码的可读性几乎总是比程序的运行时效率更为重要
- 只要有可能，函数的指针形参都应该声明为const
- 在多维数组的初始值列表中使用完整的多层花括号提高可读性

内容总结：

- 在绝大多数表达式中，数组名的值是指向数组第1个元素的指针。这个规则只有两个例外，sizeof和对数组名&。
- 指针和数组并不相等。当我们声明一个数组的时候，同时也分配了内存。但是声明指针的时候，只分配容纳指针本身的空间。
- 当数组名作为函数参数时，实际传递给函数的是一个指向数组第1个元素的指针。
- 我们不单可以创建指向普通变量的指针，也可创建指向数组的指针。



### 