---
layout: post
title: "大话数据结构：赫夫曼树及其应用"
date:  2020-06-19 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 赫夫曼树及其应用

### 赫夫曼树定义与原理

从树中一个结点到另一个结点之间的分支构成两个结点之间的路径，路径上的分支数目称作路径长度。

树的路径长度就是从树根到每一结点的路径长度之和

带权路径长度WPL最小的二叉树称为赫夫曼树 

构造赫夫曼树的赫夫曼算法描述。

1. 根据给定的n个权值{w1,w2, ... , wn} 构成n棵二叉树的集合F={T1, T2, ..., Tn}，其中每棵二叉树Ti中只有一个带权为wi根结点，其左右子树均为空
2. 在F中选取两棵根结点的权值最小的树作为左右子树构造一棵新的二叉树，且置新的二叉树的根结点的权值为其左右子树上根结点权值之和。
3. 在F中删除这两棵树，同时将新得到的二叉树加入F中。
4. 重复2和3步骤，直到F只含一棵树为止。这棵树便是赫夫曼树

### 赫夫曼编码

若要设计长短不等的编码，则必须是任一字符的编码都不是另一字符的编码的前缀，这种编码称作前缀编码。

一般地，设需要编码的字符集为{d1,d2,...,dn}，各个字符在电文中出现的次数或频率集合为{w1,w2,...,wn}，以d1,d2,...,dn作为叶子结点，以w1,w2,...,wn作为相应叶子结点的权值来构造一棵赫夫曼树。规定赫夫曼树的左分支代表0，右分支代表1，则从根结点到叶子结点所经过的路径分支组成的0和1的序列便为该结点对应字符的编码，这就是赫夫曼编码



参考资料：

- [大话数据结构]()