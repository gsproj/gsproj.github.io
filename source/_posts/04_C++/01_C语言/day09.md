---
title: C语言-Day09
date: 2024-03-20 15:36:22
categories:
- C++
- 01_C语言
tags:
---

“C语言部分-Day09”

# 一、数据结构

常用数据结构分为链表、栈、队列、哈希表、红黑树等

## 1.2 栈

栈是一种先进先出（FIFO）的数据结构，它是一种受限的线性表，只能在栈顶添加和删除元素，图示如下：

![image-20240417155227522](D:/C++/gsproj.github.io/source/img/image-20240417155227522.png)

### 1.2.1 栈的作用

1、函数调用

2、深度优先遍历

3、浏览器访问页面的前进和后退

4、括号匹配问题（※※※）

5、后缀表达式求值

- 前缀表达式：1 + 2 * 3
- 后缀表达式：1 2 3 * +



### 1.2.2 栈基本操作：

- 入栈（push）
- 出栈（pop）
- 是否为空（isEmpty）
- 查（peek）



### 1.2.3 实现栈的基本操作

栈可以用数组或者链表来实现



#### 1.2.3.1 用链表实现

mystack.h

```c
#include <stdbool.h>
#include <stdlib.h>

// 定义结点结构
typedef struct node_s {
	int val;
	struct node_s* next;
}Node;

// 方法一：入栈
void push(Node** ptr_top, int val);

// 方法二：判断是否为空
bool isEmpty(Node** ptr_top);

// 方法三：出栈
int pop(Node** ptr_top);

// 方法四：查找
int peek(Node* top);
```

mystack.c

``` c
#include "stack.h"

// 方法一：入栈
void push(Node** ptr_top, int val) {
	// 创建结点，分配空间
	Node* newnode = (Node*)malloc(sizeof(Node));
	// 容错判断
	if (newnode == NULL) {
		printf("Error! malloc failed in push.");
		exit(1);
	}
	// 给新结点赋值
	newnode->val = val;
	// 追加节点
	newnode->next = *ptr_top;
	*(ptr_top) = newnode;
}

// 方法二：判断是否为空
bool isEmpty(Node* ptr_top) {
	return ptr_top == NULL;
}


// 方法三：出栈
int pop(Node** ptr_top) {
	// 容错判断
	if (isEmpty(*ptr_top)) {
		printf("Error! stack is empty!");
		exit(1);
	}
	// 保留弹出结点
	Node* old_top = *ptr_top;
	int result = old_top->val;
	// 设置新的栈顶
	*ptr_top = (*ptr_top)->next;
	// 释放旧结点
	free(old_top);
	// 返回值
	return result;
}

// 方法四：查找
int peek(Node* top) {
	if (isEmpty(top)) {
		printf("Error! stack is empty!");
		exit(1);
	}
	// 返回查找的值
	return top->val;
}
```

main.c测试

```c
#include "stack.h"
#include <stdio.h>

int main() {

	Node* top = NULL;
	push(&top, 1);
	push(&top, 2);
	push(&top, 3);
	push(&top, 4);

	printf("%d\n", peek(top));	// 输出4

	// 出栈
	printf("%d\n", pop(&top));	//	4
	printf("%d\n", pop(&top));	// 3

	// 查找
	printf("%d\n", peek(top));	// 输出2

	// 查空
	while (!isEmpty(top)) {
		printf("%d\n", pop(&top));	// 输出：2，1，停止
	}

	return 0;
}
```



## 1.3 队列

队列也是一种受限的线性表