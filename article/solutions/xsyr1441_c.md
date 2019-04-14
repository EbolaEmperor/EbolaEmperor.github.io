# C. san

### 题意简述

有n个点，每个点有三个属性，只要点a有至少两个属性大于点b，则a可以击败b。求哪些点最后可以胜出。保证对于任何一个属性，都没有两个点是相同的

(GDOI2018原题)

### 题解

暴力思路显然。枚举两个点a和b，如果a可以击败b，则a向b连有向边。然后缩点，入度为0的强连通分量里的点是可胜出的

复杂度瓶颈在于建边。考虑线段树优化

以属性一的值作为时间轴，属性二的值作为线段树下标。然后枚举到一个点x，就向线段树区间\[1,val\[x\]\[2]\]连边，然后将这个点加入线段树。这样就处理出了靠属性一二取胜的点对。同理可以处理出靠属性一三取胜的点对、靠属性二三取胜的点对

最后缩点。不难发现这个图缩点后是一条链，所以只有编号最大的强连通分量入度为0，也就这个强连通分量内的点最后能胜出

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010,M=N*100;
struct Edge{int from,to,next;} e[M<<1];
struct Uni{int a,b,c;} un[N];
int posa[N],posb[N],posc[N];
int h[M],sum=0,tot=0,n;
int tr[N<<2];
int dfn[M],low[M],scc[M];
int dfc=0,sccn=0;
stack<int> s;

void add_edge(int u,int v)
{
	e[++sum].to=v;
	e[sum].from=u;
	e[sum].next=h[u];
	h[u]=sum;
}

void add(int o,int l,int r,int nl,int nr,int x)
{
	if(l>=nl&&r<=nr)
	{
		add_edge(x,tr[o]);
		return;
	}
	int mid=(l+r)/2;
	if(nl<=mid) add(o<<1,l,mid,nl,nr,x);
	if(nr>mid) add(o<<1|1,mid+1,r,nl,nr,x);
}

void insert(int o,int l,int r,int k,int x)
{
	if(l==r){tr[o]=x;return;}
	int mid=(l+r)/2;
	if(k<=mid) insert(o<<1,l,mid,k,x);
	else insert(o<<1|1,mid+1,r,k,x);
	if(!tr[o<<1]) tr[o]=tr[o<<1|1];
	else if(!tr[o<<1|1]) tr[o]=tr[o<<1];
	else
	{
		tr[o]=++tot;
		add_edge(tr[o],tr[o<<1]);
		add_edge(tr[o],tr[o<<1|1]);
	}
}

void build()
{
	tot=n;
	for(int i=1;i<=n;i++)
	{
		add(1,1,n,1,un[posa[i]].b,posa[i]);
		insert(1,1,n,un[posa[i]].b,posa[i]);
	}
	memset(tr,0,sizeof(tr));
	for(int i=1;i<=n;i++)
	{
		add(1,1,n,1,un[posa[i]].c,posa[i]);
		insert(1,1,n,un[posa[i]].c,posa[i]);
	}
	memset(tr,0,sizeof(tr));
	for(int i=1;i<=n;i++)
	{
		add(1,1,n,1,un[posb[i]].c,posb[i]);
		insert(1,1,n,un[posb[i]].c,posb[i]);
	}
}

void Tarjan(int u)
{
	s.push(u);
	dfn[u]=low[u]=++dfc;
	for(int t=h[u];t;t=e[t].next)
	{
		int v=e[t].to;
		if(!dfn[v]) Tarjan(v),low[u]=min(low[u],low[v]);
		else if(!scc[v]) low[u]=min(low[u],dfn[v]);
	}
	if(low[u]==dfn[u])
	{
		int o;sccn++;
		do{
			o=s.top();
			scc[o]=sccn;
			s.pop();
		}while(o!=u);
	}
}

int main()
{
	scanf("%d",&n);
	for(int i=1;i<=n;i++)
	{
		scanf("%d",&un[i].a);posa[un[i].a]=i;
		scanf("%d",&un[i].b);posb[un[i].b]=i;
		scanf("%d",&un[i].c);posc[un[i].c]=i;
	}
	build();
	for(int i=1;i<=n;i++)
		if(!dfn[i]) Tarjan(i);
	for(int i=1;i<=n;i++)
		puts(scc[i]==sccn?"1":"0");
	return 0;
}
```