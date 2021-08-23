---
layout: post
title: "大话数据结构：最小生成树"
date:  2020-06-23 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 最小生成树

构造连通网的最小代价生成树称为最小生成树（Minimum Cost Spanning Tree）

### 普里姆（Prim）算法

左侧数字为行号，其中INFINITY为权值极大值，不妨是65535，MAXVEX为顶点个数最大值。

```c
/*Prim算法生成最小生成树*/
void MiniSpanTree_Prim(MGraph G)
{
  int min, i, j, k;
  int adjvex[MAXVEX]; /*保存相关顶点下标*/
	int lowcost[MAXVEX]; /*保存相关顶点间边的权值*/
  lowcost[0] = 0; /*初始化第一个权值为0，即v0加入生成树，lowcost的值为0，在这里就是此下表的顶点已经加入生成树*/
  adjvex[0] = 0; /*初始化第一个顶点下标为0*/
  for(i = 1; i < G.numVertexes; i++) /*循环除下标为0外的全部顶点*/
  {
    lowcost[i] = G.art[0][i]; /*将v0顶点与之有边的权值存入数组*/
    adjvex[i] = 0; /*初始化都为v0的下标*/
  }
 	for(i = 1; i < G.numVertexes; i++)
  {
    min = INFINITY; /*初始化最小权值为∞，通常设施为不可能的大数字如32767、65535等*/
    j = 1; k = 0;
    while(j < G.numVertexes) /*循环全部顶点*/
    {
      if(lowcost[j] != 0 && lowcost[j] < min) /*如果权值不为0且权值小于min*/
      {
        min = lowcost[j]; /*则让当前权值成为最小值*/
        k = j; /*将当前最小值的下标存入k*/
      }
      j++;
    }
    printf("(%d, %d)", adjvex[k], k); /*打印当前顶点边中权值最小边*/
    lowcost[k] = 0; /*将当前顶点的权值设置为0，表示此顶点已经完成任务*/
    for(j = 1; j < G.numVertexes; j++) /*循环所有顶点*/
    {
      if(lowcost[j] != 0 && G.arc[k][j] < lowcost[j]) /*若下标为k顶点各边权值小于此前这些顶点未被加入生成树权值*/
      {
        lowcost[j] = G.arc[k][j]; /*将较小权值存入lowcost*/
        adjvex[j] = k; /*将下标为k的顶点存入adjvex  */
      }
    }
  }
}
```

假设N=(P,{E})是连通网，TE是N上最小生成树中边的集合。算法从U={u0}（u0∊V）,TE={}开始。重复执行下述操作：在所有u∊U, v∊V-U的边(u,v)∊E中找一条代价最小的边(u0,v0)并入集合TE，同时v0并入U，直至U=V为止。此时TE中必有n-1条边，则T=(V,{TE})为N的最小生成树。

### 克鲁斯卡尔（Kruskal）算法

edge边集数组结构定义代码：

```c
typedef struct 
{
  int begin;
  int end;
  int weight;
}Edge;
```

在克鲁斯卡尔（Kruskal）算法代码中，MAXEDGE为边数量的极大值，MAXVEX为顶点个数最大值。

```c
void MiniSpanTree_Kruskal(MGragh G) /*生成最小生成树*/
{
  int i, n, m;
  Edge edges[MAXEDGE]; /*定义边集数组*/
  int parent[MAXVEX]; /*定义一数组用来判断边与边是否形成回路 此处省略将邻接矩阵G转化为边集数组edges并按权由小到大排序的代码*/
  for(i = 0; i < G.numVertexes; i++)
    parent[i] = 0; /*初始化数组值为0*/
  for(i = 0; i < G.numEdges; i++)  /*循环每一条边*/
  {
    n = Find(parent, edges[i].begin);
    m = Find(parent, edges[i].end);
    if (n != m) /*假如n与m不等，说明此边没有现成生成树形成环路*/
    {
      parent[n] = m; /*将此边的结尾顶点放入下标为起点的parent中，表示此顶点已经在生成树集合中*/
      printf("(%d, %d) %d", edges[i].begin, edges[i].end, edges[i].weight);
    }
  }
}

int Find(int *parent, int f) /*查找连线顶点的尾部下标*/
{
  while(parent[f] > 0)
    f = parent[f];
  return f;
}
```

假设N=(V,{E})是连通网，则令最小生成树的初始状态为只有n个顶点而无边的非连通图T={V, {}}，图中每个顶点自成一个连通分量。在E中选择代价最小的边，若该边依附的顶点落在T中不同的连通分量上，则将此边加入到T中，否则舍去此边而选择下一条代价最小的边。一次类推，直至T中所有顶点都在同一连通分量上为止。


参考资料：

- [大话数据结构]()