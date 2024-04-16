---
title: C语言-Day08
date: 2024-03-20 15:36:22
categories:
- C++
- 01_C语言
tags:
---

“C语言部分-Day08”

# 一、数据结构

常用数据结构分为链表、栈、队列、哈希表、红黑树等

## 1.1 链表

### 1.1.1 单链表的基本操作

初始化链表：

- 创建链表：create_list

增加节点：

- 头插法：add_before_head
- 尾插法：add_behind_tail
- 根据索引插入：add_node

删除节点：

- 根据索引删除：remove_node
- 销毁链表：destroy_list

查询节点：

- 根据数值查询：indexOf



**<font color=blue>实现单链表的基本操作</font>**

头文件：linklist.h

```c
#include <stdbool.h>

// 链表的接口
typedef struct node_s {
	int val;
	struct node_s* next;
} Node;

typedef struct linkedlist_s {
	Node* head;
	Node* tail;
	int size;
} LinkedList;

// 构造方法: 创建一个空链表
LinkedList* create_list();

// 析构方法：释放堆堆存空间
void destroy_list(LinkedList* list);

void add_before_head(LinkedList* list, int val);
void add_behind_tail(LinkedList* list, int val);
void add_node(LinkedList* list, int index, int val);

// 删除第一个与val相等的结点, 如果没有这样的结点返回false
bool remove_node(LinkedList* list, int val);

// 找出第一个与val相等结点的索引，如果没有这样的结点, 返回-1
int indexOf(LinkedList* list, int val);

// 打印
void print_list(LinkedList* list);
```

实现：linklist.c

```c
// 链表的实现
#include "linklist.h"
#include <stdlib.h>
#include <stdio.h>

// 创建空链表
LinkedList* create_list() {
	return (LinkedList*)calloc(1, sizeof(LinkedList));
}

// 头插法
void add_before_head(LinkedList* list, int val) {
	// 创建新结点
	Node* newNode = (Node*)malloc(sizeof(Node));
	if (newNode == NULL) {
		printf("Error: malloc failed in add_before_head.\n");
		exit(1);
	}
	// 初始化结点
	newNode->val = val;
	newNode->next = list->head;
	list->head = newNode;
	// 判断链表是否为空
	if (list->tail == NULL) {
		list->tail = newNode;
	}
	// 更新size
	list->size++;
}

// 尾插法
void add_behind_tail(LinkedList* list, int val) {
	// 创建新结点
	Node* newNode = (Node*)malloc(sizeof(Node));
	if (newNode == NULL) {
		printf("Error: malloc failed in add_before_head.\n");
		exit(1);
	}
	// 初始化结点
	newNode->val = val;
	newNode->next = NULL;
	// 判断链表是否为空
	if (list->size == 0) {
		list->head = newNode;
		list->tail = newNode;
	}
	else {
		// 链接新结点
		list->tail->next = newNode;
		// 更新list->tail
		list->tail = newNode;
	}
	list->size++;
}

void add_node(LinkedList* list, int index, int val) {
	if (index < 0 || index > list->size) {
		printf("Error: Illegal index.\n");
		exit(1);
	}
	// 创建新结点
	Node* newNode = (Node*)malloc(sizeof(Node));
	if (newNode == NULL) {
		printf("Error: malloc failed in add_before_head.\n");
		exit(1);
	}
	newNode->val = val;

	if (index == 0) {
		// 头插法的逻辑
		newNode->next = list->head;
		list->head = newNode;
	}
	else {
		// 找到索引为 indx-1 的结点
		Node* p = list->head;
		for (int i = 0; i < index - 1; i++) {
			p = p->next;
		}
		newNode->next = p->next;
		p->next = newNode;
	}
	// 更新尾结点
	if (index == list->size) {
		list->tail = newNode;
	}
	list->size++;
}

// 删除第一个与val相等的结点, 如果没有这样的结点返回false
bool remove_node(LinkedList* list, int val) {
	Node* prev = NULL;
	Node* curr = list->head;
	// 寻找前驱结点
	while (curr != NULL && curr->val != val) { // 短路原则
		prev = curr;
		curr = curr->next;
	}
	// 没有这样的元素
	if (curr == NULL) return false;
	// 删除第一个元素
	if (prev == NULL) {
		if (list->size == 1) {
			list->head = list->tail = NULL;
		}
		else {
			list->head = curr->next;
		}
		free(curr);
	}
	else {
		prev->next = curr->next;
		if (prev->next == NULL) {
			list->tail = prev;
		}
		free(curr);
	}
	list->size--;
	return true;
}

// 找出第一个与val相等结点的索引，如果没有这样的结点, 返回-1
int indexOf(LinkedList* list, int val) {
	Node* curr = list->head;
	for (int i = 0; i < list->size; i++, curr = curr->next) {
		if (curr->val == val) {
			return i;
		}
	}
	// 没有找到
	return -1;
}

void destroy_list(LinkedList* list) {
	// 释放结点空间
	Node* curr = list->head;
	while (curr != NULL) {
		// 保存curr后继结点
		Node* next = curr->next;
		free(curr);
		curr = next;
	}
	// 释放LinkedList结构体
	free(list);
}

void print_list(LinkedList *list) {
	if (list == NULL) {
		return;
	}

	Node* curr = list->head;
	printf("Head: %d\nTail: %d\nSize: %d\n", list->head->val, list->tail->val, list->size);
	while (curr != NULL) {
		printf("%d ", curr->val);
		curr = curr->next;
	}
	printf("%d\n------------------\n");
}
```

测试：

```c
#include <stdio.h>
#include "linklist.h"

// 测试
int main(void) {
	// 创建链表
	LinkedList* list = create_list();

	// 位置插入
	add_node(list, 0, 55);	// 55
	print_list(list);

	// 头插
	add_before_head(list, 11);	// 11 55
	add_before_head(list, 22);	// 22 11 55

	print_list(list);

	// 尾插
	add_behind_tail(list, 33);	// 22 11 55 33
	add_behind_tail(list, 44);	// 22 11 55 33 44
	print_list(list);

	// 位置插入
	add_node(list, 2, 66);	// 22 11 66 55 33 44
	print_list(list);

	add_node(list, 5, 77);	// 22 11 66 55 33 77 44
	print_list(list);

	add_behind_tail(list, 88);	// 22 11 66 55 33 77 44 88
	print_list(list);

	// 按值，删除结点
	remove_node(list, 44);
	print_list(list);	// 22 11 66 55 33 77 88

	remove_node(list, 22);
	print_list(list);	// 11 66 55 33 77 88

	remove_node(list, 33);
	print_list(list);	// 11 66 55 77 88

	remove_node(list, 44);
	print_list(list);	// 11 66 55 77 88

	// 查找结点
	printf("Index = %d\n", indexOf(list, 66));	// 1
	printf("Index = %d\n", indexOf(list, 11));	// 0	
	printf("Index = %d\n", indexOf(list, 44));	// -1
	printf("Index = %d\n", indexOf(list, 55));	// 2

	// 销毁链表
	destroy_list(list);
	printf("Index = %d\n", indexOf(list, 55));	// -1

	return 0;
}
```

### 1.1.3 双向和单向链表的对比

时间复杂度对比

| 对比项                         | 单向链表 | 双向链表 | 备注                                             |
| ------------------------------ | -------- | -------- | ------------------------------------------------ |
| 增加结点（在某个结点前面添加） | O(n)     | O(1)     |                                                  |
| 删除结点（                     | O(n)     | O(1)     |                                                  |
| 查找：前驱结点                 | O(n)     | O(1)     |                                                  |
| 查找：根据index                | O(n)     | O(n)     | 单向链表只能单向遍历，双向链表可以逆向，双向更快 |
| 查找：根据value（大小有序）    | O(n)     | O(n)     |                                                  |
| 查找：根据value（大小无序）    | O(n)     | O(n)     |                                                  |





