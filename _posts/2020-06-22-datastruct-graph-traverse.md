---
layout: post
title: "大话数据结构：图的遍历"
date:  2020-06-22 10:01:08
author: "Viletyy"
header-style: text
tags:
  - 数据结构
---
## 图的遍历

从图中某一顶点出发访遍图中其余顶点，且使每一个顶点仅被访问一次，这一过程就叫做图的遍历（Traversing Graph）

### 深度优先遍历

深度优先遍历（Depth_First_Search），也有称为深度优先搜索，简称为DFS

它从图中某个顶点v出发，访问此顶点，然后从v的未被访问的邻接点出发深度优先遍历图，直至图中所有和v有路径相通的顶点都被访问到。

若图中尚有顶点未被访问，则另选图中一个未曾被访问到顶点作起始点，重复上述过程，直至图中所有顶点都被访问到为止。

邻接矩阵的深度优先递归算法：

```c
typedef int Boolean; /*Boolean是布尔类型，其值是TRUE或FALSE*/
Boolean visited[MAX]; /*访问标志的数组*/
void DFS(MGraph G, int i)
{
  int j;
  visited[i] = TRUE;
  printf("%c ", G.vexs[i]); /*打印顶点，也可以进行其他操作*/
  for(j = 0; j < G.numVertexes; j++)
    if(G.arc[i][j] == 1 && !visited[j])
      DFS(G, j); /*对为访问的邻接顶点递归调用*/
}

/*邻接矩阵的深度遍历操作*/
void DFSTraverse(MGraph G)
{
  int i;
  for(i = 0; i < G.numVertexes; i++)
    visited[i] = FALSE; /*初始所有顶点状态都是未访问过状态*/
  for(i = 0; i < G.numVertexes; i++)
    if(!visited[i]) /*对未访问过的顶点调用DFS，若是连通图，只会执行一次*/
      DFS(G, i);
}
```

邻接表的深度优先递归算法：

```c
void DFS(GraphAdjList GL, int i)
{
  EdgeNode *p;
  visited[i] = TRUE;
  printf("%c ", GL->adjList[i].data); /*打印顶点，也可以其他操作*/
  p = GL->adjList[i].firstedge;
  while(p)
  {
    if(!visited[p->adjvex])
      DFS(GL, p->adjvex); /*对为未访问的邻接顶点递归调用*/
    p = p->next;
  }
}
/*邻接表的深度遍历操作*/
void DFSTraverse(GraphAdjList GL)
{
  int i;
  for(i = 0; i < GL->numVertexes; i++)
    visited[i] = FALSE; /*初始所有顶点状态都是未访问过状态*/
  for(i = 0; i < GL->numVertexes; i++)
    if(!visited[i]) /*对未访问过的顶点调用DFS，若是连通图，只会执行一次*/
      DFS(GL, i);
}
```

### 广度优先遍历

广度优先遍历（Breadth_First_Search）,又称为广度优先搜索，简称BFS

邻接矩阵的广度遍历算法：

```c
void BFSTraverse(MGraph G)
{
  int i, j;
  Queue Q;
  for(i = 0; i < G.numVertexes; i++)
    visited[i] = FALSE;
  InitQueue(&Q); /*初始化一辅助用的队列*/
  for(i = 0; i < G.numVertexex; i++) 
  {
    if (!visited[i]) /*若是未访问过就处理*/
    {
      visited[i]=TURE; /*设置当前顶点访问过*/
      printf("%c ", G.vexs[i]); /*打印顶点，也可以进行其他操作*/
      EnQueue(&Q, i); /*将此顶点入队列*/
      while(!QueueEmpty(Q)) /*若当前队列不为空*/
      {
        DeQueue(&Q, &i); /*将队中元素出队列，赋值给i*/
        for(j = 0; j < G.numVertexes; j++)
        {
          /*判断其他顶点若与当前顶点存在边且未访问过*/
          if(G.art[i][j] == 1 &&. !visited[j])
          {
            visited[j] = TRUE; /*将找到的此顶点标记为已访问*/
            printf("%c ", G.vexs[j]); /*打印顶点*/
            EnQueue(&Q, j); /*将找到的此顶点入队列*/
          }
        }
      }
    }
  }
}
```

邻接表的广度遍历算法

```c
void BFSTraverse(GraphAdjList GL)
{
  int i;
  EdgeNode *p;
  Queue Q;
  for(i = 0; i < GL->numVertexes; i++)
    visited[i] = FALSE;
  InitQueue(&Q);
  for(i = 0; i < GL->numVertexes; i++)
  {
    if (!visited[i])
    {
      visited[i] = TRUE;
      printf("%c ", GL->adjList[i].data); /*打印顶点，也可以进行其他操作*/
      EnQueue(&Q, i);
      while(!QueueEmpty(Q))
      {
        DeQueue(&Q, &i);
        p = GL->adjList[i].firstedge; /*找到当前顶点边表链表头指针*/
        while(p)
        {
          if(!visited[p->adjvex]) /*若此顶点未被访问*/
          {
          	visited[p->adjvex]=TRUE;
            printf("%c ",GL->adjList[p->adjvex].data);
            EnQueue(&Q, p->adjvex); /*将此顶点入队列*/
          }
          p = p->next; /*指针指向下一个邻接点*/
        }
      }
    }
  }
}
```


参考资料：

- [大话数据结构]()