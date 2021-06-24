---
layout: post
title: "大话数据结构：线性表的双向链表存储结构"
date:  2020-06-12 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 双向链表
**双向链表**（double linked list）是在单链表的每个结点中，再设置一个指向其前驱结点的指针域。

### 线性表的双向链表存储结构
```c
typedef struct DulNode
{
    ElemType data;
    struct DulNode *prior;
    struct DulNode *next;
} DulNode, *DuLinkList
```

双向链表是单链表中扩展出来的结构，所以它的很多操作是和单链表相同的，比如求长度的ListLength，查找元素GetElem，获得元素位置的LocateElem等，这些操作都只要涉及一个方向的指针即可，另一指针多了也不能提供什么帮助。

### 双向链表的插入操作
存储元素e的结点为s，实现结点s插入到结点p和p->next之间
```c
s->prior = p; /*把p赋值给s的前驱*/
s->next = p->next; /*把p->next赋值给s的后继*/
p->next -> prior = s; /*把s赋值给p->next的前驱*/
p->next = s; /*把s赋值给p的后继*/
```

### 双向链表的删除操作
删除p结点
```c
p->prior->next = p->next; /*把p->next赋值给p->prior的后继*/
p->next->prior = p->prior; /*把p->prior赋值给p->next的前驱*/
free(p); /*释放p*/
```


参考资料：

- [大话数据结构]()