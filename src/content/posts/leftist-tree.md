---
title: 左偏树
published: 2024-07-08
description: ''
image: ''
tags: [数据结构,堆,左偏树]
category: '数据结构'
draft: false
---

# 简介

左偏树是一种可并堆，即可以快速合并的堆。

# 定义

对于一棵二叉树，定义 **外节点** 为子结点小于两个的节点。

定义一个节点的 $dist$ 为其到子树中最近的外节点所经过的边的数量，外节点的 $dist$ 为 $0$。

左偏树是一棵二叉树，它不仅具有对的性质，并且每个节点左儿子的 $dist$ 都大于等于右儿子的 $dist$。

左偏树每个节点的 $dist$ 都等于其右儿子的 $dist$ 加一。

# 操作

为了方便，本文操作均讨论大根堆。

## 合并 merge

合并是左偏树的的核心操作。

合并两个堆时，由于要满足堆性质，先取值较大的那个根作为合并后堆的根节点。

然后将根的左儿子作为合并后堆的左儿子，递归合并其右儿子与另一个堆，作为合并后的堆的右儿子。

为了满足左偏树的性质，若合并完后左儿子的 $dist$ 小于右儿子的 $dist$，就交换左右儿子。

```cpp
void merge(int x,int y) {
	if(!x||!y) return x+y;//若一个堆为空，返回另一个堆
	if(w[x]<w[y]) swap(x,y);
	rson[x]=merge(rson[x],y);
	if(dist[lson[x]]<dist[rson[x]])
		swap(lson[x],rson[x]);
	dist[x]=dist[rson[x]]+1;//更新 dist
	return x;
}
```

由于左偏性质，每递归一层，其中一个堆根节点的 $dist$ 就会减小 $1$，而一棵有 $n$ 个节点的二叉树，根的 $dist$ 不超过 $\left\lceil\log (n+1)\right\rceil$，所以合并两个大小分别为 $n$ 和 $m$ 的堆复杂度是 $O(\log n+\log m)$。

## 插入节点 insert

单个节点也可以视为一个堆，合并即可。

## 删除根 pop

合并根的左右儿子即可。

## 删除节点 erase

先将左右儿子合并，然后自底向上更新 $dist$，不满足左偏性质时交换左右儿子，当 $dist$ 无需更新时结束递归。

```cpp
void pushup(int x) {
	if(!x) return;
	if(dist[x]!=dist[rson[x]]+1) {
		dist[x]=dist[rson[x]]+1
		pushup(fa[x]);
	}
}
void erase(int x) {
	int y=merge(lson[x],rson[x]);
	fa[y]=fa[x];
	if(lson[fa[x]]==x)
		lson[fa[x]]=y;
	else rson[fa[x]]=y;
	pushup(fa[y]);
}
```