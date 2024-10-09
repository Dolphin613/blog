---
title: 网络流-费用流
published: 2024-10-08
description: ''
image: ''
tags: [图论,网络流]
category: '图论'
draft: true
---

# 定义

给定一个网络 $G=(V,E)$ ，每条边除了有容量限制 $c(u,v)$ ，还有一个单位流量的费用 $w(u,v)$ 。

当 $(u,v)$ 的流量为 $f(u,v)$ 时，需要花费 $f(u,v)\times w(u,v)$ 的费用。

$w$ 也满足斜对称性，即 $w(u,v)=-w(v,u)$ 。

则该网络中总花费最小的最大流称为最小费用最大流，即在最大化 $\sum_{(s,v)\in E}f(s,v)$ 的前提下最小化 $\sum_{(u,v)\in E}f(u,v)\times w(u,v)$。

# 求法

见 [OI-wiki](https://oi-wiki.org/graph/flow/min-cost/)。

<!-- ```cpp
struct net{
	int tot,maxflow,mincost,head[N],cur[N],dis[N],vis[N];
	int to[M],nxt[M],cost[M],cap[M],flow[M];
	void clear() {
		tot=1;
		memset(head,0,sizeof head);
	}
	void _add(int x,int y,int z,int w) {
		to[++tot]=y;
		cap[tot]=z;
		cost[tot]=w;
		flow[tot]=0;
		nxt[tot]=head[x];
		head[x]=tot;
	}
	void add(int x,int y,int z,int w) {
		_add(x,y,z,w),_add(y,x,0,-w);
	}
	bool spfa(int s,int t) {
		memset(dis,0x3f,sizeof dis);
		queue<int>q;
		q.push(s),vis[s]=1,dis[s]=0;
		while(!q.empty()) {
			int x=q.front();
			q.pop(),vis[x]=0;
			for(int i=head[x];i;i=nxt[i]) {
				int y=to[i];
				if(dis[y]>dis[x]+cost[i]&&cap[i]>flow[i]) {
					dis[y]=dis[x]+cost[i];
					if(!vis[y]) q.push(y),vis[y]=1;
				}
			}
		}
		return dis[t]!=Inf;
	}
	int dfs(int x,int t,int now) {
		if(x==t) return now;
		int ret=0;
		vis[x]=1;
		for(int i=cur[x];i&&now>ret;i=nxt[i]) {
			cur[x]=i;
			int y=to[i];
			if(!vis[y]&&dis[y]==dis[x]+cost[i]&&cap[i]>flow[i]) {
				int d=dfs(y,t,min(now-ret,cap[i]-flow[i]));
				mincost+=d*cost[i],ret+=d,flow[i]+=d,flow[i^1]-=d;
			}
		}
		vis[x]=0;
		return ret;
	}
	void dinic(int s,int t) {
		maxflow=mincost=0;
		while(spfa(s,t)) {
			memcpy(cur,head,sizeof cur);
			maxflow+=dfs(s,t,Inf);
		}
	}
};
``` -->

# 应用

### [洛谷P4452 国家集训队 航班安排](https://www.luogu.com.cn/problem/P4452)

> 神犇航空有 $K$ 架飞机，为了简化问题，我们认为每架飞机都是相同的。神犇航空的世界中有 $N$ 个机场，以 $0\cdots N-1$ 编号，其中 $0$ 号为基地机场，每天 $0$ 时刻起飞机才可以从该机场起飞，并不晚于 $T$ 时刻回到该机场。
>
> 一天，神犇航空接到了 $M$ 个包机请求，每个请求为在 $s$ 时刻从 $a$ 机场起飞，在恰好 $t$ 时刻到达 $b$ 机场，可以净获利 $c$。换言之，你只需要在 $s$ 时刻在 $a$ 机场选择提供一架飞机给请求方，那么这架飞机就会在 $t$ 时刻准时出现在 $b$ 机场，并且你将获得 $c$ 的净利润。
>
> 设计一种方案，使得总收益最大。
>
> $1\le N,M\le 200$，$1\le K\le 10$，$1\le T\le 3000$。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=410,M=1e6+10,Inf=0x3f3f3f3f;
struct net{...};
net g;
struct rec{
	int x,y,a,b,g;
}q[N];
int n,m,k,p,c[N][N],f[N][N];
int main() {
	g.clear();
	scanf("%d%d%d%d",&n,&m,&k,&p);
	int S=2*m+1,s=2*m+2,t=2*m+3;
	for(int i=1;i<=n;i++)
		for(int j=1;j<=n;j++)
			scanf("%d",&c[i][j]);
	for(int i=1;i<=n;i++)
		for(int j=1;j<=n;j++)
			scanf("%d",&f[i][j]);
	for(int i=1;i<=m;i++) {
		scanf("%d%d",&q[i].x,&q[i].y);
		scanf("%d%d%d",&q[i].a,&q[i].b,&q[i].g);
		q[i].x++,q[i].y++;
	}
	for(int i=1;i<=m;i++) {
		g.add(i,i+m,1,-q[i].g);
		if(q[i].b+c[q[i].y][1]>p)
			continue;
		g.add(i+m,t,Inf,f[q[i].y][1]);
		if(q[i].a>=c[1][q[i].x])
			g.add(s,i,Inf,f[1][q[i].x]);
		for(int j=1;j<=m;j++)
			if(q[i].b+c[q[i].y][q[j].x]<=q[j].a)
				g.add(i+m,j,Inf,f[q[i].y][q[j].x]);
	}
	g.add(S,s,k,0);
	g.dinic(S,t);
	printf("%d",-g.mincost);
	return 0;
}
```

