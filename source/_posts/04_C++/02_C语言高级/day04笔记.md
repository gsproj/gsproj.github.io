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



# 2 结构体

## 2.1 结构体基础知识

通过案例来了解结构体的基础知识

### 2.1.1 结构体创建和使用

结构体类型的使用分为三步：

1. 定义结构体类型
2. 定义结构体变量
3. 使用结构体变量

可以用`struct`关键字来定义结构体，为方便使用也可以用`typdef`给结构体类型`起别名`

以下是三步的案例：

```c
#include <stdio.h>

//	1.定义结构体类型 
// 定义结构体类型：方法一
struct Person {
        char name[20];
        int age;
};


// 定义结构体类型：方法二
typedef struct _PERSON {
        char name[20];
        int age;
}Person2;

// 2.使用结构体类型创建变量 
void test01() {
       	// 3、使用方法一创建的结构体:创建结构体变量，并赋值初始化、输出
        struct Person p1 = {"小阳人", 18};
        printf("p1的名字是:%s,年龄：%d\n", p1.name, p1.age);
        // p1的名字是:小阳人,年龄：18
}

// 使用方法二创建的结构体:创建结构体变量
void test02() {
    	// 3、使用方法一创建的结构体:创建结构体变量，并赋值初始化、输出
        Person2 p2 = {"喜洋洋", 9}; // 明显更简化，不用再写"struct"关键字
        printf("p2的名字是:%s,年龄：%d\n", p2.name, p2.age);
        // p2的名字是:喜洋洋,年龄：9
}

int main() {
        test01();
        test02();
        return 0;
}
```

**注意：**

定义结构体类型时不要直接给成员赋值，结构体只是一个类型，编译器还没有为其分配空间，只有根据其

类型定义变量时，才分配空间，有空间后才能赋值。

### 2.1.2 结构体数组

结构体数组，顾名思义是一个数组中包含多个结构体，在初始化的时候可以选择在`栈上`或者`堆上`分配内存，使用案例如下：

**版本一：栈上分配内存**

```c
#include <stdio.h>

// 创建结构体类型
struct Person {
        char name[20];
        int age;
};

void test01() {
        // 创建结构体数组，初始化值（在栈上分配空间）
        struct Person mans[4] = {
                {"小阳人01", 18},
                {"小阳人02", 19},
                {"小阳人03", 20},
                {"小阳人04", 21},
        };

        // 输出内容
        for (int i = 0; i < 4; i++) {
                printf("name = %s, age = %d\n", mans[i].name, mans[i].age);
        }
    
    	/*
    		输出：
    		name = 小阳人01, age = 18
            name = 小阳人02, age = 19
            name = 小阳人03, age = 20
            name = 小阳人04, age = 21
    	*/

}

int main() {
        test01();
        return 0;
}
```

**版本二：堆上分配内存**

```c
#include <stdio.h>
#include <stdlib.h>

// 创建结构体类型
struct Person {
        char name[20];
        int age;
};

void test02() {
        // 创建结构体数组，初始化值(堆上分配内存)
        struct Person *mans = (struct Person*)malloc(sizeof(struct Person) * 4);
        for (int i = 0; i < 4; i++) {
                sprintf(mans[i].name, "小阳人_%d", i);
                mans[i].age = i + 18;
        }

        // 输出内容
        for (int i = 0; i < 4; i++) {
                printf("name = %s, age = %d\n", mans[i].name, mans[i].age);
        }

        // 释放内存
        free(mans);
        mans = NULL;

}

int main() {
        test02();
        return 0;
}
```

### 2.1.3 深拷贝和浅拷贝

深浅拷贝的概念：

- 深拷贝：将对象及值复制过来，两个对象修改其中任意的值另一个值不会改变
- 浅拷贝：只是复制了对象的引用地址，两个对象指向同一个内存地址，所以修改其中任意的值，另一个值都会随之变化

通过案例来分析：

浅拷贝的案例

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct __TEACHER {
        char *name;
}Teacher;

void test01() {
        Teacher t1;
        Teacher t2;
        t1.name = (char*)malloc(sizeof(char) * 20);
        strcpy(t1.name, "hi");
    	// 浅拷贝
        t2.name = t1.name;

        printf("t1.name = %s, t2.name = %s\n", t1.name, t2.name);
        *(t1.name) = 'i';  // 修改t1，同 t1.name[0] = 'i';
        printf("t1.name = %s, t2.name = %s\n", t1.name, t2.name);
    
    	/* 
            输出:
            t1.name = hi, t2.name = hi
            t1.name = ii, t2.name = ii // 修改t1影响到t2
		*/
}

int main() {
        test01();
        return 0;
}
```

深拷贝案例：

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct __TEACHER {
        char *name;
}Teacher;

void test02() {
        Teacher t1;
        Teacher t2;
        t1.name = (char*)malloc(sizeof(char) * 20);
    	
    	// 改成手动分配内存拷贝（深拷贝）
        t2.name = (char*)malloc(sizeof(char) * 20);
        strcpy(t1.name, "hi");
        strcpy(t2.name, t1.name);

        printf("t1.name = %s, t2.name = %s\n", t1.name, t2.name);
        *(t1.name) = 'i'; // 修改t1，同 t1.name[0] = 'i';
        printf("t1.name = %s, t2.name = %s\n", t1.name, t2.name);
    
    	/* 
            输出:
            t1.name = hi, t2.name = hi
            t1.name = ii, t2.name = hi // 修改t1不影响t2
		*/
}

int main() {
        test02();
        return 0;
}
```

## 2.2 结构体嵌套指针

### 2.2.1 结构体嵌套一级指针

案例如下

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 创建结构体
struct Person {
        char *name;
        int age;
};

// 分配内存空间（使用结构体指针的地址--二级指针）
void allocate_mem(struct Person ** person) {
        if (person == NULL) {
                return;
        }

        struct Person * temp = (struct Person *)malloc(sizeof(struct Person));
        if (temp == NULL) {
                return;
        }

        temp->name = (char *)malloc(sizeof(char) * 50);
        strcpy(temp->name, "testname");
        temp->age = 100;

        *person = temp;
}

// 输出内容
void print_person(struct Person * person) {
        printf("name: %s, age: %d\n", person->name, person->age);
}

// 释放空间
void free_person(struct Person **person) {
        if (person == NULL) {
                return;
        }

        struct Person * temp = *person;
        if (temp->name != NULL) {
                free(temp->name);
                temp->name = NULL;
        }
        free(temp);
}

int main() {
        struct Person * p = NULL;
        allocate_mem(&p);
        print_person(p);
        free_person(&p);
        return 0;
}

```

### 2.2.2 结构体嵌套二级指针

案例如下

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

/*
 *      案例：创建多个教师，每个教师有多个学生
 */

// 创建结构体
typedef struct __Teacher {
        char name[64];
        char **students;
}Teacher;

// 分配内存，添加内容
void alloc_teacher(Teacher** teacher,int n,int m){

        if (teacher == NULL) {
                return;
        }

        // 创建老师数组
        Teacher* teachers = (Teacher*)malloc(sizeof(Teacher)* n);
        if (teachers == NULL) { // =号一个都不能少
                return;
        }

    	// 学生数组分配内存
        int num = 0;
        for (int i = 0; i < n; i++) {
                sprintf(teachers[i].name, "老师_%d", i+1);
                teachers[i].students = (char **)malloc(sizeof(char*) * m);
				// 数组内的学生分配内存
                for (int j = 0; j < m; j++) {
                        teachers[i].students[j] = (char *)malloc(sizeof(char *) * 50);
                        sprintf(teachers[i].students[j], "学生_%d", num + 1);
                        num++;
                }
        }

        *teacher = teachers;
}

// 输出
void print_teacher(Teacher *teachers, int n, int m) {
        for (int i = 0 ; i < n; i++) {
                printf("教师姓名: %s\n", teachers[i].name);
                for (int j = 0; j < m; j++) {
                        printf("学生姓名：%s\n", teachers[i].students[j]);
                }
                printf("----------------\n");
        }
}

// 释放内存
void free_teacher(Teacher **t, int n, int m) {
        if (t == NULL) {
                return;
        }

        Teacher *temp = *t;
        for (int i = 0; i < n; i++) {
                for (int j = 0; j < m; j++) {
                        free(temp[i].students[j]);
                        temp[i].students[j] = NULL;
                }
                free(temp[i].students);
                temp[i].students = NULL;
        }
        free(temp);
}

int main() {
        Teacher *tea = NULL;
        alloc_teacher(&tea, 3, 5);
        print_teacher(tea, 3, 5);
        free_teacher(&tea, 3, 5);
        return 0;
}

// 输出
教师姓名: 老师_1
学生姓名：学生_1
学生姓名：学生_2
学生姓名：学生_3
学生姓名：学生_4
学生姓名：学生_5
----------------
教师姓名: 老师_2
学生姓名：学生_6
学生姓名：学生_7
学生姓名：学生_8
学生姓名：学生_9
学生姓名：学生_10
----------------
教师姓名: 老师_3
学生姓名：学生_11
学生姓名：学生_12
学生姓名：学生_13
学生姓名：学生_14
学生姓名：学生_15
----------------
```

## 2.3 结构体成员偏移量

一旦结构体定义下来，结构体中的成员内存布局就定下了

案例如下：

```c
#include <stdio.h>
#include <stddef.h>

// 定义结构体
struct Teacher {
        char a;
        int b;
};

void test01() {
        struct Teacher t1; // 创建结构体变量t1
        struct Teacher *p = &t1; // 创建t1的指针p


        int offsize1 = (int)&(p->b) - (int)p; // 成员b相对于结构体Teacher的偏移量
        int offsize2  = offsetof(struct Teacher, b); // 效果同上

        printf("offsize1 = %d\n", offsize1); // 4
        printf("offsize2 = %d\n", offsize2); // 4
        // 虽然并没有给结构体变量赋值，但是结构体中的成员已经占用内存了
}

int main() {
        test01();
        return 0;
}
```

## 2.4 结构体内存字节对齐

可以参考[文章](https://mp.weixin.qq.com/s?src=11&timestamp=1673576126&ver=4285&signature=CYU7t5D8sKY287hkU*JP-MNiPxj2APtPFmFENpvkdFopHPu2bqr16vR5Rs-mnM*ESHGGA9okBcO6g1VvGLrF37BRAX0fTl-mCNEAFtDu6FOpf2yDJ0N9mF2lbAyJPKF*&new=1)

### 2.4.1 什么是内存对齐？

当使用sizeof计算结构体所占空间时，并不是简单的将结构体内每个元素所占的内存空间相加，这里涉及到字节对齐的问题，比如

```c
#include <stdio.h>

struct Person {
        char name[20]; // 占20字节
        int age;        // 占4字节
        int id;         // 占4字节
        char class;     // 占1字节
};

int main() {
        printf("sizeof = %d\n", sizeof(struct Person)); // sizeof = 32, 为什么不是29?
        return 0;
}
```

从理论上讲，对于任何变量的访问都可以从任何地址开始访问，但是事实上不是如此，实际上访问特定类型的变量

只能在特定的地址访问，这就需要各个变量在空间上按一定的规则排列， 而不是简单地顺序排列，这就是内存对

齐。

### 2.4.2 内存对齐的原因

我们知道内存的最小单元是一个字节

理论上：当cpu从内存读取数据的时候，应该是是一个一个字节读取的

实际上：cpu将内存当成多个块，每次从内存中读取一个块，这个块的大小可能是2、4、8、16等

**为什么使用内存对齐？**

是操作系统提高内存访问的策略，操作系统在访问内存的时候，每次读取一定长度，只要读一次（这次读4字节，

下次还是4字节...方便），而如果没有对齐，为了访问一个变量可能产生二次访问（这次读4字节，下次读1字节，

再下次读3字节，我乱了）

**内存对齐的好处**

- 提高存取数据的速度
- 某些平台智能在特定的地址处访问特定类型的数据，否则会抛出硬件异常给操作系统

### 2.4.3 如何内存对齐

内存对齐是如何对齐的？

- 对于标准数据类型，它的地址只要是它的长度的整数倍

- 对于非标准数据类型（比如结构体），要遵循对齐原则

  - 数组成员对齐规则。第一个数组成员应该放在offset为0的地方，以后每个数组成员应该放在offset为

    **min**（当前成员的大小，#pargama pack(n)）**整数倍的地方开始（比如int在32位机器为４字节，#pargama pack(2)，那么从2的倍数地方开始存储）。

  - 结构体总的大小，也就是sizeof的结果，必须是**min（结构体内部最大成员，#pargama pack(n)）**的整数倍，不足要补齐。

  - 结构体做为成员的对齐规则。如果一个结构体B里嵌套另一个结构体A,还是以最大成员类型的大小对齐，但是结构体A的起点为A内部最大成员的整数倍的地方。（struct B里存有struct A，A里有char，int，double等成员，那A应该从8的整数倍开始存储。），结构体A中的成员的对齐规则仍满足原则1、原则2。

### 2.4.5 手动这是对齐模数

```c
#pragma pack(show)
//显示当前packing alignment的字节数，以warning message的形式被显示。

#pragma pack(push) 
// 将当前指定的packing alignment数组进行压栈操作，这里的栈是the internal compiler stack,同时设置当前的packing alignment为n；如果n没有指定，则将当前的packing alignment数组压栈。

#pragma pack(pop) 
// 从internal compiler stack中删除最顶端的reaord; 如果没有指定n,则当前栈顶record即为新的packing alignement数值；如果指定了n，则n成为新的packing alignment值

#pragma pack(n)
// 指定packing的数值，以字节为单位，缺省数值是8，合法的数值分别是1,2,4,8,16。 
```

### 2.4.6 内存对齐的案例

```c
#include <stdio.h>

struct Person01 {
        int a; // 4
        char b; // 1
        double c; // 8
        float d; // 4
}; // 内部对齐之后是20，整体对齐，整体为最大类型（8）的整数倍，24

struct Person02 {
        char a; // 1
        struct Person01 b; // 24
        double c; // 8
}; // 内部对齐40, 整体对齐为8的被数，正好也是40

int main() {
        printf("sizeof struct Person01 = %d\n", sizeof(struct Person01)); // 24
        printf("sizeof struct Person02 = %d\n", sizeof(struct Person02)); // 40
        return 0;
}
```

## 2.5 结构体自身引用

问题1：请问结构体可以嵌套本类型的结构体变量吗？

问题2：请问结构体可以嵌套本类型的结构体指针变量吗？

通过实验来验证：

```c
typedef struct _STUDENT{
	char name[64];
	int age;
}Student;

typedef struct _TEACHER{
	char name[64];
	Student stu; //结构体可以嵌套其他类型的结构体
	//Teacher stu;
	//struct _TEACHER teacher; //此时Teacher类型的成员还没有确定，编译器无法分配内存
	struct _TEACHER* teacher; //不论什么类型的指针，都只占4个字节，编译器可确定内存分配
}Teacher;
```

得出结论：

- 结构体可以嵌套另外一个结构体的任何类型变量;
- 结构体嵌套本结构体普通变量（不可以）。本结构体的类型大小无法确定，类型本质：固定大小内存块别名;
- 结构体嵌套本结构体指针变量（可以）, 指针变量的空间能确定，32位， 4字节， 64位， 8字节;

