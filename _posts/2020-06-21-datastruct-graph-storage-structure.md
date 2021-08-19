---
layout: post
title: "大话数据结构：图的存储结构"
date:  2020-06-21 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 图的抽象数据类型

```c
ADT 图(Graph)
Data
  	顶点的有穷非空集合和边的集合。
Operation
  	CreateGraph(*G, V, VR): 按照顶点集V和边弧集VR的定义构造图G
		DestroyGraph(*G):	图G存在则销毁
		LocateVex(G, u): 若图G中存在顶点u，则返回图中的位置
		GetVex(G, v): 返回图G中顶点v的值
		PutVex(G, v, value): 将图G中顶点v赋值value
		FirstAdjVex(G, *v): 返回顶点v的一个邻接顶点，若顶点在G中无邻接顶点返回空
		NextAdjVex(G, v, *w): 返回顶点v相对于顶点w的下一个邻接顶点，若w是v的最后一个邻接点则返回“空”
		InsertVex(*G, v): 在图G中增添新顶点
		DeleteVex(*G, v): 删除图G中顶点v及其相关的弧
		InsertArc(*G, v, w): 在图G中增添弧<v,w>, 若G是无向图，还需要增添对称弧<w,v>
		DeleteArc(*G, v, w): 在图G中删除弧<v,w>, 若G是无向图，则还删除对称弧<w,v>
		DFSTraverse(G): 对图G中进行深度优先遍历，在遍历过程对每个顶点调用
		HFSTraverse(G): 对图G中进行广度优先遍历，在遍历过程对每个顶点调用
endADT
```

## 图的存储结构

### 邻接矩阵

图的邻接矩阵（Adjacency Matrix）存储方式是用两个数组来表示图。一个一维数组存储图中顶点信息，一个二维数组（称为邻接矩阵）存储图中的边或弧的信息。

**邻接矩阵存储的结构**，代码如下：

```c
typedef char VertexType; /*顶点类型应由用户定义*/
typedef int EdgeType; /*边上权值类型应由用户定义*/
#define MAXVEX 100 /*最大顶点数，应由用户定义*/
#define INFINITY 65535 /*用65535来代表∞*/
typedef struct 
{
  VertexType vexs[MAXVEX]; /*顶点表*/
  EdgeType arc[MAXVEX][MAXVEX]; /*邻接矩阵，可看作边表*/
  int numVertexes, numEdges; /*图中当前的顶点数和边数*/
}MGraph;
```

**建立无向网图的邻接矩阵**，代码如下：

```c
void CreateMGraph(MGraph *G)
{
  int i, j, k, w;
  printf("输入顶点数和边数： \n");
  scanf("%d,%d", &G->numVertexes, &G->numEdges); /*输入顶点数和边数*/
  for(i = 0; i < G->numVertexes; i++) /*读入顶点信息，建立顶点表*/
    scanf(&G->vexs[i]);
  for(i = 0; i < G->numVertexes; i++) 
    for(j = 0; j < G->numVertexes; j++)
      G->arc[i][j] = INFINITY; /*邻接矩阵初始化*/
  for(k = 0; k < G->numEdges; k++) 
  {
    printf("输入边（vi,vj)上的下标i，下表j和权w：\n");
    scanf("%d, %d, %d", &i, &j, &w); /*输入边(vi,vj)上的权w*/
    G->arc[i][j]=w;
    G->arc[j][i]= G->arc[i][j] /*因为上无向图，矩阵对称*/
  }
}
```

### 邻接表

数组与链表相结合的存储方法称为邻接表（Adjacency List）

邻接表的处理办法是这样的

1. 图中顶点用一个一维数组存储，放入顶点也可以用单链表来存储，不过数组可以较容易地读取顶点信息，更加方便。另外，对于顶点数组中，每个数据元素还需要存储指向第一个邻接点的指针，以便于查找该顶点的边信息。
2. 图中每个顶点vi的所有邻接点构成一个线性表，由于邻接点的个数不定，所以用单链表存储，无向图称为顶点vi的边表，有向图则称为顶点vi作为弧尾的出边表。

顶点表的各个结点由data和firstedge两个域表示，data是数据域，存储顶点的信息，firstedge是指针域，指向边表的第一个结点，即此顶点的第一个邻接点。边表结点由adjvex和next两个域组成。adjvex是邻接点域，存储某顶点的邻接点在顶点表中的下标，next则存储指向边表中下一个结点的指针。比如v1顶点与v0，v2互为邻接点，则在v1的边表中，adjvex分别为v0的0和v2的2

若是有向图，邻接表结构是类似的。建立一个有向图的逆邻接表，即对每个顶点vi都建立一个链接vi为弧头的表。

对于带权值的网图，可以在边表结点定义中再增加一个weight的数据域，存储权值信息即可。

**结点定义**的代码如下：

```c
typedef char VertexType; /*顶点类型应由用户定义*/
typedef int EdgeType; /*边上的权值类型应由用户定义*/

typedef struct EdgeNode /*边表结点*/
{
  int adjvex; /*邻接点域，存储该顶点对应的下标*/
  EdgeType weight; /*用于存储权值，对于非网图可以不需要*/
  struct EdgeNode *next; /*链域，指向下一个邻接点*/
}EdgeNode;

typedef struct VertexNode /*顶点表结点*/
{
  VertexType data; /*顶点域，存储顶点信息*/
  EdgeNode *firstedge; /*边表头指针*/
}VertexNode, AdjList[MAXVEX];

typedef struct {
  AdjList adjList;
  int numVertexex, numEdges; /*图中当前顶点数和边数*/
}GraghAdjList;
```

**无向图邻接表的创建**，代码如下

```c
void CreateALGraph(GraphAdjList *G)
{
  int i, j, k;
  EdgeNode *e;
  printf("输入顶点数和边数：\n")
  scanf("%d,%d", &G->numVertexex, &G->numEdges); /*输入顶点数和边数*/
  for (i = 0; i < G->numVertexes; i++)
  {
    scanf(&G->adjList[i].data); /*输入顶点信息*/
    G->adjList[i].firstedge=NULL; /*将边表置为空表*/
  }
  for(k = 0;k < G->numEdges; k++)
  {
    printf("输入边（vi,vj）上的顶点序号：\n");
    scanf("%d,%d", &i, &j); /*输入边(vi,vj)上的顶点序号*/
    e = (EdgeNode *) malloc(sizeof(EdgeNode)); /*向内存申请空间，生成边表结点*/
    e -> adjvex=j; /*邻接序号为j*/
    e -> next=G->adjList[i].firstedge; /*将e指针指向当前顶点指向的结点*/
    G->adjList[i].firstedge=e; /*将当前顶点的指针指向e*/
    e = (EdgeNode *)malloc(sizeof(EdgeNode)); /*向内存申请空间，生成边表结点*/
    e->adjvex=i; /*邻接序号为i*/
    e->next=G->adjList[j].firstedge; /*将e指针指向当前顶点指向的结点*/
    G->adjList[j].firstedge=e; /*将当前顶点的指针指向e*/
  }
}
```

### 十字链表

把邻接表和逆邻接表结合起来。

顶点表结点结构中firstin表示入边表头指针，指向该顶点的入边表中第一个结点，firstout表示出边表头指针，指向该顶点的出边表中的第一个结点。

边表结点结构中tailvex是指弧起点在顶点表的下标，headvex是指弧重点在顶点表中的下标，headlink是指入边表指针域，指向终点相同的下一条边，taillink是指边表指针域，指向起点相同的下一条边

如果是网，还可以再增加一个weight域来存储权值。

### 邻接多重表

边表结点结构中ivex和jvex是与某条边依附的两个顶点在顶点表中下标。ilink指向依附顶点ivex的下一条边，jlink指向依附顶点jvex的下一条边，这就是邻接多重表结构

### 边集数组

边集数组是由两个一维数组构成。一个存储顶点的信息；另一个是存储边的信息，这个边数组每个数据元素由一条边的起点下标（begin）、终点下标（end）和权（weight）组成。



参考资料：

- [大话数据结构]()