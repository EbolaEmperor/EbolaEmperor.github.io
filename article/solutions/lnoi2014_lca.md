[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/TCD)

# 【LNOI2014】LCA 题解

上午拿这题做了一下NOIP模拟赛（TM确定这是NOIP难度？？？），然后我当即想起了昨晚做的[HNOI2015]开店，用我做那题的思路来搞这题，这题简直就是完全弱化版。然而机房一个大佬真的写了三个小时的可持久化树剖，还是没做出来，默哀三秒钟……其实我也想就把昨天代码搬上去的，然后发现，时间只有1s，空间只有128M，写个鬼的可持久化树剖！

这题没有强制在线，太好了，直接离线

还是考虑路径覆盖

按p=1~n的顺序将点p到根节点的路径覆盖，对于所有L=p+1的询问，答案减去目前为止u到根节点的覆盖长度，对于所有R=p的询问，答案加上到目前为止u到根节点的覆盖长度。然后再去覆盖下一个

就这样，AC，非常快

```cpp
#include<bits/stdc++.h>
#define INF 0x3f3f3f3f
#define Mod 201314
#define newnode(x) x=new Node,x->lc=x->rc=null
using namespace std;

typedef long long LL;
struct Edge{int to,next;} e[300010];
int h[150010],sum=0;
struct Node
{
	Node *lc,*rc;
	int tag;LL s;
	Node(){tag=0;s=0;lc=rc=NULL;}
};
Node *rt,*null;
int top[50010],fa[50010];
int size[50010],hson[50010];
LL dis[50010];
int dfn[50010],dfs_clock=0;
vector<int> q1[50010],q2[50010];
int U[50010];
LL ans[50010];

void add_edge(int u,int v)
{
	if(u==v) return;
	sum++;
	e[sum].to=v;
	e[sum].next=h[u];
	h[u]=sum;
}

void build(Node* &o,int l,int r)
{
	newnode(o);
	if(l==r) return;
	int mid=(l+r)/2;
	build(o->lc,l,mid);
	build(o->rc,mid+1,r);
}

void modify(Node* &o,int l,int r,int nl,int nr)
{
	o->s+=nr-nl+1;
	if(l==nl&&r==nr){o->tag++;return;}
	int mid=(l+r)/2;
	if(nl>mid) modify(o->rc,mid+1,r,nl,nr);
	else if(nr<=mid) modify(o->lc,l,mid,nl,nr);
	else modify(o->lc,l,mid,nl,mid),modify(o->rc,mid+1,r,mid+1,nr);
}

LL query(Node *o,int l,int r,int nl,int nr)
{
	if(l==nl&&r==nr) return o->s;
	LL res=(LL)(nr-nl+1)*o->tag;
	int mid=(l+r)/2;
	if(nl>mid) res+=query(o->rc,mid+1,r,nl,nr);
	else if(nr<=mid) res+=query(o->lc,l,mid,nl,nr);
	else res+=query(o->lc,l,mid,nl,mid)+query(o->rc,mid+1,r,mid+1,nr);
	return res;
}

void dfs1(int u,int la)
{
	size[u]=1;
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
	{
		int v=e[tmp].to;
		if(v==la) continue;
		fa[v]=u;dis[v]=dis[u]+1;
		dfs1(v,u);size[u]+=size[v];
		if((!hson[u])||size[v]>size[hson[u]]) hson[u]=v;
	}
}

void dfs2(int u,int tp)
{
	dfn[u]=++dfs_clock;
	top[u]=tp;
	if(hson[u]) dfs2(hson[u],tp);
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
	{
		int v=e[tmp].to;
		if(v==fa[u]||v==hson[u]) continue;
		dfs2(v,v);
	}
}

int main()
{
	newnode(null);
	int n,q,u,v,w,l,r;
	cin>>n>>q;
	for(int i=2;i<=n;i++)
	{
		scanf("%d",&u);u++;
		add_edge(u,i);
		add_edge(i,u);
	}
	dfs1(1,0);dfs2(1,1);
	build(rt,1,n);
	for(int i=1;i<=q;i++)
	{
		scanf("%d%d%d",&l,&r,U+i);
		l++;r++;U[i]++;
		q1[l-1].push_back(i);
		q2[r].push_back(i);
	}
	for(int i=1;i<=n;i++)
	{
		int p=i;
		while(top[p]!=1)
		{
			modify(rt,1,n,dfn[top[p]],dfn[p]);
			p=fa[top[p]];
		}
		modify(rt,1,n,1,dfn[p]);
		for(int j=0;j<q1[i].size();j++)
		{
			p=U[q1[i][j]];
			while(top[p]!=1)
			{
				ans[q1[i][j]]-=query(rt,1,n,dfn[top[p]],dfn[p]);
				ans[q1[i][j]]%=Mod;
				p=fa[top[p]];
			}
			ans[q1[i][j]]-=query(rt,1,n,1,dfn[p]);
			ans[q1[i][j]]%=Mod;
		}
		for(int j=0;j<q2[i].size();j++)
		{
			p=U[q2[i][j]];
			while(top[p]!=1)
			{
				ans[q2[i][j]]+=query(rt,1,n,dfn[top[p]],dfn[p]);
				ans[q2[i][j]]%=Mod;
				p=fa[top[p]];
			}
			ans[q2[i][j]]+=query(rt,1,n,1,dfn[p]);
			ans[q2[i][j]]%=Mod;
		}
	}
	for(int i=1;i<=q;i++) printf("%lld\n",ans[i]);
	return 0;
}
```
