---
layout: post
title: "大话数据结构：栈"
date:  2020-06-13 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 栈
**栈**（Stack）是限定仅在表尾进行插入和删除操作的线性表。

允许插入和删除的一段称为栈顶(top)，另一端称为栈底(bottom),不含任何数据元素的栈称为空栈。栈又称为后进先出(Last In First Out)的线性表，简称LIFO结构。

栈的插入操作，也叫进栈，也称压栈、入栈。

栈的删除操作，也叫出栈，也有的叫做弹栈。

### 栈的抽象数据类型
```c
ADT 栈(stack)
Data
    同线性表。元素具有相同类型，相邻元素具有前驱和后继关系。
Operation
    InitStack(*S): 初始化操作，建立一个空栈。
    DestroyStack(*S): 若栈存在，则销毁它。
    ClearStack(*S): 将栈清空。
    StackEmpty(S): 若栈为空，返回true，否则返回false。
    GetTop(S, *e): 若栈存在且非空，用e返回S的栈顶元素。
    Push(*S, e): 若栈S存在，插入新元素e到栈S中并成为栈顶元素。
    Pop(*S, *e): 删除栈S中栈顶元素，并用e返回其值。
    StackLength(S): 返回栈S的元素个素。
endADT
```

### 栈的顺序存储结构及实现
结构定义
```c
typedef int SElemType; /*SElemType类型根据实际情况而定，这里假设为int*/
typedef struct 
{
    SElemType data[MAXSIZE];
    int top; /*用户栈顶指针*/
}SqStack
```

#### 进栈操作
```c
/*插入元素e为新的栈顶元素*/
Status Push(SqStack *S, SElemType e)
{
    if (S->top == MAXSIZE - 1) /*栈满*/
        return ERROR;
    S->top++; /*栈顶指针增加1*/
    S->data[S->top]=e; /*将新插入元素赋值给栈顶空间*/
    return OK;
}
```
#### 出栈操作
```c
/*若栈不空，则删除S的栈顶元素，用e返回其值，并返回OK；否则返回ERROR*/
Status Pop(SqStack *S, SElemType *e) 
{
    if (S->top==-1)
        return ERROR;
    *e=S->data[S->top];
    S->top--;
    return OK;
}
```

### 两栈共享空间
结构定义
```c
/*两栈共享空间结构*/
typedef struct 
{
    SElemType data[MAXSIZE];
    int top1; /*栈1栈顶指针*/
    int top2; /*栈2栈顶指针*/
}SqDoubleStack;
```
#### 进栈操作
```c
/*插入元素e为新的栈顶元素*/
Status Push(SqDoubleStack *S, SElemType e, int stackNumber)
{
    if (S->top1 + 1 == S->top) /*栈已满，不能再push新元素了*/
        return ERROR;
    if (stackNumber == 1) /*栈1有元素进栈*/
        S->data[++S->top1]=e; /*先增加栈1栈顶指针，然后赋值为栈1栈顶元素*/
    if (stackNumber == 2) /*栈2有元素进栈*/
        S->data[++S->top2]=e; /*先增加栈2栈顶指针，然后赋值为栈2栈顶元素*/
    return OK;
}
```

#### 出栈操作
```c
/*若栈不空，则删除S的栈顶元素，用e返回其值，并返回OK；否则返回ERROR*/
Status Pop(SqDoubleStack *S, SElemType *e, int stackNumber)
{
    if (stackNumber==1)
    {
        if (S->top1 == -1)  
            return ERROR; /*栈1已经是空栈，溢出*/
        *e=S->data[S->top1--]; /*将栈1栈顶元素出栈*/
    }
    else if (stackNumber==2)
    {
        if (S->top2 == MAXSIZE -1) 
            return ERROR; /*栈2已经是空栈，溢出*/
        *e=S->data[S->top2++]; /*将栈2栈顶元素出栈*/
    }
    return OK;
}
```

### 栈的链式存储结构及实现
结构如下
```c
typedef struct StackNode
{
    SElemType data;
    struct StackNode *next;
}StackNode, *LinkStackPtr;

typedef struct LinkStack
{
    LinkStackPtr top;
    int count;
}LinkStack;
```

#### 进栈操作
```c
/*插入元素e为新的栈顶元素*/
Status Push(LinkStack *S, SElemType e)
{
    LinkStackPtr s = (LinkStackPtr)malloc(sizeof(StackNode));
    s->data = e;
    s->next=S->top; /*把当前的栈顶元素赋值给新结点的直接后继*/
    S->top = s; /*将新的结点s赋值给栈顶指针*/
    S->count++;
    return OK;
}
```
#### 出栈操作
```c
/*若栈不空，则删除S的栈顶元素，用e返回其值，并返回OK；否则返回ERROR*/
Status Pop(LinkStack *S, SElemType *e)
{
    LinkStackPtr p;
    if (StackEmpty(*S)) 
        return ERROR;
    *e=S->top->data;
    p=S->top; /*将栈顶结点赋值给p*/
    S->top=S->top->next; /*使得栈顶指针下移一位，指向后一结点*/
    free(p); /*释放结点p*/
    S->count--;
    return OK;
}
```
### 栈的应用——递归

#### 斐波那契数列
前面相邻两项之和，构成了后一项。

> 常规的迭代方法，实现打印出前40位的斐波那契数列数

```c
int main()
{
    int i;
    int a[40];
    a[0] = 0;
    a[1] = 1;
    printf("%d ", a[0])
    printf("%d ", a[1])
    for (i = 2; i < 40; i++)
    {
        a[i] = a[i-1] + a[i-2];
        printf("%d ", a[i]);    
    }
    return 0;
}
```

> 斐波那契数列的递归函数

```c
int Fbi(int i)
{
    if ( i < 2 )
        return i == 0 ? 0 : 1
    return Fbi(i-1) + Fbi(i-2);
}

int main()
{
    int i;
    for (int i = 0; i < 40; i++)
        printf("%d ", Fbi(i));
    return 0;                    
}
```

#### 递归定义 
把一个直接调用自己或通过一系列的调用语句简介地调用自己的函数，称作递归函数。

每个递归定义必须至少有一个条件，满足时递归不再进行，即不再引用自身而是返回值退出。

### 栈的应用——四则运算表达式求值
#### 后缀（逆波兰）表示法定义
一种不需要括号的后缀表达法，我们也把它称为逆波兰(Reverse Polish Notation, RPN)表示。

叫后缀的原因在于所有的符号都是在要运算数字的后面出现。

#### 后缀表达式计算结果

后缀表达式

`9 3 1 - 3 * + 10 2 / +`

规则：从左到右遍历表达式的每个数字和符号，遇到数字就进栈，遇到是符号，就将处于栈顶两个数字出栈，进行运算，运算结果进栈，一直到最终获得结果。

1. 初始化一个空栈。此栈用来对要运算的数字进出使用。
2. 后缀表达式中的前三个都是数字，所以`9`、`3`、`1`进栈
3. 接下来是`-`，所以将栈中的`1`出栈做为减数，`3`出栈做为被减数，并运算`3-1`得到`2`，再将`2`进栈
4. 接下来是数字`3`进栈
5. 后面是`*`，也就意味着栈中`3`和`2`出栈，`2`与`3`相乘，得到`6`，并将`6`进栈。
6. 下面是`+`，所以栈中`6`和`9`出栈，`9`和`6`相加，得到`15`，将`15`进栈
7. 接着是`10`与`2`两数字进栈
8. 接下来是符号`/`，因此，栈顶的`2`与`10`出栈，`10`与`2`相除，得到`5`，将`5`进栈
9. 最后一个符号`+`，所以`15`与`5`出栈并相加，得到`20`，将`20`进栈
10. 结果是`20`出栈，栈变为空。

#### 中缀表达式转后缀表达式 
平时所用的标准四则运算表达式，即`9 + (3 - 1) * 3 + 10 / 2`叫做中缀表达式。

中缀表达式`9 + (3 - 1) * 3 + 10 / 2`
转化为后缀表达式`9 3 1 - 3 * + 10 2 / + `

规则：从左到右遍历中缀表达式的每个数字和符号，若是数字就输出，即称为后缀表达式的一部分；若是符号，则判断其与栈顶符号的优先级，是右括号或优先级低于栈顶符号（乘除优先加减）则栈顶元素依次出栈并输出，并将当前符号进栈，一直刀最终输出后缀表达式为止。

1. 初始化一空栈，用来对符号进出栈使用。
2. 第一个数字是`9`，输出`9`，后面是符号`+`，进栈
3. 第三个字符是`（`，依然是符号，因其是左括号，还未配对，故进栈。
4. 第四个字符是数字`3`，输出，总表达式为`9 3`，接着是`-`，进栈。
5. 接下来是数字`1`，输出，总表达式为`9 3 1`，后面是符号`（`，此时我们需要去匹配此前的`（`，所以栈顶一次出栈，并输出，直到`（`出栈为止。此时左括号上方只有`-`,因此输出`-`。总的输出表达式为`9 3 1 -`。
6. 接着是数字`3`，输出，总表达式为`9 3 1 - 3`。紧接这是符号`*`，因此此时的栈顶符号为`+`号，优先级低于`*`，因此不输出，`*`进栈
7. 之后是符号`+`，此时当前栈顶元素`*`比这个`+`优先级高，因此栈中元素出栈并输出（没有比`+`号更低的优先级，所以全部出栈），总输出表达式为`9 3 1 - 3 * +`。然后将当前这个符号`+`进栈，也就是说，前6步的栈底的`+`指的是中缀表示式开头的`9`后面的那个`+`，而现在栈底的`+`指的是最后一个`+`
8. 紧接着数字`10`，输出，总表达式变为`9 3 1 - 3 * + 10`，后面是符号 `/`，所以 `/`进栈
9. 最后一个数字`2`，输出，总的表达式为`9 3 1 - 3 * + 10 2`
10. 因已经到最后了，所以将栈中符号全部出栈并输出，最终输出到后缀表达式结果为`9 3 1 - 3 * + 10 2 / +`



参考资料：

- [大话数据结构]()