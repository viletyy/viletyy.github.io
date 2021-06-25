---
layout: post
title: "大话数据结构：队列"
date:  2020-06-14 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 队列
**队列**（Queue）是只允许在一端进入插入操作，而在另一端进行删除操作的线性表

队列是一种先进先出（First In First Out）的线性表，简称FIFO。允许插入的一端称为队尾，允许删除的一端称为对头。

### 队列的抽象数据类型
```c
ADT 队列(Queue)
Data
    同线性表。元素具有相同的类型，相邻元素具有前驱和后继关系。
Operation
    InitQueue(*Q): 初始化操作，建立一个空队列Q。
    DestroyQueue(*Q): 若队列Q存在，则销毁它。
    ClearQuene(*Q): 将队列Q清空。
    QueueEmpty(Q): 若队列Q为空，返回true，否则返回false。
    GetHead(Q, *e): 若队列Q存在且非空，用e返回队列Q的对头元素。
    EnQueue(*Q, e): 若队列Q存在，插入新元素e到队列Q中并称为队尾元素。
    DeQueue(*Q, e): 删除队列Q中队头元素，并用e返回其值。
    QueueLength(Q): 返回队列Q的元素个数。
endADT
```

### 循环队列
队列这种头尾相接的顺序存储结构称为循环队列。

最大尺寸为QueueSIze

队列满的条件是`(rear+1)%QueueSize == front`

通用的计算队列长度公式：`(rear-front+QueueSize)%QueueSize`

#### 结构

```c
typedef int QElemType; /*QElemType类型根据实际情况而定，这里假设为int*/
/*循环队列的顺序存储结构*/
typedef struct 
{
    QElemType data[MAXSIZE];
    int front; /*头指针*/
    int rear; /*尾指针，若队列不空，指向队列尾元素的下一个位置*/
}SqQueue;
```

#### 初始化
```c
Status InitQueue(SqQueue *Q)
{
    Q->front=0;
    Q->rear=0;
    return OK;
}
```

#### 求队列长度
```c
int QueueLength(SqQueue Q)
{
    return (Q.rear - Q.front + MAXSIZE) % MAXSIZE;
}
```

#### 入队操作
```c
/*若队列未满，则插入元素e为Q新的队尾元素*/
Status EnQueue(SqQueue *Q, QElemType e)
{
    if ((Q->rear+1)%MAXSIZE == Q->front) /*队列满的判断*/
        return ERROR;
    Q->data[Q->rear]=e; /*将元素e赋值给队尾*/
    Q->rear=(Q->rear+1)%MAXSIZE; /*rear指针向后移一位，若到最后则转到数组头部*/
    
    return OK;
}
```

#### 出队操作
```c
/*若队列不空，则删除Q中队头元素，用e返回其值*/
Status DeQueue(SqQueue *Q, QElemType *e)
{
    if (Q->front == Q->rear) /*队列空的判断*/
        return ERROR;
    *e=Q->data[Q->front];   /*将队头元素赋值给e*/
    Q->front=(Q->front+1)%MAXSIZE; /*front指针向后移一位，若到最后则转到数组头部*/
    
    return OK;
}
```

### 队列的链式存储结构及实现
队列的链式存储结构，其实就是线性表的单链表，只不过它只能尾进头出而已，我们把它简称为链队列。
#### 结构
```c
typedef int QElemType; /*QElemType类型根据实际情况而定，这里假设为int*/

typedef struct QNode /*结点结构*/
{
    QElemType data;
    struct QNode *next;
}Qnode, *QueuePtr;

typedef struct  /*队列的链表结构*/
{
    QueuePtr front,  rear; /*队头，队尾指针*/
}LinkQueue;
```

#### 入队操作
```c
/*插入元素e为Q的新的队尾元素*/
Status EnQueue(LinkQueue *Q, QElemType e)
{
    QueuePtr s =(QueuePtr)malloc(sizeof(QNode));
    if(!s) /*存储分配失败*/
        exit(OVERFLOW);
    s->data=e;
    s->next=NULL;
    Q->rear->next=s; /*把拥有元素e新结点s赋值给原队尾结点的后继*/
    Q->rear=s; /*把当前的s设置为队尾结点，rear指向s*/
    return OK;
}
```

#### 出队操作
```c
/*若队列不空，删除Q的队头元素，用e返回其值，并返回OK，否则返回ERROR*/
Status DeQueue(LinkQueue *Q, QElemType *e)
{
    QueuePtr p;
    if (Q->front==Q->rear)
        return ERROR;
    p=Q->front->next; /*将欲删除的队头结点暂存给p*/
    *e = p->data; /*将欲删除的对头结点的值赋值给e*/
    Q->front->next=p->next /*将原队头结点后继p->next赋值给头结点后继*/
    if (Q->rear == p) /*若队头是队尾，则删除后将rear指向头结点*/
        Q->rear = Q->front;
    free(p);
    return OK;
}
```


参考资料：

- [大话数据结构]()