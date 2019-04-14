[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/TCD)

# 【HNOI2015】开店 题解

傻逼BZOJ把它设为了权限题，呵呵，洛谷上可以直接做

简化一下题意，就是：给定一棵n个点的树，有边权也有点权，询问点权在[L,R]之间的点到点u的距离总和

听说是动态树分治，但我也懒得学，就想到了一个鬼畜的做法：可持久化树链剖分

假设我们已经求出任一点到根的距离dis，我们知道，点u、v的距离dist(u,v)=dis[u]+dis[v]-2*dis[LCA(u,v)]

我们按点权排序n个点，那么点u到[L,R]之间所有点的距离就是(R-L+1)*dis[u]+sigma(dis[s],s∈[L,R])-2*sigma(dis[LCA(u,s)],s∈[L,R])

式子的第一项可以直接算，第二项可以对排好序后的n个点的dis数组做一次前缀和，也可以直接算，现在我们只要求出第三项就好了

我们考虑路径覆盖的问题

把[L,R]之间的点到根的路径覆盖一遍，然后把根到点u这一段的覆盖长度求出来就是第三项了

可以用树链剖分搞路径覆盖，因为有一个区间，所以对它进行可持久化

万物皆可持久化！

```cpp
#include<bits/stdc++.h>
#define INF 0x3f3f3f3f
#define newnode(x) x=new Node,x->lc=x->rc=null
using namespace std;

typedef long long LL;
struct Edge{int to,capa,next;} e[300010];
int h[150010],sum=0;
struct Node
{
	Node *lc,*rc;
	int tag;LL s;
	Node(){tag=0;s=0;lc=rc=NULL;}
};
Node *rt[150010],*null,*last;
int top[150010],fa[150010];
struct POI
{
	int age,id;
	POI(int x=0,int y=0):age(x),id(y){}
} point[150010];
int size[150010],hson[150010];
LL dis[150010];
int dfn[150010],dfs_clock=0,lst[150010];
LL sume[150010],sumd[150010];

void add_edge(int u,int v,int w)
{
	sum++;
	e[sum].to=v;
	e[sum].capa=w;
	e[sum].next=h[u];
	h[u]=sum;
}

bool operator < (const POI &x,const POI &y){return (x.age<y.age)||(x.age==y.age&&x.id<y.id);}

void modify(Node* &o,Node *pre,int l,int r,int nl,int nr)
{
	newnode(o);
	o->lc=pre->lc;o->rc=pre->rc;
	o->s=pre->s;o->tag=pre->tag;
	o->s+=sume[nr]-sume[nl-1];
	if(l==nl&&r==nr){o->tag++;return;}
	int mid=(l+r)/2;
	if(nl>mid) modify(o->rc,pre->rc,mid+1,r,nl,nr);
	else if(nr<=mid) modify(o->lc,pre->lc,l,mid,nl,nr);
	else modify(o->lc,pre->lc,l,mid,nl,mid),modify(o->rc,pre->rc,mid+1,r,mid+1,nr);
}

LL query(Node *o,int l,int r,int nl,int nr)
{
	if(l==nl&&r==nr) return o->s;
	LL res=(LL)(sume[nr]-sume[nl-1])*o->tag;
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
		fa[v]=u;dis[v]=dis[u]+(LL)e[tmp].capa;
		lst[v]=e[tmp].capa;
		dfs1(v,u);size[u]+=size[v];
		if((!hson[u])||size[v]>size[hson[u]]) hson[u]=v;
	}
}

void dfs2(int u,int tp)
{
	dfn[u]=++dfs_clock;
	sume[dfs_clock]=lst[u];
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
	newnode(null);rt[0]=last=null;
	int n,A,q,u,v,w,l,r;
	cin>>n>>q>>A;
	for(int i=1;i<=n;i++)
	{
		scanf("%d",&point[i].age);
		point[i].id=i;
	}
	sort(point+1,point+1+n);
	for(int i=1;i<n;i++)
	{
		scanf("%d%d%d",&u,&v,&w);
		add_edge(u,v,w);
		add_edge(v,u,w);
	}
	dfs1(1,0);dfs2(1,1);
	for(int i=1;i<=n;i++) sume[i]+=sume[i-1];
	for(int i=1;i<=n;i++) sumd[i]=sumd[i-1]+dis[point[i].id];
	for(int i=1;i<=n;i++)
	{
		int p=point[i].id;
		while(top[p]!=1)
		{
			modify(rt[i],last,1,n,dfn[top[p]],dfn[p]);
			last=rt[i];
			p=fa[top[p]];
		}
		modify(rt[i],last,1,n,1,dfn[p]);
		last=rt[i];
	}
	LL ans=0,res;
	for(int i=1;i<=q;i++)
	{
		scanf("%d%d%d",&u,&v,&w);
		ans%=A;v+=(int)ans;w+=(int)ans;
		l=min(v%A,w%A);r=max(v%A,w%A);
		l=lower_bound(point+1,point+1+n,POI(l,0))-point-1;
		r=upper_bound(point+1,point+1+n,POI(r,INF))-point-1;
		ans=(LL)(r-l)*dis[u]+sumd[r]-sumd[l];res=0;
		while(top[u]!=1)
		{
			res+=query(rt[r],1,n,dfn[top[u]],dfn[u]);
			res-=query(rt[l],1,n,dfn[top[u]],dfn[u]);
			u=fa[top[u]];
		}
		res+=query(rt[r],1,n,1,dfn[u]);
		res-=query(rt[l],1,n,1,dfn[u]);
		ans-=2*res;
		printf("%lld\n",ans);
	}
	return 0;
}
```
