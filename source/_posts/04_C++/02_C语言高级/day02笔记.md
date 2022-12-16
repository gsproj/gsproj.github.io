---
title: Day02-C高级编程
date: 2022-12-08 13:36:22
categories:
- C++
- 02_C高级编程
tags:
---

“C高级编程第2天学习笔记”

# 1 函数

## 1.1 宏函数

实际上是“宏替换”，定义的代码会展开

案例如下：

```c
#include <stdio.h>

#define MYADD(x, y) ((x) + (y)) // 不加括号会怎么样？

int main() {
        printf("%d\n", MYADD(2, 3) * 10);
}

```

宏函数在运行中会展开，即：

```c
printf("%d\n", MYADD(2,3));

// 会展开成
printf("%d\n", ((2) + (3) * 10); // 结果：50
```

如果定义的时候不加括号

```c
// 如下定义
#define MYADD(x, y) x + y

// 则会展开成
printf("%d\n", 2 + 3 * 10); // 结果：32
```

宏函数的特点：

1. 宏函数需要使用给小括号修饰，保证运算的完整性
2. 通常会将频繁、短小的函数写成宏函数
3. 宏函数比普通函数在一定程度上要效率高些，省去普通函数入栈、出栈的时间开销（以空间换时间）

## 1.2 普通函数的调用模型

普通函数的调用是需要入栈、出栈的，用同1.1的宏函数相同作用的例子来分析：

```c
#include <stdio.h>

int addnum(int x, int y) {  // x,y入栈
        int xx = x;     // xx 入栈
        int yy = y;     // yy 入栈
        return xx + yy; // 结果暂存到寄存器
}

int main() {
        int sum = 0; // sum入栈
        sum = addnum(2, 5); // 调用结束，从寄存器获取返回值，x、y、xx、yy出栈
        printf("sum = %d\n", sum);
        return 0; // 程序结束，sum出栈
}
```

C/C++下默认调用惯例：`cdecl`

- 规则：从右到左入栈，主调函数管理出栈

代码里面其实隐藏加了调用惯例（示例，再加编译会报错）

```c
__cdecl int addnum(int x, int y) {  // x,y入栈
        int xx = x;     // xx 入栈
        int yy = y;     // yy 入栈
        return xx + yy; // 结果暂存到寄存器
}
```

## 1.3 函数变量传递分析

如图所示五种情况下的函数变量传递：

![image-20221208165725348](../../../img/image-20221208165725348.png)

![image-20221208165236985](../../../img/image-20221208165236985.png)

![image-20221208165254417](../../../img/image-20221208165254417.png)

![image-20221208165302390](../../../img/image-20221208165302390.png)

![image-20221208165313729](../../../img/image-20221208165313729.png)

# 2 栈的生长方向和内存的存放方向

用一个例子来分析栈的生长方向和内存的存放方向，案例如下：

```c
//1. 栈的生长方向
void test01(){

	int a = 10; // 栈底
	int b = 20;
	int c = 30;
	int d = 40; // 栈顶

	printf("a = %d\n", &a);
	printf("b = %d\n", &b);
	printf("c = %d\n", &c);
	printf("d = %d\n", &d);

	//a的地址大于b的地址，故而生长方向向下
}

//2. 内存生长方向(小端模式)
void test02(){
	
	//高位字节 -> 地位字节
	int num = 0xaabbccdd;
	unsigned char* p = &num;

	//从首地址开始的第一个字节
	printf("%x\n",*p);	// dd
	printf("%x\n", *(p + 1));	// cc
	printf("%x\n", *(p + 2));	// bb
	printf("%x\n", *(p + 3));	// aa
}
```

解析：

栈的生长方向（视频中windows是这样，但是我在linux试验，栈的生长方向相反！）：

- 栈底 --- 高地址
- 栈顶 --- 低地址

内存存放方向（小端对齐模式，大端则相反）：

- 高位字节数据 -- 高地址
- 低位字节数据 -- 低地址

如图所示：

![image-20221209111639247](../../../img/image-20221209111639247.png)

# 3 指针强化

## 3.1 指针是一种数据结构

### 3.1.1 指针变量

指针是一种数据类型，占用内存空间（8个字节），用来保存内存地址

```c
#include <stdio.h>

int main() {
        // 指针是有大小的，占用内存空间
        int *p1 = 0x12345;
        int ***p2 = 0x11111;
        printf("p1 size: %d\n", sizeof(p1));	// 8
        printf("p2 size: %d\n", sizeof(p2));	// 8

        // 指针也可以被赋值
        int a = 10;
        p1 = &a;
        printf("p1 address: %p\n", p1); // p1 address: 0x7fea5a7e34
        printf("p2 address: %p\n", p2); // p2 address: 0x11111
        printf("a address: %p\n", &a); // a address: 0x7fea5a7e34
}
```

### 3.1.2 空指针

空指针又称为NULL指针，是一个特殊的指针变量，表示不指向任何地址，要使指针为空指针，可以给它赋NULL值

```c
// 创建一个空指针
int *p = NULL;
```

空指针注意事项：**不允许**向**空指针和非法内存地址**拷贝数据

```c
#include <stdio.h>
#include <string.h>

int main() {
        // 创建指针
        char *p1 = NULL; // 空指针
        char *p2 = 0; // 空指针
        char *p3 = 0x1122; // 非法指针

        // 赋值
        strcpy(p1, "Hello"); // 段错误
        strcpy(p2, "Hello"); // 段错误
        strcpy(p3, "Hello"); // 段错误

        return 0;
}
```

### 3.1.3 野指针

野指针是指向一个**已经被删除的对象**或者是**未申请访问受限内存区域**的指针

什么情况下会产生野指针？

- 指针变量未初始化（根据编译器来，有的会自动优化成NULL）
  - 未初始化的指针会乱指，而不是默认赋值为NULL，为了安全需要初始化指针
- 指针释放后未置为NULL
  - free和delete只是把指针指向的内存干掉，并没干掉指针本身，释放后指针会指向垃圾内存，需要置NULL
- 指针操作超越变量作用域
  - 不要返回指向栈内存的指针或引用，因为栈内存在函数结束后会被释放

操作野指针的案例：

```c
#include <stdio.h>

int main() {
        int *p = 0x12321; // 模拟野指针
        printf("p address : %p\n", p);
        *p = 100; // err 操作非法内存，段错误
}
```

### 3.1.4 指针解引用

指针解引用，也叫**间接访问内存**

```c
#include <stdio.h>

int main() {
        // 间接引用输出变量的值
        int *p = NULL;
        int a = 10;
        p = &a;
        printf("%d\n", *p); // 解引用，输出10
}
```

### 3.1.5 指针的步长

指针是一种数据结构，是指它指向的内存空间的数据类型。指针所指向的内存空间决定了 指针的步长。

步长：指针+1时，移动了多少字节单位

案例如下：

```c
#include <stdio.h>

int main() {
        int a = 0xaabbccdd;
        unsigned int *p1 = &a;
        unsigned char *p2 = &a;

        printf("%x\n", *p1); // aabbccdd
        printf("%x\n", *p2); // dd

        printf("p1 = %d\n", p1); // -843135676
        printf("p1 + 1 = %d\n", p1+1); // -843135672 移动4个字节
        printf("p2 = %d\n", p2); // -843135676
        printf("p2 + 1 = %d\n", p2+2); // -843135674 移动2个字节
}
```

## 3.2 指针的意义-简介赋值

### 3.2.1 间接赋值的三大条件

三个条件：

1. 2个变量，可以有如下两种
   - 一个普通变量，一个指针
   - 一个实参，一个形参
2. 建立关系
3. 通过*操作指向的内存

案例如下：

```c
#include <stdio.h>

int main() {
        int num = 300; // 定义普通变量
        int *p = NULL; // 定义指针变量
        p = &num; // 建立关系
    	*p = 500; // 操作内存
        printf("%d\n", *p); // 通过*解引用，输出500
}

```

### 3.2.2 多级指针定义

多级指针：一级指针存放普通变量的地址，二级指针存放一级指针的地址

案例：

```c
#include <stdio.h>

int main() {
        int b = 30;
        int *p = &b;    // 一级指针
        int **p2 = &p;  // 二级指针
        int ***p3 = &p2;        // 三级指针
        return 0;
}

```

### 3.2.3 间接赋值-1级指针修改变量值

案例如下：

```c
#include <stdio.h>

void changenum(int *a) {
        *a = 300;
}

int main() {
        int a = 30;
        changenum(&a);
        printf("a = %d\n", a); // 输出a = 300
        return 0;
}
```

### 3.2.4 间接赋值-2级指针

案例如下

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void setString(char **p) {
        *p = (char *)malloc(100);	// 解引用二级指针，给里面的一级指针分配空间
        strcpy(*p, "Hello World!");	// 给里面的一级指针拷贝内容
}

void freeSpace(char **p) {
        if (*p != NULL) {
                free(*p);
                *p = NULL;
        } else {
                return;
        }
}

int main() {
        char *p = NULL;
        setString(&p);
        printf("String is : %s\n", p);
        freeSpace(&p);
        return 0;
}
```

## 3.3 指针做函数参数

指针做函数参数，有**输入**和**输出**的特性

- 输入：主调函数分配内存
- 输出：被调函数分配内存

### 3.3.1 输入特性

输入特性：由主动调用的函数来分配内存，案例如下

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void fun(char *p) {
        strcpy(p, "Good DAY");
}

int main() {
        char *str = (char *)malloc(100); // 主调函数分配内存
        fun(str);
        printf("str = %s\n", str);

        free(str);
        str = NULL;
        return 0;
}
```

### 3.3.2 输出特性

输出特性：被调用的函数来分配内存，案例如下

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void fun(char **p, int *len) {
        // 需要输出的指针，由被掉函数分配空间
        char *tmp = (char *)malloc(100);
        if (tmp == NULL) {
                return;
        }
        strcpy(tmp, "Test String");

        // 间接赋值输出
        *p = tmp;
        *len = strlen(tmp);
}

int main() {
        char *p = NULL;
        int len = 0;
        fun(&p, &len);
        if (p != NULL) {
                printf("p = %s, len = %d\n", p, len);
                free(p);
                p = NULL;
        }
        return 0;
}
```

# 4 字符串指针强化

## 4.1 字符串指针做函数参数

了解字符串指针的基本操作、拷贝、反转

### 4.1.1 字符串基本操作

字符串赋值的基本操作

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// 字符串的基本操作

int main() {
        /* 1.字符串是以'0'或者'\0'结尾的字符数组 */
        char str1[] = {'H', 'E', 'L', 'L', 'O'}; // 初始化5个字符
        printf("str1 = %s\n", str1); // 输出，直到遇到0为止 str1 = HELLO

        /* 2.字符数组部分初始化，其余部分默认填0 */
        char str2[100] = {'G', 'O', 'O', 'D'};
        printf("str2 = %s\n", str2); // str2 = GOOD

        /* 3.如果以字符串初始化，那么编译器默认会在字符串尾部添加'\0' */
        char str3[] = "ABCDE";
        printf("str3 = %s\n", str3); // str3 = ABCDE
        // sizeof获取占用内存空间大小, strlen获取实际字符串长度
        printf("str3's sizeof = %d, strlen = %d\n", sizeof(str3), strlen(str3)); // 6 5
        // 如果这样写，sizeof就不一样了
        char str4[100] = "ABCDE";
        printf("str4's sizeof = %d, strlen = %d\n", sizeof(str4), strlen(str4)); // 100 5

        /* 4.下面的情况呢？ */
        char str5[] = "ABC\0DE";
        printf("str5's sizeof = %d, strlen = %d\n", sizeof(str5), strlen(str5)); // 7 3

        char str6[] = "ABC\012DE";
        printf("str6's sizeof = %d, strlen = %d\n", sizeof(str6), strlen(str6)); // 7 6

        return 0;
}
```

### 4.1.2 字符串拷贝功能的实现

有三种方法可以实现，第三种方法最精简

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 方法1 我能想到的
void copy_str1(char *des, char *src) {
        for (int i = 0; src[i]!= '\0'; i++) {
                des[i] = src[i];
        }
}

// 方法2
void copy_str2(char *des, char *src) {
        while(*src != '\0') {
                *des = *src;
                des++;
                src++;
        }
}

// 方法3 最精简
void copy_str3(char *des, char *src) {
        while(*des++ = *src++) {}
}

int main() {
        char *SRC = "Hello World!";
        char *DES = (char *)malloc(100);

        copy_str3(DES, SRC);
        printf("DES = %s\n", DES);

        if (DES != NULL) {
                free(DES);
                DES = NULL;
        }
}
```

### 4.1.3 反转字符串

通过下标反转字符串，案例如下：

```c
#include <stdio.h>
#include <string.h>

void reverse_str(char *str) {
        if (str == NULL) {
                return;
        }

        int begin = 0;
        int end = strlen(str) - 1;
        char temp = 0;

        while(begin < end) {
                // 交换元素
                temp = str[begin];
                str[begin] = str[end];
                str[end] = temp;
                begin++;
                end--;
        }
}

int main() {
        char str[] = "ABCDE1234";
        printf("str = %s\n", str); // str = ABCDE1234
        reverse_str(str);
        printf("reverse str = %s\n", str); // reverse str = 4321EDCBA
        return 0;
}
```

## 4.2 字符串的格式化

### 4.2.1 使用sprintf

`sprintf`方法可以用于：

- 格式化字符串
- 拼接字符串
- 数字转字符串
- 设置字符串对齐
- 字符串进制转换

该方法的描述：

```c
#include <stdio.h>
int sprintf(char *str, const char *format, ...);
功能：
     根据参数format字符串来转换并格式化数据，然后将结果输出到str指定的空间中，直到出现字符串结束符 '\0'为止
参数： 
	str：字符串首地址
	format：字符串格式，用法和printf()一样
返回值：
	成功：实际格式化的字符个数
	失败： - 1
```

使用sprintf的案例：

```c
#include <stdio.h>
#include <string.h>

// 使用sprintf()

int main() {
        // 1、格式化字符串
        char buf[100] = {0};
        sprintf(buf, "你好,%s,欢迎加入我们！", "John");
        printf("%s\n", buf); // 你好,John,欢迎加入我们！

        // 2.拼接字符串
        char str1[] = "I want to";
        char str2[] = "do someting";
        memset(buf, 0,100);
        sprintf(buf, "%s %s", str1, str2);
        printf("%s\n", buf); // I want to do something

        // 3.数字转字符串
        memset(buf, 0, 100); // 内存空间清零
        int age = 200;
        sprintf(buf, "我今年%d岁了", age);
        printf("%s\n", buf); // 我今年200岁了

        // 设置宽度右对齐
        memset(buf, 0, 100);
        sprintf(buf, "%8d\n", age);
        printf("%s\n", buf); //      200

        // 设置宽度左对齐
        memset(buf, 0, 100);
        sprintf(buf, "%-8d\n", age);
        printf("%s\n", buf); //200

        // 转成16进制字符串
        memset(buf, 0, 100);
        sprintf(buf, "200的16进制为：%x\n", age);
        printf("%s\n", buf); //c8


        // 转成8进制字符串
        memset(buf, 0, 100);
        sprintf(buf, "200的8进制为：%od\n", age);
        printf("%s\n", buf); //310d
}
```

### 4.2.2 使用sscanf

`sscanf`可用于数据匹配

它的描述如下：

```c
#include <stdio.h>
int sscanf(const char *str, const char *format, ...);
功能：
    从str指定的字符串读取数据，并根据参数format字符串来转换并格式化数据。
参数：
	str：指定的字符串首地址
	format：字符串格式，用法和scanf()一样
返回值：
	成功：成功则返回参数数目，失败则返回-1
	失败： - 1
```

| **格式**   | **作用**                           |
| ---------- | ---------------------------------- |
| %\*s或%\*d | 跳过数据                           |
| %[width]s  | 读指定宽度的数据                   |
| %[a-z]     | 匹配a到z中任意字符(尽可能多的匹配) |
| %[aBc]     | 匹配a、B、c中一员，贪婪性          |
| %\[^a]     | 匹配非a的任意字符，贪婪性          |
| %\[^a-z]   | 表示读取除a-z以外的所有字符        |

使用案例：

```c
#include <stdio.h>
#include <string.h>

int main() {
        // 1.跳过%d的数据
        char buf[1024] = {0};
        // 一个一个字符匹配，如果是数字则跳过，如果是字符则停止匹配
        sscanf("123456aaa333", "%*d%s", buf); // aaa333
        printf("%s\n", buf);

        // 2.跳过指定宽度的数据, 读取前10个字符
        memset(buf, 0, 100);
        sscanf("123456aaa333", "%10s", buf); // 123456aaa3
        printf("%s\n", buf);

        // 3.匹配a-z中的任意字符
        // 从第一个字符开始匹配，符合条件保留，继续下一个
        // 遇到不符合条件的，停止匹配
        memset(buf, 0, 100);
        sscanf("abc23defg123", "%[a-z]", buf); // abc
        // sscanf("1122abc33", "%[a-z]", buf); // 空
        // sscanf("bAddwq44", "%[a-z]", buf); // b
        printf("%s\n", buf);

        // 4.匹配aBc中任何一个
        memset(buf, 0, 100);
        // sscanf("abcdefg123", "%[aBc]", buf); // a
        // sscanf("123Bdas", "%[aBc]", buf); // 空
        sscanf("Bac432", "%[aBc]", buf); // Bac

        // 5.匹配非a的字符
        memset(buf, 0, 100);
        sscanf("123Bdas", "%[^a]", buf); // 123Bd
        printf("%s\n", buf);

        // 6.匹配非a-z的字符
        memset(buf, 0, 100);
        sscanf("123Bdas", "%[^a-z]", buf); // 123B
        printf("%s\n", buf);
}
```

### 4.2.3 练习

题1：已给定字符串为: `Email:beijing@sina.com.cn`,请编码实现分别输出`Email, beijing, sina, com.cn`

```c
#include <stdio.h>

int main() {
        char buf1[1024] = {0};
        char buf2[1024] = {0};
        char buf3[1024] = {0};
        char buf4[1024] = {0};
        char string[] = "Email:beijing@sina.com.cn";
        sscanf(string, "%[^:]:%[^@]@%[^.].%s", buf1, buf2, buf3, buf4);
        printf("%s, %s, %s, %s\n", buf1, buf2, buf3, buf4);
        return 0;
}
// 输出 Email, beijing, sina, com.cn
```

题2：已给定字符串为:`123abcd$myname@000qwe`请编码实现匹配出`myname`字符串，并输出.

```c
#include <stdio.h>

int main() {
        char buf1[1024] = {0};
        char buf2[1024] = {0};
        char buf3[1024] = {0};
        char buf4[1024] = {0};
        char string[] = "123abcd$myname@000qwe";
        sscanf(string, "%[^$]$%[^@]@", buf1, buf2);
        printf("%s, %s\n", buf1, buf2);
        return 0;
}

// 输出 123abcd, myname
```

# 5 一级指针易错点

## 5.1 越界

越界会造成乱码，以下案例数组大小为3，实际加上‘’\0‘’存入4个字符，已经越界：

```c
#include <stdio.h>

int main() {
        char buf[3] = "abc";
        printf("%s\n", buf);
        return 0;
}

// 输出： abcrU 
```

## 5.2 指针++会不断改变指针指向

指针`++`和`--`会改变指针的指向

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
        char *p = (char *)malloc(100);
        char buf[] = "Abcd Efg";
        int len = strlen(buf);
        printf("len = %d\n", len); // p = 0x5593ebc2a0

        printf("p = %p\n", p);
        for (int i = 0; i < len; i++) {
                *p = buf[i];
                p++; // 修改原指针的指向
        }

        printf("p = %p\n", p); // p = 0x5593ebc2a8

        free(p); // err 段错误

        return 0;
}
```

## 5.3 返回局部变量的地址

```c
char *get_str()
{
	char str[] = "abcdedsgads"; //栈区，
	printf("[get_str]str = %s\n", str);
	return str;
}
```

## 5.4 同一块内存释放多次（不可以释放野指针）

```c
void test(){	
	char *p = NULL;

	p = (char *)malloc(50);
	strcpy(p, "abcdef");

	if (p != NULL)
	{
		//free()函数的功能只是告诉系统 p 指向的内存可以回收了
		// 就是说，p 指向的内存使用权交还给系统
		//但是，p的值还是原来的值(野指针)，p还是指向原来的内存
		free(p); 
	}

	if (p != NULL)
	{
		free(p);
	}
}
```

# 6 const使用

## 6.1 了解const的特性

const用于定义`常量`，即只读变量是不能修改的，但是C语言的常量是<font color=red>**假常量**</font>，可以通过指针修改值。

```c
#include <stdio.h>

int main() {

        // 1.const修饰的"常量"是只读变量，不能修改
        const int b = 300;
        // b = 600; //  error: assignment of read-only variable ‘b’

        // 2.定义const变量最好初始化，否则也不能改变值
        const int a;
        // a = 300; // error: assignment of read-only variable ‘b’

        // 3.C语言的常量是假常量，可以通过指针修改
        int *p = &a;
        *p = 300;
        printf("a = %d\n",a); // 修改成功，a = 300

        return 0;
}
```

## 6.2 const修饰指针

const修饰指针，放在不同的地方有不同的效果，比如：

```c
int a = 10;
// 三种修饰方法
const int *p1 = &a;	// 修饰值（值不能改）
int * const p2 = &a;	// 修饰指针（指针指向不能改）
const int * const p3 = &a;	// 修饰值和指针（值和指针指向都不能改）
```

好家伙，是不是当场迷乱？那么这三种修饰方法到底有什么作用，我们通过下面的案例来了解：

```c
#include <stdio.h>

// 分别演示三种const修饰指针的方法

/*
 * 1.const int *p1（修饰值）
 *      值不能改
 *      指针指向能改
 * */
void test01() {
        int a = 10;
        int b = 20;

        const int *p1 = &a;
        // *p1 = 300; // error: assignment of read-only location ‘*p1’
        p1 = &b;
        printf("*p1现在的值是: %d\n", *p1); // 20
}



/*
 * 2.int * const p2（修饰指针）
 *      值能改
 *      指针指向不能改
 */
void test02() {
        int a = 10;
        int b = 20;

        int * const p2 = &a;
        *p2 = 300;
        printf("*p2 = %d\n", *p2); // 300
        // p2 = &b; // error: assignment of read-only variable ‘p2’
}


/*
 * 3.const int * const p3（修饰指针和值）
 *      值和指针指向都不能改
 */
void test03() {
        int a = 10;
        int b = 20;

        const int * const p3 = &a;
        // *p3 = 500; // error: assignment of read-only location ‘*p3’
        // p3 = &b; // error: assignment of read-only variable ‘p3’
}


int main() {
        test01();
        test02();
        test03();
        return 0;
}

```



