---
title: LCA
published: 2023-07-18
description: ''
image: ''
tags: [图论,树上问题,LCA]
category: '图论'
draft: false
---

# 定义

LCA（Lowest Common Ancestor）指的是最近公共祖先。对于有根树的两个节点 $x,y$，它们的最近公共祖先 $\text{LCA}(x,y)$ 表示 $x,y$ 的公共祖先中深度最大的那个。

# 性质

1. $\text{LCA}(\{x\})=x$；
2. $x$ 是 $y$ 的祖先，当且仅当 $\text{LCA}(x,y)=x$；
3. 如果 $x$ 不为 $y$ 的祖先并且 $y$ 不为 $x$ 的祖先，那么 $x,y$ 分别处于 $\text{LCA}(x,y)$ 的两棵不同子树中；
4. 前序遍历中，$\text{LCA}(S)$ 出现在所有 $S$ 中元素之前，后序遍历中 $\text{LCA}(S)$ 则出现在所有 $S$ 中元素之后；
5. 两点集并的最近公共祖先为两点集分别的最近公共祖先的最近公共祖先，即 $\text{LCA}(A\cup B)=\text{LCA}(\text{LCA}(A), \text{LCA}(B))$；
6. 两点的最近公共祖先必定处在树上两点间的最短路上；
7. $d(x,y)=h(x)+h(y)-2h(\text{LCA}(x,y))$，其中 $d$ 是树上两点间的距离，$h$ 代表某点到树根的距离。

# 求法

## 朴素 LCA

- 对于无根树来说，可以建立双向边，并选一个点作根进行搜索。
- 从根开始搜，对于一个节点 $x$，记录节点的深度标记 $deep(x)$ 和父亲节点的编号 $fa(x)$。
- 对于 $q$ 个询问 $x,y$ 的 LCA，每次调用一次 $\text{LCA}(x,y)$。

### 步骤

1. 如果 $x$ 的深度小于 $y$ 的深度，交换 $x,y$。
2. 循环减少 $x$ 的深度，直到 $x,y$ 的深度相同。
3. 循环同时减少 $x,y$ 的深度，直到 $x$ 等于 $y$ 时算法停止，此时的 $x$ 或 $y$ 即为所求。

### 代码

```cpp
void dfs(int x,int fath) {
    fa[x]=fath;// 记录 x 的父亲
    dep[x]=dep[fath]+1;//记录 x 的深度
    for(int i=head[x];i;i=nxt[i]) {
        int y=to[i];
        if(y!=fath) dfs(y,x);//深搜 x 的子节点 y
    }
}
int LCA(int x,int y) {
    if(dep[x]<dep[y])//如果 x 比 y 浅就交换 x,y
        swap(x,y);
    while(dep[x]>dep[y])//x 往上跳直到和 y 深度相同
        x=fa[x];
    while(x!=y) {//x,y 同时往上跳直到 x,y 相同
        x=fa[x];
        y=fa[y];
    }
    return x;//返回 x
}
```

- `fa[x]` 表示 $x$ 的父亲节点。
- `dep[x]` 表示 $x$ 的深度。

### 注意事项

- 要保证 $x$ 比 $y$ 深。

## 倍增 LCA

可以 $O(\log n)$ 地单次查询两点的 LCA。

定义 $fa(i,j)$ 表示 $i$ 的 $2^j$ 辈祖先，那么就有 $fa(i,j)=fa(fa(i,j-1),j-1)$。

这样就可以在 $O(n\log n)$ 的时间复杂度内预处理出每个节点 $x$ 的 $2^k\,(k\leq\log deep(x))$ 辈祖先。

其实就是把朴素 LCA 的小步走变成了大步跳，算是朴素 LCA 的一种优化，本质上是没有区别的。

### 代码

```cpp
void dfs(int x,int fath) {
    fa[x][0]=fath;
    dep[x]=dep[fath]+1;
    for(int i=1;i<=lg[dep[x]];i++)//递推 x 的 2^i 辈父亲
        fa[x][i]=fa[fa[x][i-1]][i-1];
    for(int i=head[x];i;i=nxt[i]) {
        int y=to[i];
        if(y!=fath) dfs(y,x);
    }
}
int LCA(int x,int y) {
    if(dep[x]<dep[y]) swap(x,y);
    while(dep[x]>dep[y])//x 大步跳
        x=fa[x][lg[dep[x]-dep[y]]-1];
    if(x==y) return x;//这行很重要，返回的是 x
    for(int i=lg[dep[x]];i>=0;i--)
        if(fa[x][i]!=fa[y][i]) {//x,y 大步跳
            x=fa[x][i];
            y=fa[y][i];
        }
    return fa[x][0];//返回 x 的父亲
}
```

- `lg[i]` 表示 $\log_2i$。
- `fa[x][i]` 表示 $x$ 的 $2^i$ 辈父亲。
- `dep[x]` 表示 $x$ 的深度。

### 注意事项

- 中间要判断 $x,y$ 是否已经相等，如果相等要返回 $x$ 或 $y$。
- 最后返回的是 $x$ 或 $y$ 的父亲

## 离线 Tarjan 求 LCA

- 利用并查集优越的时空复杂度，可以在近乎 $O(1)$ 地完成 LCA 的单次查询。
- Tarjan 算法基于深度优先搜索的框架，在一次遍历中把所有询问一次性解决。
- 也就是说我们一边搜索节点，一边处理询问。要求我们把所有的询问都离散化到每一个节点中。
- 所谓离线就是会把所有的询问都提前给出，反之在线就是指处理完当前询问后才知道下一个询问是什么。

### 步骤

1. 对于新搜到的一个节点 $x$，对 $x$ 的每一个子树进行搜索。
2. 当某一棵子树搜索完后，把子树所形成的集合与当前节点 $x$ 用并查集合并，并将 $x$ 设为这个集合的祖先，直到 $x$ 的所有子树搜索完。
3. 标记 $x$ 节点的搜索标记 $vis_x=1$，并且把 $x$ 所在的集合与 $fa_x$ 所在的集合合并，根为 $fa_x$。
4. 接下来处理有关 $x$ 的询问，对于询问 $\text{LCA}(x,y)$：
    - 如果 $y$ 节点已经被访问过了，即 $vis_y=1$，那么 $\text{LCA}(x,y)=find(y)$，其中 $find$ 表示某集合的根。
    - 如果 $y$ 节点还没有被访问过，那么此次询问的答案将会在 $y$ 节点被访问过后被计算。
5. 如果有一个关于 $x,y$ 的询问，且 $y$ 已被标记，则由于进行的是深度优先搜索，假设 $x,y$ 的最近公共祖先为 $z$，而 $z$ 的包含 $y$ 的子树一定已经搜索过了，那么 $z$ 一定是 $y$ 所在并查集的根。

### 代码

```cpp
void tarjan(int x) {
    for(int i=head[x];i;i=nxt[i]) {
        int y=to[i];
        if(!v[y]) {
            tarjan(y);//深搜 y
            fa[y]=x;//将 y 所在的集合合并
        }
    }
    v[x]=1;//搜完所有子树后再标记
    for(int i=qhead[x];i;i=qnxt[i]) {//处理关于 x 的询问
        int y=qto[i];
        if(v[y]) ans[id[i]]=find(y);//记录答案
    }
}
```

- `v[x]` 表示 $x$ 是否已搜索完。
- `fa[x]` 表示 $x$ 在并查集中的父亲节点。
- `ans[i]` 表示对于第 $i$ 条询问的答案。
- `id[i]` 表示第 $i$ 条询问边对应的询问编号。
- `find(x)` 表示 $x$ 在并查集中的根。

### 注意事项

- 搜索完 $x$ 的所有子树时再标记 $x$。
- 在将询问离散化到节点上时要建双向边。