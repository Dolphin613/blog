---
title: 网络流-最大流/最小割
published: 2024-07-19
description: ''
image: ''
tags: [图论,网络流]
category: '图论'
draft: false
---

# 网络流基本概念

网络是指一个特殊的有向图 $G=(V,E)$，其与一般有向图的不同之处在于有容量和源汇点。

- 定义容量函数 $c:V\times V\to \R^+$。对于边 $(u,v)\in E$，其容量为给定值。对于边 $(u,v)\notin E$，可以认为 $c(u,v)=0$。
- $V$ 中有两个特殊的点：源点 $s$ 和汇点 $t$。

对于网络 $G=(V,E)$，定义流函数 $f:V\times V\to \R^+$，其满足以下性质：

1. 容量限制：对于每条边 $(u,v)$，流经该边的流量不得超过该边的容量，即 $0\leq f(u,v)\leq c(u,v)$；
2. 流量守恒：定义节点 $u$ 的净流量 $F(u)=\sum_{v\in V}f(u,v)-\sum_{v\in V}f(v,u)$。对于除源汇点外的任意节点 $u$，满足其净流量 $F(u)=0$，即流出 $u$ 的流量等于流入 $u$ 的流量。

对于网络 $G=(V,E)$ 和其上的流 $f$，我们定义 $f$ 的流量 $|f|$ 为 $s$ 的净流量 $F(s)$。作为流量守恒的推论，这也等于 $t$ 的净流量的相反数 $-F(t)$。

对于网络 $G=(V,E)$，如果 $\{S, T\}$ 是 $V$ 的一个划分，即 $S\cup T=V$ 且 $S\cap T=\varnothing$，且满足 $s\in S,t\in T$，则我们称 $\{S, T\}$ 是 $G$ 的一个 $s$ - $t$ 割。我们定义 $s$ - $t$ 割 $\{S, T\}$ 的容量为 $C(S,T)=\sum_{u\in S}\sum_{v\in T}c(u,v)$，即所有满足 $u\in S,v\in T$ 的边 $(u,v)$ 的容量之和。

# 定义

最大流问题：令 $G=(V,E)$ 是一个有源汇点的网络，我们希望在 $G$ 上指定合适的流 $f$，以最大化整个网络的流量 $|f|$。

最小割问题：令 $G=(V,E)$ 是一个有源汇点的网络，我们希望求 $G$ 上的一个割 $\{S,T\}$，使得割的容量 $C(S,T)$ 最小。

# 求法

见 [OI-wiki](https://oi-wiki.org/graph/flow/max-flow/)。

<!-- ## 最大流最小割定理

**对于任意网络 $G=(V,E)$，其上的最大流 $f$ 和最小割 $\{S,T\}$ 总是满足 $|f|=c(S,T)$。**

为了证明最大流最小割定理，我们先从一个引理出发：对于网络 $G=(V,E)$，任取一个流 $f$ 和割 $\{S,T\}$，总是有 $f\leq c(S,T)$，其中等号成立当且仅当 $\{(u,v)|u\in S,v\in T\}$ 的所有边均满流，且 $\{(u,v)|u\in T,v\in S\}$ 的所有边均空流。

引理证明：

$$
\begin{aligned}
|f|&=f(s)\\
&=\sum_{u\in S}f(u)\\
&=\sum_{u\in S}\left(\sum_{v\in V}f(u,v)-\sum_{v\in V}f(v,u)\right)\\
&=\sum_{u\in S}\left(\sum_{v\in T}f(u,v)+\sum_{v\in S}f(u,v)-\sum_{v\in T}f(v,u)-\sum_{v\in S}f(v,u)\right)
&=\sum_{u\in S}\left(\sum_{v\in T}f(u,v)-\sum_{v\in T}f(v,u)\right)+\sum_{u\in S}\sum_{v\in S}f(u,v)-\sum_{u\in S}\sum_{v\in S}f(v,u)\\
&=\sum_{u\in S}\left(\sum_{v\in T}f(u,v)-\sum_{v\in T}f(v,u)\right)\\
&\leq\sum_{u\in S}\sum_{v\in T}f(u,v)\\
&\leq\sum_{u\in S}\sum_{v\in T}c(u,v)\\
&=c(S,T)
\end{aligned}
$$

为了取等，第一个不等号需要 $\{(u,v)|u\in T,v\in S\}$ 的所有边均空流，第二个不等号需要 $\{(u,v)|u\in S,v\in T\}$ 的所有边均满流。原引理得证。

那么，对于任意网络，以上取等条件是否总是能被满足呢？如果答案是肯定的，则最大流最小割定理得证。以下我们尝试证明。 -->

<!-- ```cpp
struct net{
	int maxflow,dep[N],vis[N];
	int tot,head[N],cur[N],to[M],nxt[M],cap[M],flow[M];
	void clear() {
		tot=1;
		memset(head,0,sizeof head);
	}
	void _add(int x,int y,int z) {
		to[++tot]=y;
		cap[tot]=z;
		flow[tot]=0;
		nxt[tot]=head[x];
		head[x]=tot;
	}
	void add(int x,int y,int z) {
		_add(x,y,z),_add(y,x,0);
	}
	bool bfs(int s,int t) {
		memset(dep,0,sizeof dep);
		queue<int>q;
		q.push(s),dep[s]=1;
		while(!q.empty()) {
			int x=q.front();
			q.pop();
			for(int i=head[x];i;i=nxt[i]) {
				int y=to[i];
				if(!dep[y]&&cap[i]>flow[i]) {
					dep[y]=dep[x]+1;
					q.push(y);
				}
			}
		}
		return dep[t];
	}
	int dfs(int x,int t,int now) {
		if(x==t) return now;
		int ret=0;
		vis[x]=1;
		for(int i=cur[x];i&&now>ret;i=nxt[i]) {
			cur[x]=i;
			int y=to[i];
			if(!vis[y]&&dep[y]==dep[x]+1&&cap[i]>flow[i]) {
				int d=dfs(y,t,min(now-ret,cap[i]-flow[i]));
				ret+=d,flow[i]+=d,flow[i^1]-=d;
			}
		}
		vis[x]=0;
		return ret;
	}
	void dinic(int s,int t) {
		maxflow=0;
		while(bfs(s,t)) {
			memcpy(cur,head,sizeof cur);
			maxflow+=dfs(s,t,Inf);
		}
	}
};
``` -->

# 应用

## 最大流

### [洛谷P4311 士兵占领](https://www.luogu.com.cn/problem/P4311)

> 有一个 $M\times N$ 的棋盘，有的格子是障碍。现在你要选择一些格子来放置一些士兵，一个格子里最多可以放置一个士兵，障碍格里不能放置士兵。我们称这些士兵占领了整个棋盘当满足第 $i$ 行至少放置了 $L_i$ 个士兵，第 $j$ 列至少放置了 $C_j$ 个士兵。现在你的任务是要求使用最少个数的士兵来占领整个棋盘。若无解则输出 $\tt{JIONG!}$。
>
> $1\leq M,N\leq 100$。

首先考虑如果士兵只能给一行或一列造成贡献的答案是 $\sum_{i=1}^ml_i+\sum_{i=1}^nc_i$。但是发现有的士兵可以同时给一列和一行造成贡献，那就算出这些士兵的最大个数就行了。

每一行和每一列都建立一个点。源点 $s$ 向每一行 $i$ 连一条容量为的 $l_i$ 边，每一列 $j$ 向汇点 $t$ 连一条容量为的 $c_j$ 边。若第 $i$ 行第 $j$ 列不是障碍，则将第 $i$ 行向第 $j$ 列连一条容量为 $1$ 的边。

跑出来的最大流就是贡献为 $2$ 的士兵的最大数量，注意特判无解情况。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=210,M=40010,Inf=0x3f3f3f3f;
int n,m,k,ans,c[N],l[N],p[N][N],sum[N];
struct net{...};
net g;
int main() {
	g.clear();
	scanf("%d%d%d",&m,&n,&k);
	int s=m+n+1,t=m+n+2;
	for(int i=1;i<=m;i++) {
		scanf("%d",&l[i]);
		ans+=l[i];
		g.add(s,i,l[i]);
	}
	for(int i=1;i<=n;i++) {
		scanf("%d",&c[i]);
		ans+=c[i];
		g.add(i+m,t,c[i]);
	}
	for(int i=1;i<=k;i++) {
		int x,y;
		scanf("%d%d",&x,&y);
		p[x][y]=1;
	}
	for(int i=1;i<=m;i++)
		for(int j=1;j<=n;j++)
			if(!p[i][j]) {
				g.add(i,j+m,1);
				sum[i]++,sum[j+m]++;
			}
	for(int i=1;i<=m;i++)
		if(sum[i]<l[i]) {
			printf("JIONG!");
			return 0;
		}
	for(int i=1;i<=n;i++)
		if(sum[i+m]<c[i]) {
			printf("JIONG!");
			return 0;
		}
	g.dinic(s,t);
	printf("%d",ans-g.maxflow);
	return 0;
}
```

### [洛谷P2891 USACO07OPEN Dining G](https://www.luogu.com.cn/problem/P2891)

> 有 $F$ 种食物和 $D$ 种饮料，每种食物或饮料只能供一头牛享用，且每头牛只享用一种食物和一种饮料。现在有 $n$ 头牛，每头牛都有自己喜欢的食物种类列表和饮料种类列表，问最多能使几头牛同时享用到自己喜欢的食物和饮料。
>
> $1\leq n,F,D\leq 100$。

容易想到先将源点 $s$ 向每种食物连边，每种饮料向汇点 $t$ 连边，并将牛喜欢的食物向牛连边，牛向喜欢的饮料连边，边的容量都为 $1$。

但是这样无法限制牛只能选一种食物和一种饮料。我们可以拆点来限制通过点的流量，将点拆成入点和出点，入点向出点连一条容量为 $1$ 的边。将连向原来的点的边连向入点，从原来的点连出的边改为从出点连出。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=410,M=50010,Inf=0x3f3f3f3f;
int n,f,d;
struct net{...};
net g;
int main() {
	g.clear();
	scanf("%d%d%d",&n,&f,&d);
	int s=f+d+2*n+1,t=s+1;
	for(int i=1;i<=f;i++) g.add(s,i,1);
	for(int i=1;i<=d;i++) g.add(i+f,t,1);
	for(int i=1;i<=n;i++) {
		int a,b,x;
		int in=f+d+i,out=n+in;
		g.add(in,out,1);
		scanf("%d%d",&a,&b);
		for(int j=1;j<=a;j++) {
			scanf("%d",&x);
			g.add(x,in,1);
		}
		for(int j=1;j<=b;j++) {
			scanf("%d",&x);
			g.add(out,x+f,1);
		}
	}
	g.dinic(s,t);
	printf("%d",g.maxflow);
	return 0;
}
```

### [UVA10779 Collectors Problem](https://www.luogu.com.cn/problem/UVA10779)

> Bob 和他的朋友从糖果包装里收集贴纸。这些朋友每人手里都有一些（可能有重复的）贴纸，并且只跟别人交换他所没有的贴纸，贴纸总是一对一交换。
>
> Bob 比这些朋友更聪明，因为他意识到只跟别人交换自己没有的贴纸并不总是最优的。在某些情况下，换来一张重复的贴纸更划算。
>
> 假设 Bob 的朋友只和 Bob 交换（他们之间不交换），并且这些朋友只会出让手里的重复贴纸来交换他们没有的不同贴纸。你的任务是帮助 Bob 算出他最终可以得到的不同贴纸的最大数量。
>
> $2\leq n\leq 10$，$5\leq m\leq 25$，其中 $n$ 表示人数，$m$ 表示贴纸的种类数。

Bob 的朋友只会出让手里的重复贴纸来交换他们没有的不同贴纸。所以，对于 Bob 的某个朋友 Friend，Bob 只能把一种 Friend 没有的贴纸给他，并且一种最多给一次。Friend 只会把他手里重复的贴纸给 Bob，如果 Friend 有 $i(i\geq 2)$ 张某种贴纸，他至多给 Bob $i-1$ 张。

那么，Bob 的朋友的作用是将 Bob 的贴纸 $X$ 变成另一种贴纸 $Y$。

对每种贴纸 $i$ 建立点 $A_i$。源点 $s$ 向 $A_i$ 连边，容量为 Bob 拥有的贴纸 $i$ 的数量。$A_i$ 向汇点 $t$ 连边，容量为 $1$。

对 Bob 的每个朋友 $j$ 建立点 $B_j$。若朋友 $j$ 没有贴纸 $i$，就从 $A_i$ 向 $B_j$ 连边，容量为 $1$。若朋友 $j$ 有 $k(k\geq 2)$ 张贴纸 $i$，就从 $B_j$ 向 $A_i$ 连边，容量为 $k-1$。

我们用 $A$ 表示点 $A_i$ 的集合，用 $B$ 表示点 $B_i$ 的集合。

一条从 $s$ 到 $t$ 的流，是先从 $s$ 到 $A$，表示 Bob 最初拥有的某种贴纸。然后经过若干次（可能是 $0$ 次）到 $B$ 再到 $A$ 的过程，表示和朋友进行了交换。最后从 $A$ 到 $t$，表示交换结束后 Bob 手中的贴纸的种类。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=110,M=10010,Inf=0x3f3f3f3f;
int T,n,m,cnt[N];
struct net{...};
net g;
int main() {
	scanf("%d",&T);
	for(int loop=1;loop<=T;loop++) {
		g.clear();
		scanf("%d%d",&n,&m);
		int k,x,s=n+m,t=s+1;
		for(int i=1;i<=m;i++)
			g.add(i,t,1);
		memset(cnt,0,sizeof cnt);
		scanf("%d",&k);
		for(int i=1;i<=k;i++) {
			scanf("%d",&x);
			cnt[x]++;
		}
		for(int i=1;i<=m;i++)
			if(cnt[i]) g.add(s,i,cnt[i]);
		for(int j=1;j<=n-1;j++) {
			memset(cnt,0,sizeof cnt);
			scanf("%d",&k);
			for(int i=1;i<=k;i++) {
				scanf("%d",&x);
				cnt[x]++;
			}
			for(int i=1;i<=m;i++) {
				if(!cnt[i])
					g.add(i,j+m,1);
				else if(cnt[i]>1)
					g.add(j+m,i,cnt[i]-1);
			}
		}
		g.dinic(s,t);
		printf("Case #%d: %d\n",loop,g.maxflow);
	}
	return 0;
}
```

## 最小割

### [洛谷P1344 USACO4.4 追查坏牛奶 Pollutant Control](https://www.luogu.com.cn/problem/P1344)

> 你第一天接手三鹿牛奶公司就发生了一件倒霉的事情：公司不小心发送了一批有三聚氰胺的牛奶。
>
> 很不幸，你发现这件事的时候，有三聚氰胺的牛奶已经进入了送货网。这个送货网很大，而且关系复杂。你知道这批牛奶要发给哪个零售商，但是要把这批牛奶送到他手中有许多种途径。
>
> 送货网由一些仓库和运输卡车组成，每辆卡车都在各自固定的两个仓库之间单向运输牛奶。在追查这些有三聚氰胺的牛奶的时候，有必要保证它不被送到零售商手里，所以必须使某些运输卡车停止运输，但是停止每辆卡车都会有一定的经济损失。
> 
> 你的任务是，在保证坏牛奶不送到零售商的前提下，制定出停止卡车运输的方案，使损失最小。输出最小的损失和在损失最小的前提下，最少要停止的卡车数。
>
> $2\leq N\leq 32$，$0\leq M\leq 10^3$，$N$ 表示仓库的数目，$M$ 表示运输卡车的数量。仓库 $1$ 代表发货工厂，仓库 $N$ 代表有三聚氰胺的牛奶要发往的零售商。

第一问即为求最小割，但是难点在于第二问，如何才能求出边数最小的最小割呢？

我们可以在建边的时候将每条边的容量乘上一个大于边数的常量 $k$ 再加一。此时跑出来的最小割的容量 $c(S,T)$ 即为原来最小割容量的 $k$ 倍加上最小割的边数。而且因为最小割的定义，这个最小割的边数一定最小。故第一问的答案为 $c(S,T)/k$，第二问的答案为 $c(S,T)\mod k$。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=40,M=2010;
const long long Inf=0x3f3f3f3f3f3f3f3f,mod=1001;
struct net{...};
net g;
int n,m;
int main() {
	g.clear();
	scanf("%d%d",&n,&m);
	for(int i=1;i<=m;i++) {
		int x,y,z;
		scanf("%d%d%d",&x,&y,&z);
		g.add(x,y,z*mod+1);
	}
	g.dinic(1,n);
	printf("%lld %lld",g.maxflow/mod,g.maxflow%mod);
	return 0;
}
```

### [洛谷P2774 方格取数问题](https://www.luogu.com.cn/problem/P2774)

> 有一个 $m$ 行 $n$ 列的方格图，每个方格中都有一个正整数。现要从方格中取数，使任意两个数所在方格没有公共边，且取出的数的总和最大，请求出最大的和。
>
> $1\leq n,m\leq 100$。

限制条件是如果要取某一个方格，那么禁止取相邻的四个方格，不限制其它所有方格。

所以猜测，从禁止的角度考虑才会更高效。也就是说，先选中所有方格，再想办法删去权值和尽量小的一批方格。

将方格图黑白染色，横纵坐标和的奇偶性相同的两个点肯定不互斥。把互斥的点连边的话，会形成一个二分图。

源点连向二分图的一个点集，边权为方格中的数。割一条边表示不取这个方格。

二分图的另一个点集连向汇点，边权还是方格中的数。割边也表示不取此点。

二分图内部的边，连接着互斥的点。边权全部赋为无穷大，以保证在最小割中不被割。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=10010,M=1e5+10,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t,sum;
int dx[]={0,0,1,-1},dy[]={1,-1,0,0};
int main() {
	g.clear();
	scanf("%d%d",&m,&n);
	s=n*m+1,t=s+1;
	for(int i=1;i<=m;i++)
		for(int j=1;j<=n;j++) {
			int x,id=(i-1)*n+j;
			scanf("%d",&x);
			sum+=x;
			if((i+j)&1) {
				g.add(s,id,x);
				for(int k=0;k<=3;k++) {
					int tx=i+dx[k],ty=j+dy[k];
					if(tx>=1&&tx<=m&&ty>=1&&ty<=n)
						g.add(id,(tx-1)*n+ty,Inf);
				}
			}
			else g.add(id,t,x);
		}
	g.dinic(s,t);
	printf("%d",sum-g.maxflow);
	return 0;
}
```

## 最小点割集

### [洛谷P1345 USACO5.4 奶牛的电信 Telecowmunication](https://www.luogu.com.cn/problem/P1345)

> 农夫约翰的奶牛们喜欢通过电邮保持联系，于是她们建立了一个奶牛电脑网络，以便互相交流。这些机器用如下的方式发送电邮：如果存在一个由 $N$ 台电脑组成的序列$c_1,c_2,\cdots,c_N$，且 $c_1$ 与 $c_2$ 相连，$c_2$ 与 $c_3$ 相连，等等。那么电脑 $c_1$ 和 $c_N$ 就可以互发电邮。
>
> 很不幸，有时候奶牛会不小心踩到电脑上，农夫约翰的车也可能碾过电脑，这台倒霉的电脑就会坏掉。这意味着这台电脑不能再发送电邮了，于是与这台电脑相关的连接也就不可用了。
>
> 有两头奶牛就想：如果我们两个不能互发电邮，至少需要坏掉多少台电脑呢？请注意，$c_1,c_2$ 不能被破坏。请编写一个程序为她们计算这个最小值。
>
> $1\leq N\leq 100$，$1\leq M\leq 600$，其中 $N$ 是电脑总数，电脑由 $1$ 到 $N$ 编号。$M$ 是电脑之间连接的总数。

第一眼像是最小割模板，一交 $80$ 分。

哪里出问题了呢？仔细一看发现我们要割的是点，而最小割割的是边。

我们可以将原图的边的容量设为无穷大，而对于点拆点处理，边的容量为 $1$。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=210,M=3010,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t;
int main() {
	g.clear();
	scanf("%d%d%d%d",&n,&m,&s,&t);
	for(int i=1;i<=n;i++)
		g.add(i,i+n,1);
	for(int i=1;i<=m;i++) {
		int x,y;
		scanf("%d%d",&x,&y);
		g.add(x+n,y,Inf);
		g.add(y+n,x,Inf);
	}
	g.dinic(s+n,t);
	printf("%d",g.maxflow);
	return 0;
}
```

## 最大收益/最小花费划分

### [洛谷P2057 SHOI2007 善意的投票](https://www.luogu.com.cn/problem/P2057)

> 幼儿园里有 $n$ 个小朋友打算通过投票来决定睡不睡午觉。
>
> 对他们来说，这个问题并不是很重要，于是他们决定发扬谦让精神。
>
> 虽然每个人都有自己的主见，但是为了照顾一下自己朋友的想法，他们也可以投和自己本来意愿相反的票。
>
> 我们定义一次投票的冲突数为下面两者相加：
>
> - 实际投票不同的好朋友对数。
> - 自己实际投票和自己本来意愿不同的人数。
>
> 我们的问题就是，每位小朋友应该怎样投票，才能使冲突数最小？
>
> $2\leq n\leq 300$，$1\leq m\leq\frac{n(n-1)}{2}$。其中 $n$ 代表总人数，$m$ 代表好朋友的对数。

首先，小朋友只有两种互斥的选择：同意睡午觉和不同意睡午觉。

我们可以先将源点向本来意愿睡午觉的小朋友连边，将本来意愿不睡午觉的小朋友向汇点连边。

此时源点和汇点分别表示一种立场，而一条 $A\to B$ 的边，表示 $A$ 要求 $B$ 和它同立场。

而题目中的冲突有两种：

1. 违背自身意愿。
2. 违背好朋友的意愿。

关于第一种冲突，根本上是由于第二种冲突引起的。假如这群小朋友之间不存在好朋友关系，那么就不会存在任何冲突，因为不会违背好朋友的意愿，也不会为了和好朋友的立场保持一致而违背了自身的意愿。

关于第二种冲突，是由于一对好朋友持不同的立场。设有一对好朋友 $A$ 和 $B$，$A$ 要求 $B$ 同自己一个立场，而 $B$ 要求 $A$ 同自己一个立场，于是便产生了冲突。根据一条单向边的含义，应在 $A$ 和 $B$ 建立双向边。

由于违背自己意愿的冲突根本上是由于好朋友之间的冲突，那么，解决冲突就必须先解决好朋友间的冲突。

所以，如何解决好朋友间的冲突呢？很简单，改变两人间任意一个人的立场，即割掉好朋友间的一条边，一个人就不再要求另一个人的立场相同。

其次，如何解决违背自身的冲突呢？无非也是割边，让自己不再被要求持该立场。

如何付出最小的代价，就解决所有的冲突呢？所割的边容量之和最小。这就转化成最小割问题了。

在最小花费的建模中，单项边表示冲突，而它的容量为解决冲突的花费。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=310,M=110010,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t;
int main() {
	g.clear();
	scanf("%d%d",&n,&m);
	s=n+1,t=n+2;
	for(int i=1;i<=n;i++) {
		int x;
		scanf("%d",&x);
		if(x) g.add(s,i,1);
		else g.add(i,t,1);
	}
	for(int i=1;i<=m;i++) {
		int x,y;
		scanf("%d%d",&x,&y);
		g.add(x,y,1);
		g.add(y,x,1);
	}
	g.dinic(s,t);
	printf("%d",g.maxflow);
	return 0;
}
```

### [洛谷P4313 文理分科](https://www.luogu.com.cn/problem/P4313)

> 文理分科是一件很纠结的事情！（虽然看到这个题目的人肯定都没有纠结过）
>
> 小 P 所在的班级要进行文理分科。他的班级可以用一个 $n\times m$ 的矩阵进行描述，每个格子代表一个同学的座位。每位同学必须从文科和理科中选择一科。同学们在选择科目的时候会获得一个满意值。满意值按如下的方式得到：
>
> - 如果第 $i$ 行第 $j$ 列的同学选择了文科，则他将获得 $art_{i,j}$ 的满意值，如果选择理科，将得到 $science_{i,j}$ 的满意值。
> - 如果第 $i$ 行第 $j$ 列的同学选择了文科，并且他相邻（两个格子相邻当且仅当它们拥有一条相同的边）的同学全部选择了文科，则他会更开心，所以会增加 $same\text{\underline{ }}art_{i,j}$ 的满意值。
> - 如果第 $i$ 行第 $j$ 列的同学选择了理科，并且他相邻的同学全部选择了理科，则增加 $same\text{\underline{ }}science_{i,j}$ 的满意值。
>
> 小 P 想知道，大家应该如何选择，才能使所有人的满意值之和最大。请告诉他这个最大值。
>
> $n,m\leq 100$。

如何求解最大收益类问题呢？我们可以先假设可以获得所有收益，但此时可能会产生冲突。再建模跑一遍最小花费，将所有收益的总和减去最小花费就是可获得的最大收益了。

之后就和上面那题差不多了。

我们把每一个人视作一个点 $i$，规定：$i$ 被割到 $S$ 集里，代表这个人选文，$i$ 被割到 $T$ 集里，代表这个人选理。

将源点 $s$ 连向每个点，容量为 $art$，将每个点连向汇点 $t$，容量为 $science$。

对于同时选择的情况，我们可以新建点。

对于几个相邻的点，我们新建一个点，将 $s$ 连向这个点，容量为 $same\text{\underline{ }}art$，如果该边被割，则说明这些人不同时选文，不能获得同时选文可获得的收益。再将新建的点向这个点和相邻的每一个点连一条容量为无穷大的边，这样的边在最小割中肯定不会存在，则保证了在代表共同选择收益的边不被割时（即都选一个科目）新建的点与相邻的点在同一个点集。

同样，我们新建一个点，将这个点连向 $t$，容量为 $same\text{\underline{ }}science$，如果该边被割，则说明这些人不同时选理，不能获得同时选理可获得的收益。再将这个点和相邻的每一个点向新建的点连一条容量为无穷大的边。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=30010,M=1e6,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t,sum;
int tx[]={0,0,1,-1},ty[]={1,-1,0,0};
int main() {
	g.clear();
	scanf("%d%d",&n,&m);
	s=3*n*m+1,t=s+1;
	int cnt=n*m;
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++) {
			int x,id=(i-1)*m+j;
			scanf("%d",&x);
			sum+=x;
			g.add(s,id,x);
		}
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++) {
			int x,id=(i-1)*m+j;
			scanf("%d",&x);
			sum+=x;
			g.add(id,t,x);
		}
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++) {
			int x,id=(i-1)*m+j;
			scanf("%d",&x);
			sum+=x,cnt++;
			g.add(s,cnt,x);
			g.add(cnt,id,Inf);
			for(int k=0;k<=3;k++) {
				int dx=i+tx[k],dy=j+ty[k];
				if(dx>=1&&dx<=n&&dy>=1&&dy<=m)
					g.add(cnt,(dx-1)*m+dy,Inf);
			}
		}
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++) {
			int x,id=(i-1)*m+j;
			scanf("%d",&x);
			sum+=x,cnt++;
			g.add(cnt,t,x);
			g.add(id,cnt,Inf);
			for(int k=0;k<=3;k++) {
				int dx=i+tx[k],dy=j+ty[k];
				if(dx>=1&&dx<=n&&dy>=1&&dy<=m)
					g.add((dx-1)*m+dy,cnt,Inf);
			}
		}
	g.dinic(s,t);
	printf("%d",sum-g.maxflow);
	return 0;
}
```

### [洛谷P1361 小M的作物](https://www.luogu.com.cn/problem/P1361)

> 小 M 在 MC 里开辟了两块巨大的耕地 $A$ 和 $B$（你可以认为容量是无穷），现在，小 P 有 $n$ 种作物的种子，每种作物的种子有 $1$ 个（就是可以种一棵作物），编号为 $1$ 到 $n$。
>
> 现在，第 $i$ 种作物种植在 $A$ 中种植可以获得 $a_i$ 的收益，在 $B$ 中种植可以获得 $b_i$ 的收益，而且，现在还有这么一种神奇的现象，就是某些作物共同种在一块耕地中可以获得额外的收益，小 M 找到了规则中共有 $m$ 种作物组合，第 $i$ 个组合中的作物共同种在 $A$ 中可以获得 $c_{1,i}$ 的额外收益，共同种在 $B$ 中可以获得 $c_{2,i}$ 的额外收益。
>
> 小 M 很快的算出了种植的最大收益，但是他想要考考你，你能回答他这个问题么？
>
> $1\leq n\leq 10^3$，$1\leq m\leq 10^3$。

与上一题相同，将源点 $s$ 向每个作物 $i$ 连一条容量为 $a_i$ 的边，每个作物 $i$ 向汇点 $t$ 连一条容量为 $b_i$ 的边。

每个组合新建两个点。将 $s$ 向第一个点连边，容量为 $c_{1,i}$，将第一个点向要求的每个点连一条容量为无穷大的边。将的二个点向 $t$ 连边，容量为 $c_{2,i}$，将要求的每个点向第二个点连一条容量为无穷大的边。

将所有收益的总和减去跑出来的最小花费就可以了。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=30010,M=1e7,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t,sum;
int main() {
	g.clear();
	scanf("%d",&n);
	s=n+1,t=n+2;
	for(int i=1;i<=n;i++) {
		int a;
		scanf("%d",&a);
		sum+=a;
		g.add(s,i,a);
	}
	for(int i=1;i<=n;i++) {
		int b;
		scanf("%d",&b);
		sum+=b;
		g.add(i,t,b);
	}
	scanf("%d",&m);
	for(int i=1;i<=m;i++) {
		int k,a,b;
		int p1=n+1+2*i,p2=p1+1;
		scanf("%d%d%d",&k,&a,&b);
		g.add(s,p1,a);
		g.add(p2,t,b);
		sum+=a+b;
		for(int j=1;j<=k;j++) {
			int x;
			scanf("%d",&x);
			g.add(p1,x,Inf);
			g.add(x,p2,Inf);
		}
	}
	g.dinic(s,t);
	printf("%d",sum-g.maxflow);
	return 0;
}
```

## 最大权闭合子图

### [讲解](https://blog.csdn.net/can919/article/details/77603353)

### [洛谷P3410 拍照](https://www.luogu.com.cn/problem/P3410)

> 小 B 有 $N$ 个下属，现小 B 要带着一些下属让别人拍照。
>
> 有 $M$ 个人，每个人都愿意付给小 B 一定钱来和 $N$ 个下属中的一些人进行合影。如果这一些下属没带齐那么就不能拍照，小B也不会得到钱。
>
> 求表示最大收益。
>
> $M,N\leq 100$。

如果选择一个合影就必须付给要求的下属费用，满足最大权闭合子图问题的适用条件。

将源点 $s$ 向每个合影连边，容量为其收益。每个合影向要求的下属合影连一条容量为无穷大的边。每个下属向汇点 $t$ 连边，容量为请下属拍照的费用。

答案为所有合影的收益和减去最小割。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=210,M=30010,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t,sum;
int main() {
	g.clear();
	scanf("%d%d",&n,&m);
	s=n+m+1,t=n+m+2;
	for(int i=1;i<=n;i++) {
		int x;
		scanf("%d",&x);
		sum+=x;
		g.add(s,i,x);
		while(1) {
            scanf("%d",&x);
            if(!x) break;
            g.add(i,n+x,Inf);
        }
	}
	for(int i=1;i<=m;i++) {
		int x;
		scanf("%d",&x);
		g.add(n+i,t,x);
	}
	g.dinic(s,t);
	printf("%d",sum-g.maxflow);
	return 0;
}
```

### [洛谷P4177 CEOI2008 order](https://www.luogu.com.cn/problem/P4177)

> 有 $N$ 个工作，$M$ 种机器，每种机器可以租或者买。
>
> 每个工作包括若干道工序，每道工序需要某种机器来完成，每种工作租用及其有不同的费用。
>
> 你需要最大化利益。
>
> $1\leq N,M\leq 1200$。

如果不能租用肯定就是最大权闭合子图的模板题。

租用就是只可以在这一个任务上使用这台机器，不影响其他任务。换言之就是只会影响这一个任务与这一台机器之间的联通关系。

考虑到之前不能租用的模型，任务与机器之间的连边是无穷大，现在更改成租用费用即可。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=3010,M=1e7,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t,sum;
int main() {
	g.clear();
	scanf("%d%d",&n,&m);
	s=n+m+1,t=n+m+2;
	for(int i=1;i<=n;i++) {
		int x,y,k;
		scanf("%d%d",&x,&k);
		sum+=x;
		g.add(s,i,x);
		for(int j=1;j<=k;j++) {
			scanf("%d%d",&x,&y);
			g.add(i,n+x,y);
		}
	}
	for(int i=1;i<=m;i++) {
		int x;
		scanf("%d",&x);
		g.add(n+i,t,x);
	}
	g.dinic(s,t);
	printf("%d",sum-g.maxflow);
	return 0;
}
```

## 二分图问题

二分图和网络流的关系非常密切，从许多方面都能看得出来。事实上，网络流可以解决所有常见的二分图问题。以下就举二分图最大匹配和二分图最小点覆盖的例子来讲解。

### 二分图最大匹配

二分图最大匹配可以转化成最大流模型。假设二分图可以分成 $X$ 和 $Y$ 两个点集，使得没有一条边的两个端点都在同一个集合内。

将源点向 $X$ 中的每一个点连边，容量为 $1$，表示只能有一条匹配边经过这个点。将 $Y$ 中的每个点向汇点连边，容量为 $1$，意义同上。

将 $X$ 中的每个点向 $Y$ 中的可匹配点连边，容量为无穷大（也可以是大于等于 $1$ 的任何数），若这条边有流量则表示这条边被选中。

此时的最大流即为最大匹配数。

### 二分图最小点覆盖

二分图最小点覆盖可以转化为最小割模型。同上将二分图分成 $X$ 和 $Y$ 两个点集，使得没有一条边的两个端点都在同一个集合内。

将源点向 $X$ 中的每一个点连边，容量为 $1$，表示这个点之多被覆盖一次，若便被割表示这个点被选中。将 $Y$ 中的每个点向汇点连边，容量为 $1$，意义同上。

将 $X$ 中的每个点向 $Y$ 中的可匹配点连边，容量为无穷大，防止其被割。

怎么保证每条边都有端点被覆盖呢？假设存在一条边没有端点被覆盖，则必然存在一条从源点到汇点经过这条边的通路，这样就不满足最小割了，矛盾。所以此时所有边都至少有一个定点被覆盖。

容易看出，Kőnig 定理（即二分图中，最小点覆盖等于最大匹配）是最大流最小割定理的特殊情形。实际上，它们都和线性规划中的对偶有关。