---
title: 网络流-费用流
published: 2024-10-08
description: ''
image: ''
tags: [图论,网络流]
category: '图论'
draft: false
---

# 定义

给定一个网络 $G=(V,E)$，每条边除了有容量限制 $c(u,v)$，还有一个单位流量的费用 $w(u,v)$。

当 $(u,v)$ 的流量为 $f(u,v)$ 时，需要花费 $f(u,v)\times w(u,v)$ 的费用。

$w$ 也满足斜对称性，即 $w(u,v)=-w(v,u)$ 。

则该网络中总花费最小的最大流称为最小费用最大流，即在最大化 $|f|$ 的前提下最小化 $\sum_{u\in V,v\in V}f(u,v)\times w(u,v)$。

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

### [洛谷P2517 HAOI2010 订货](https://www.luogu.com.cn/problem/P2517)

> 某公司估计市场在第 $i$ 个月对某产品的需求量为 $U_i$，已知在第 $i$ 月该产品的订货单价为 $d_i$，上个月月底未销完的单位产品要付存贮费用 $m$。
>
>每月月初订购，订购后产品立即到货，进库并供应市场，于当月被售掉则不必付存贮费。仓库有容量 $S$。
>
>假定第一月月初的库存量为 $0$，第 $n$ 月月底的库存量也为 $0$，问如何安排这 $n$ 个月订购计划，才能使成本最低？
>
> $0\leq n\leq 50$。

首先我们知道，每月都可以进货，并且不限数量，所以我们就从源点向表示每个月的点连一条容量为无穷大，费用为订货单价的边。

由于仓库容量和存贮费用的存在，我们就会自然的想到在相邻的月之间连一条费用为存贮费用，容量为仓库容量的边。

仅仅这样是不够的，我们还需要模拟出货物出售的情况，所以要从表示每个月的点向汇点连一条容量为需求量，费用为 $0$ 的边。

跑最小费用最大流就可以了。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e3+10,M=1e6+10,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,k,s,t;
int main() {
	g.clear();
	scanf("%d%d%d",&n,&m,&k);
	s=n+1,t=n+2;
	for(int i=1;i<=n;i++) {
		int x;
		scanf("%d",&x);
		g.add(i,t,x,0);
	}
	for(int i=1;i<=n;i++) {
		int x;
		scanf("%d",&x);
		g.add(s,i,Inf,x);
	}
	for(int i=1;i<n;i++) g.add(i,i+1,k,m);
	g.dinic(s,t);
	printf("%d",g.mincost);
	return 0;
}
```

### [洛谷P2153 SDOI2009 晨跑](https://www.luogu.com.cn/problem/P2153)

> Elaxia 最近迷恋上了空手道，他为自己设定了一套健身计划，比如俯卧撑、仰卧起坐等等，不过到目前为止，他坚持下来的只有晨跑。 
>
> 现在给出一张学校附近的地图，这张地图中包含 $N$ 个十字路口和 $M$ 条街道，Elaxia 只能从 一个十字路口跑向另外一个十字路口，街道之间只在十字路口处相交。
>
> Elaxia 每天从寝室出发跑到学校，保证寝室编号为 $1$，学校编号为 $N$。 
>
> Elaxia 的晨跑计划是按周期（包含若干天）进行的，由于他不喜欢走重复的路线，所以在一个周期内，每天的晨跑路线都不会相交（在十字路口处），寝室和学校不算十字路口。
>
> Elaxia 耐力不太好，他希望在一个周期内跑的路程尽量短，但是又希望训练周期包含的天数尽量长。
>
> 除了练空手道，Elaxia 其他时间都花在了学习和找 MM 上面，所有他想请你帮忙为他设计 一套满足他要求的晨跑计划。
>
> 可能存在 $1\rightarrow n$ 的边。这种情况下，这条边只能走一次。
>
> $N\leq 200$，$M\leq 2\times 10^4$。

首先我们发现需要求路程最短，天数尽量长。那么我们可以考虑最小费用最大流，其中路程为费用，天数为流量。

由于除了 $1$ 和 $n$ 外其他点都只能走一次，所以拆点处理，在入点和出点之间连容量为 $1$，费用为 $0$ 的边。

再将原图中的边的容量设为 $1$，费用设为其长度就好了。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=410,M=1e6+10,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t;
int main() {
	g.clear();
	scanf("%d%d",&n,&m);
	s=1,t=n;
	for(int i=2;i<n;i++) g.add(i,n+i,1,0);
	for(int i=1;i<=m;i++) {
		int x,y,z;
		scanf("%d%d%d",&x,&y,&z);
		if(x!=1&&x!=n) x+=n;
		g.add(x,y,1,z);
	}
	g.dinic(s,t);
	printf("%d %d",g.maxflow,g.mincost);
	return 0;
}
```

### [洛谷P4452 国家集训队 航班安排](https://www.luogu.com.cn/problem/P4452)

> 神犇航空有 $K$ 架飞机，为了简化问题，我们认为每架飞机都是相同的。神犇航空的世界中有 $N$ 个机场，以 $0\cdots N-1$ 编号，其中 $0$ 号为基地机场，每天 $0$ 时刻起飞机才可以从该机场起飞，并不晚于 $T$ 时刻回到该机场。
>
> 一天，神犇航空接到了 $M$ 个包机请求，每个请求为在 $s$ 时刻从 $a$ 机场起飞，在恰好 $t$ 时刻到达 $b$ 机场，可以净获利 $c$。换言之，你只需要在 $s$ 时刻在 $a$ 机场选择提供一架飞机给请求方，那么这架飞机就会在 $t$ 时刻准时出现在 $b$ 机场，并且你将获得 $c$ 的净利润。
>
> 设计一种方案，使得总收益最大。
>
> $1\leq N,M\leq 200$，$1\leq K\leq 10$，$1\leq T\leq 3000$。
>
> $t_{i,i}=f_{i,i}=0$，$t_{i,j}\leq t_{i,k}+t_{k,j}$，$f_{i,j}\leq f_{i,k}+f_{k,j}$。其中 $t_{i,j}$ 表示从机场 $i$ 空载飞至机场 $j$ 需要的时间，$f_{i,j}$ 表示从机场 $i$ 空载飞至机场 $j$ 需要的费用。

考虑以请求为点进行建图，对每个请求进行拆点，拆点后两个点之间连一条费用为 $c$，容量为 $1$ 的边，表示一个请求只能执行一次。

然后我们考虑时间限制：

对于一个请求，如果 $0$ 时刻可以从基地机场飞到该请求的起点，那么源点向该请求连一条费用为飞行费用的相反数，容量为无穷大的边。同理，若一个请求的结束时间，加上它的终点飞回基地机场的时间不超过 $T$，则该请求向汇点连边。

但是每次执行完一个请求并不一定要飞回基地机场，也可以飞去其他请求的起点，所以两两枚举请求，如果满足时间条件也进行连边。

最后考虑有 $k$ 架飞机，所以再建一个真正的源点，向原来的源点连费用为 $0$，流量为 $k$ 的边即可。

最后跑最大费用最大流。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=410,M=1e6+10,Inf=0x3f3f3f3f;
struct net{...};//此处为最大费用最大流
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
		g.add(i,i+m,1,q[i].g);
		if(q[i].b+c[q[i].y][1]>p)
			continue;
		g.add(i+m,t,Inf,-f[q[i].y][1]);
		if(q[i].a>=c[1][q[i].x])
			g.add(s,i,Inf,-f[1][q[i].x]);
		for(int j=1;j<=m;j++)
			if(q[i].b+c[q[i].y][q[j].x]<=q[j].a)
				g.add(i+m,j,Inf,-f[q[i].y][q[j].x]);
	}
	g.add(S,s,k,0);
	g.dinic(S,t);
	printf("%d",g.maxcost);
	return 0;
}
```

### [洛谷P4013 数字梯形问题](https://www.luogu.com.cn/problem/P4013)

> 给定一个由 $n$ 行数字组成的数字梯形。
>
> 梯形的第一行有 $m$ 个数字。从梯形的顶部的 $m$ 个数字开始，在每个数字处可以沿左下或右下方向移动，形成一条从梯形的顶至底的路径。
>
> 分别遵守以下规则：
>
> 1. 从梯形的顶至底的 $m$ 条路径互不相交；
> 2. 从梯形的顶至底的 $m$ 条路径仅在数字结点处相交；
> 3. 从梯形的顶至底的 $m$ 条路径允许在数字结点相交或边相交。
>
> 求每种规则下所有路径上数字和的最大总和。
>
> $1\leq m,n\leq 20$。

首先将源点向第一层的各个点连边，容量为 $1$，费用为 $0$。将最后一行的各个点向汇点连边，容量待定，费用为这个点的点权。

再将除最后一层外的所有点向左下和右下的点各连一条边，容量待定，费用为这个点的点权。

对于第一问，即路径没有公共点，也就是限制通过各点流量。拆点即可，容量为 $1$，费用为 $0$。所有待定的容量设为 $1$。

对于第二问，要求路径没有公共边，但可以有公共点。所以将梯形中的点互相连的边的容量设为 $1$，连向汇点的边的容量设为无穷大。

对于第三问，路径可以有公共边和公共点，将所有待定的容量设为 $0$ 就可以了。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1210,M=1e6+10,Inf=0x3f3f3f3f;
struct net{...};//此处为最大费用最大流
net g;
int n,m,s,t,a[100][100];
int getid(int x,int y) {
	return (2*m+x-2)*(x-1)/2+y;
}
int main() {
	scanf("%d%d",&m,&n);
	int all=(2*m+n-1)*n/2;
	for(int i=1;i<=n;i++)
		for(int j=1;j<m+i;j++)
			scanf("%d",&a[i][j]);
	g.clear();
	s=2*all+1,t=2*all+2;
	for(int i=1;i<=n;i++)
		for(int j=1;j<m+i;j++) {
			int in=getid(i,j),out=all+getid(i,j);
			g.add(in,out,1,0);
			if(i<n) {
				g.add(out,getid(i+1,j),1,a[i][j]);
				g.add(out,getid(i+1,j+1),1,a[i][j]);
			}
			if(i==1) g.add(s,in,1,0);
			if(i==n) g.add(out,t,1,a[i][j]);
		}
	g.dinic(s,t);
	printf("%d\n",g.maxcost);
	g.clear();
	s=all+1,t=all+2;
	for(int i=1;i<=n;i++)
		for(int j=1;j<m+i;j++) {
			int id=getid(i,j);
			if(i<n) {
				g.add(id,getid(i+1,j),1,a[i][j]);
				g.add(id,getid(i+1,j+1),1,a[i][j]);
			}
			if(i==1) g.add(s,id,1,0);
			if(i==n) g.add(id,t,Inf,a[i][j]);
		}
	g.dinic(s,t);
	printf("%d\n",g.maxcost);
	g.clear();
	s=all+1,t=all+2;
	for(int i=1;i<=n;i++)
		for(int j=1;j<m+i;j++) {
			int id=getid(i,j);
			if(i<n) {
				g.add(id,getid(i+1,j),Inf,a[i][j]);
				g.add(id,getid(i+1,j+1),Inf,a[i][j]);
			}
			if(i==1) g.add(s,id,1,0);
			if(i==n) g.add(id,t,Inf,a[i][j]);
		}
	g.dinic(s,t);
	printf("%d",g.maxcost);
	return 0;
}
```

### [洛谷P2053 SCOI2007 修车](https://www.luogu.com.cn/problem/P2053)

> 同一时刻有 $N$ 位车主带着他们的爱车来到了汽车维修中心。
>
> 维修中心共有 $M$ 位技术人员，不同的技术人员对不同的车进行维修所用的时间是不同的。
>
> 现在需要安排这 $M$ 位技术人员所维修的车及顺序，使得顾客平均等待的时间最小。
>
> 顾客的等待时间是指从他把车送至维修中心到维修完毕所用的时间。
>
> $2\leq M\leq 9$，$1\leq N\leq 60$。

本题的错误建图方式是，建一个二分图，左边是车，右边是工人，连边跑最小费用最大流。

对于每个工人而言，他在修第 $k$ 辆车的时候，之前已经修了 $k−1$ 辆车，所以对于不同的 $k$，对应的客人等待的时间是不同的，因此我们需要将每个工人拆成 $n$ 个点，分别表示修第几辆车的他。

但是你会发现这样做的话连边时的边权比较困难。所以我们需要做一个推导。假设某个工人一共修了 $K$ 辆车，花的时间分别为 $T_1,T_2,\cdots,T_K$，那么当他修第一辆车时，后面的 $K−1$ 个人必须等着，同时第 $1$ 个人也要等他修好，所以总共等待时间为 $K\times T_1$；接下来修第二辆车时同理，有 $K−1$ 个人在等他修，所以时间是 $(K−1)\times T_2$，以此类推，总等待时间即为 $\sum_{i=1}^KT_i(K−i+1)$。

不难发现，他倒数第 $i$ 个修的车对应的客人总共要贡献 $T_i\times i$ 的等待时间。于是有了这一步转化，我们可以给出最终的建图方案了：将每个工人拆成 $n$ 个点，分别表示修倒数第几辆车的他；如果第 $j$ 个工人修第 $i$ 辆车要花 $T(i,j)$ 的时间，我们枚举 $k=1,2,\cdots,n$，从第 $i$ 辆车向第 $j$ 个工人的第 $k$ 个点连边，容量为 $1$，费用为 $k\times T(i,j)$。然后跑最小费用最大流，最后总费用除以 $n$ 即可。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+10,M=1e6+10,Inf=0x3f3f3f3f;
struct net{...};
net g;
int n,m,s,t;
int main() {
	g.clear();
	scanf("%d%d",&m,&n);
	s=n*m+n+1,t=n*m+n+2;
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++) {
			int x;
			scanf("%d",&x);
			for(int k=1;k<=n;k++)
				g.add(i,j*n+k,1,x*k);
		}
	for(int i=1;i<=n;i++) g.add(s,i,1,0);
	for(int i=1;i<=n*m;i++) g.add(i+n,t,1,0);
	g.dinic(s,t);
	printf("%.2lf",1.0*g.mincost/n);
	return 0;
}
```

### [洛谷P2050 NOI2012 美食节](https://www.luogu.com.cn/problem/P2050)

> CZ 市为了欢迎全国各地的同学，特地举办了一场盛大的美食节。
>
> 作为一个喜欢尝鲜的美食客，小 M 自然不愿意错过这场盛宴。他很快就尝遍了美食节所有的美食。然而，尝鲜的欲望是难以满足的。尽管所有的菜品都很可口，厨师做菜的速度也很快，小 M 仍然觉得自己桌上没有已经摆在别人餐桌上的美食是一件无法忍受的事情。于是小 M 开始研究起了做菜顺序的问题，即安排一个做菜的顺序使得同学们的等待时间最短。
>
> 小 M 发现，美食节共有 $n$ 种不同的菜品。每次点餐，每个同学可以选择其中的一个菜品。总共有 $m$ 个厨师来制作这些菜品。当所有的同学点餐结束后，菜品的制作任务就会分配给每个厨师。然后每个厨师就会同时开始做菜。厨师们会按照要求的顺序进行制作，并且每次只能制作一人份。
>
> 此外，小 M 还发现了另一件有意思的事情——虽然这 $m$ 个厨师都会制作全部的 $n$ 种菜品，但对于同一菜品，不同厨师的制作时间未必相同。他将菜品用 $1, 2, \ldots, n$ 依次编号，厨师用 $1, 2, \ldots, m$ 依次编号，将第 $j$ 个厨师制作第 $i$ 种菜品的时间记为 $t_{i,j}$。
>
> 小 M 认为：每个同学的等待时间为所有厨师开始做菜起，到自己那份菜品完成为止的时间总长度。换句话说，如果一个同学点的菜是某个厨师做的第 $k$ 道菜，则他的等待时间就是这个厨师制作前 $k$ 道菜的时间之和。而总等待时间为所有同学的等待时间之和。
>
> 现在，小 M 找到了所有同学的点菜信息——有 $p_i$ 个同学点了第 $i$ 种菜品（$i=1, 2, \ldots, n$）。他想知道的是最小的总等待时间是多少。
>
> $n\leq 40$，$m\leq 100$。

这是一道非常好的动态加点的费用流题目。是这道题目上道题的加强版。

我们先按上道题的思路：把厨师分层。

如果第 $j$ 个厨师先后做了第 $a_1,a_2,\cdots ,a_w$ 种菜，那么等待时间之和就是 $t_{a_1,j}w+t_{a_2,j}(w−1)+\cdots +t_{a_w,j}$。

第 $k$ 项的意思是做了第 $k$ 道菜让后面的 $w−k$ 道菜每个都等了 $t_{a_k,j}$ 的时间。

那么我们让第 $j$ 个厨师的第 $w$ 层表示第 $j$ 个厨师做了倒数 $w$ 道菜。方便起见，把它用 $(j,w)$ 表示。

它与第 $i$ 道菜连边就表示第 $j$ 个厨师做的倒数第 $t$ 道菜是第 $i$ 种菜。

设源点为 $s$，汇点为 $t$，菜的编号为 $1\sim n$，分层厨师就用上面说的 $(j,w)$ 表示。

- 从源点 $s$ 向每道菜连一条容量为 $p_i$，费用为 $0$ 的边，表示第 $i$ 中菜需要做 $p_i$ 道。

- 每个 $(j,w)$ 向汇点 $t$ 连一条容量为 $1$，费用为 $0$ 的边，表示第 $j$ 个厨师做倒数第 $w$ 道菜。

- 每道菜向每个 $(j,w)$ 连一条容量为 $1$，费用为 $t_{i,j}\times w$ 的边，表示第 $i$ 种菜的其中一道是第 $j$ 个厨师做的倒数第 $w$ 道菜，根据开头给出的那个式子可知，它贡献的费用是 $t_{i,j}\times w$。

这样连为什么正确呢？

根据贪心的思想，设第 $j$ 个厨师做了 $w_j$ 道菜，那么被用到的点一定是 $(j,1)\sim (j,w_j)$ 连续的。

假设是断续的，由于中间没有被用到的层需要的费用一定比后面的层所需要的费用小，所以断续的情况一定不是最优的。

这样做点数的规模是 $O(mp+n)$ 的，边数的规模是 $O(nmp)$ 的。

根据这个建图，就能获得 $60pts$ 的好成绩！

其实根据上面的建模可以发现，好多 $(j,w)$ 的点是没有用到的。

根据上述的贪心证明可知：被用到的一定是从 $(j,1)$ 开始的连续的几个层。

那么我们为什么不一边跑流一边加点呢？这样我们就能省去很多无用的点。

我们设第 $j$ 个厨师已经加到了第 $top_j$ 层。若再一次跑流之后，$(j,top_j)$ 被用掉了，那么我们直接把 $(j,top_j+1)$ 加进图里面就好了。

这样点数的规模就降到了 $O(n+m+p)$，边数的规模降到了 $O(np)$。

那么怎么判断 $(j,top_j)$ 是否被跑过呢？可以直接看 $(j,top_j)$ 是否被某一条增广路经过，在跑流的时候打标记就好了。

`dfs` 改为：

```cpp
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
			if(d) v[x]=y;//增加此行
		}
	}
	vis[x]=0;
	return ret;
}
```

`dinic` 改为：

```cpp
void dinic(int s,int t) {
	maxflow=mincost=0;
	while(spfa(s,t)) {
		memcpy(cur,head,sizeof cur);
		maxflow+=dfs(s,t,Inf);
		//从此行开始
		for(int j=1;j<=m;j++)
			if(v[n+id(j,top[j])]&&top[j]<sum) {
				int now=n+id(j,++top[j]);
				for(int i=1;i<=n;i++)
					add(i,now,1,top[j]*c[i][j]);
				add(now,t,1,0);
			}
		//增加到此行
	}
}
```

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+10,M=1e6+10,K=1e3+10,Inf=0x3f3f3f3f;
int n,m,s,t,cnt=1,sum,p[K],c[K][K],top[K],v[N];
int id(int x,int y) {
	return (x-1)*sum+y;
}
struct net{...};
net g;
int main() {
	g.clear();
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;i++) {
		scanf("%d",&p[i]);
		sum+=p[i];
	}
	s=m*sum+n+1,t=m*sum+n+2;
	for(int i=1;i<=n;i++) g.add(s,i,p[i],0);
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++) {
			scanf("%d",&c[i][j]);
			g.add(i,n+id(j,1),1,c[i][j]);
		}
	for(int i=1;i<=m;i++)
		top[i]=1,g.add(n+id(i,1),t,1,0);
	g.dinic(s,t);
	printf("%d",g.mincost);
	return 0;
}
```