---
layout: post
title: "大话数据结构: 线性表的顺序存储结构"
date:  2020-06-08 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 线性表的顺序存储结构

> 线性表的顺序存储的结构代码。
```c
#define MAXSIZE 20 /*存储空间初始分配量*/
typedef int ElemType; /*ElemType类型根据实际视情况而定，这里假设为int*/
typedef struct
{
    ElemType data[MAXSIE]; /*数组存储数据元素，最大值为MAXSIZE*/
    int length; /*线性表当前长度*/
}SqList
```

顺序存储结构需要三个属性：

* 存储空间的起始位置：数组data，它的存储位置就是存储空间的存储位置。
* 线性表的最大存储容量：数组长度MaxSize
* 线性表的当前长度：length

### 数组长度与线性表长度区别
数组的长度是存放线性表的存储空间的长度，存储分配后这个量一般是不变的。

线性表的长度是线性表中元素的个数，随着线性表插入和删除操作的进行，这个量是变化的。

### 地址计算方法
存储器中每个存储单元都有自己的编号，这个编号称为地址。

假设每个数据元素占用的是c个存储单元，那么线性表中第i+1个数据元素的存储位置和第i个数据的存储位置满足下列关系（LOC表示获得存储位置的函数）
```math
LOC(a_{i+1}) = LOC(a_i) + c
```
所以对于第i个数据元素ai的存储位置可以由a1推算得出

```math
LOC(a_i) = LOC(a_1) + (i-1)*c
```


### 获得元素操作
即：只要i的数值在数组的下标范围内，就是把数组第i-1下表的值返回即可

```c
#define OK 1
#define ERROR 0
#define TRUE 1 
#define FALSE 0
typedef int Status;
/*Status是函数的类型，其值是函数结果状态代码，如OK等*/
/*初始条件：顺序线性表L已存在，1<=i<=ListLength(L)*/
/*操作结果：用e返回L中第i个数据元素的值*/
Status GetElem(SqList L,int i,ElemType *e)
{
    if(L.length==0 || i<1 || i>length)
        return ERROR;
    *e=L.data[i-1];
    return OK;
}
```

### 插入操作

思路：

* 如果插入位置不合理，抛出异常；
* 如果线性表长度大于等于数组长度，则抛出异常或动态增加容量；
* 从最后一个元素开始向前便利到第i个位置，分别都将它们向后移动一个位置；
* 将要插入元素填入位置i处；
* 表长加1

```c
/*初始条件：顺序线性表L已存在，1<=i<=ListLength(L)*/
/*操作结果：在L中第i个位置之前插入新的数据元素e，L的长度加1*/
Status ListInsert(SqList *L, int i, ElemType e)
{
    int k;
    if (L->length==MAXSIZE) /*顺序线性表已满*/
        return ERROR;
    if (i<1 || i>L->length+1)/*当i不再范围内时*/
        return ERROR;
    if (i<=L->length) /*若插入数据位置不在表尾*/
    {
        for (k=L->length-1;k>=i-1;k--) /*将要插入位置后数据元素向后移动一位*/
            L->data[k+1]=L->data[k];
    }
    L->data[i-1]=e; /*将新元素插入*/
    L->length++;
    return OK;
}
```

### 删除操作
思路：

* 如果删除位置不合理，抛出异常；
* 取出删除元素；
* 从删除元素位置开始便利到最后一个元素位置，分别将它们都向前移动一个位置；
* 表长减1

```c
/*初始条件：顺序线性表L已存在，1<=i<=ListLength(L)*/
/*操作结果：删除L的第i个数据元素，并用e返回其值，L的长度减1*/
Status ListDelete(SqList *L,int i, ElemType *e)
{
    int k;
    if (L->length==0) 
        return ERROR;
    if (i<1 || i>L->length)
        return ERROR;
    *e=L>data[i-1];
    if (i<L->length)
    {
        for(k=i;k<=L->length;k++)
            L->data[k-1]=L->data[k]
    }
    L->length--;
    return OK;
}
```

线性表的顺序存储结构，在存、读数据时，不管是哪个位置，时间复杂度都是O(1);而插入或删除时，时间复杂度都是O(n)。这就说明，它比较适合元素个数不太变化，而更多是存取数据的应用。

### 线性表顺序存储结构的优缺点

* 优点
    * 无须为表示表中元素之间的逻辑关系而增加额外的存储空间
    * 可以快速地存取表中任一位置的元素
* 缺点
    * 插入和删除操作需要移动大量元素
    * 当线性表长度变化较大时，难以确定存储空间的容量
    * 造成存储空间的“碎片”


参考资料：

- [大话数据结构]()