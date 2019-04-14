更好的阅读体验：[戳我！](https://ebola-emperor.blog.luogu.org/gdoi2016-feng-kuang-dong-wu-cheng-ti-xie)

### 简明题意

给你一棵$n$个节点的树。你需要在线支持三种操作：

1. 将$x$到$y$路径上所有点的点权加$k$

2. 退回到第$x$次$1$类操作的状态

3. 询问$x$到$y$路径上的点权的阶梯和之和（包括点$x$，不包括点$y$，若$x=y$则答案为0）

其中，若干个数（$a_1,a_2,...,a_n$）的“阶梯和之和”定义为：$\sum\limits_{i=1}^na_i\sum\limits_{j=1}^{n-i+1}j$

### 题解

毒瘤数据结构题，人均代码量应该是6K吧……

看一眼就应该知道是树链剖分套可持久化线段树，比较难想的就是线段树要维护什么、怎么维护

那么线段树维护的方式有很多，有一些做法是和方向有关，我就说一下我的与方向无关的做法

对于每个线段树节点，维护：子节点树上深度之和$d$，子节点树上深度平方和$d2$，子节点深度乘权值之和$ds$，子节点深度平方乘权值之和$d2s$，子节点权值之和$s$，懒标记$tag$。其中$d$和$d2$在一开始建树时就可以确定下来，不需要进行后续的维护操作，可持久化时直接继承即可。然后因为可持久化线段树不方便进行pushdown操作，那么我们这里采用标记永久化的方式来解决，还顺便降下了一些常数

上传操作是显然的，因为$ds$、$d2s$、$s$都具有可加性，直接将两个子节点的对应信息加起来即可，由于标记永久化，要记得加上自己的懒标记对相关变量造成的影响

对于一个区间加$k$的操作，那么在继承的基础上，$ds$要加$k\times d$，$d2s$要加$k\times d2$，$s$是常规操作就不说了，然后$tag$要加$k$（话说我$tag$忘了继承，结果debug了半小时……）

那么怎么利用这些东西来得到询问的答案？

我们设$x$到$y$路径上一个点$u$到$y$的路径长度为$dis_u$，那么，任意一个点$u$对本次询问的贡献都是$a_u\frac{dis_u(dis_u+1)}{2}$，这个除以2好丑，我们先不管它，算完全部再统一除以2。那么我们需要计算的就是$a_udis_u(dis_u+1)$。

考虑$lca(x,y)$左边的点，有$dis_u=deep_u+deep_y-2deep_{lca}$，对于这一部分的点来说，$deep_y-2deep_{lca}$是一个常数，我们将它设为$c$，并且先算出它来。于是我们要求的式子化为了$a_u(deep_u+c)(deep_u+c+1)$，大力展开一下，就变成了$a_udeep_u^2+a_udeep_u(2c+1)+a_u(c^2+c)$，那如果把一段区间的加起来，我们维护的$d2s$、$ds$、$s$就可以直接用了

考虑$lca(x,y)$右边的点，有$dis_u=deep_y-deep_u$，对于这一部分的点来说，$deep_y$是一个常数，我们设为$c$。于是我们要求的式子化为了$a_u(c-deeo_u)(c-deep_u+1)$，大力展开：$a_udeep_u^2-a_udeep_u(2c+1)+a_u(c^2+c)$，计算方式和上面那个是一样的

最后答案记得要除以2。然后……~~愉快地码吧~~

```cpp
#include<bits/stdc++.h>
#define ha 20160501
using namespace std;

namespace IO
{
	const int S=(1<<20)+5;
	//Input Correlation
	char buf[S],*H,*T;
	inline char Get()
	{
		if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
		if(H==T) return -1;return *H++;
	}
	inline int read()
	{
		int x=0;char c=Get();
		while(!isdigit(c)) c=Get();
		while(isdigit(c)) x=x*10+c-'0',c=Get();
		return x;
	}
	//Output Correlation
	char obuf[S],*oS=obuf,*oT=oS+S-1,c,qu[55];int qr;
	inline void flush(){fwrite(obuf,1,oS-obuf,stdout);oS=obuf;}
	inline void putc(char x){*oS++ =x;if(oS==oT) flush();}
	template <class I>inline void print(I x)
	{
		if(!x) putc('0');
		if(x<0) putc('-'),x=-x;
		while(x) qu[++qr]=x%10+'0',x/=10;
		while(qr) putc(qu[qr--]);
		putc('\n');
	}
}

typedef pair<int,int> pii;
using namespace IO;
const int N=100010;
struct Edge{int to,next;} e[N<<1];
int h[N],esum=0,n,m,val[N];
int fa[N],hson[N],top[N];
int size[N],dep[N];
int dfn[N],idx[N],clk=0;

void add_edge(int u,int v)
{
	e[++esum].to=v;
	e[esum].next=h[u];
	h[u]=esum;
}

void dfs1(int u,int la)
{
	size[u]=1;int mx=0;
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
	{
		int v=e[tmp].to;
		if(v==la) continue;
		dep[v]=dep[u]+1;
		dfs1(v,u);fa[v]=u;
		size[u]+=size[v];
		if(size[v]>mx) mx=size[v],hson[u]=v;
	}
}

void dfs2(int u,int tp)
{
	top[u]=tp;idx[dfn[u]=++clk]=u;
	if(hson[u]) dfs2(hson[u],tp);
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
		if(e[tmp].to!=fa[u]&&e[tmp].to!=hson[u])
			dfs2(e[tmp].to,e[tmp].to);
}

int deep[N<<5],deep2[N<<5],sum[N<<5];
int dsum[N<<5],d2sum[N<<5],tag[N<<5];
int root[N],lc[N<<5],rc[N<<5],tot=0,now=0,ptx=0;
struct TRD
{
	int a,b,c;
	TRD(int x=0,int y=0,int z=0):a(x),b(y),c(z){}
	friend TRD operator + (const TRD &x,const TRD &y){return TRD((x.a+y.a)%ha,(x.b+y.b)%ha,(x.c+y.c)%ha);}
};

inline void maintain(int o,int l,int r)
{
	sum[o]=(sum[lc[o]]+sum[rc[o]]+1ll*tag[o]*(r-l+1))%ha;
	dsum[o]=(dsum[lc[o]]+dsum[rc[o]]+1ll*tag[o]*deep[o])%ha;
	d2sum[o]=(d2sum[lc[o]]+d2sum[rc[o]]+1ll*tag[o]*deep2[o])%ha;
}

inline void copynode(int o,int p)
{
	sum[o]=sum[p];
	tag[o]=tag[p];
	deep[o]=deep[p];
	deep2[o]=deep2[p];
	dsum[o]=dsum[p];
	d2sum[o]=d2sum[p];
	lc[o]=lc[p];rc[o]=rc[p];
}

void Build(int &o,int l,int r)
{
	o=++tot;
	if(l==r)
	{
		sum[o]=val[idx[l]];
		deep[o]=dep[idx[l]];
		deep2[o]=1ll*deep[o]*deep[o]%ha;
		dsum[o]=1ll*deep[o]*sum[o]%ha;
		d2sum[o]=1ll*dsum[o]*deep[o]%ha;
		return;
	}
	int mid=(l+r)/2;
	Build(lc[o],l,mid);
	Build(rc[o],mid+1,r);
	maintain(o,l,r);
	deep[o]=(deep[lc[o]]+deep[rc[o]])%ha;
	deep2[o]=(deep2[lc[o]]+deep2[rc[o]])%ha;
}

void Modify(int &o,int pre,int l,int r,int nl,int nr,int k)
{
	if(!o) copynode(o=++tot,pre);
	if(l==nl&&r==nr)
	{
		sum[o]=(sum[o]+1ll*k*(nr-nl+1))%ha;
		dsum[o]=(dsum[o]+1ll*k*deep[o])%ha;
		d2sum[o]=(d2sum[o]+1ll*k*deep2[o])%ha;
		tag[o]=(tag[o]+k)%ha;
		return;
	}
	int mid=(l+r)/2;
	if(nr<=mid)
	{
		if(lc[o]==lc[pre]) lc[o]=0;
		Modify(lc[o],lc[pre],l,mid,nl,nr,k);
	}
	else if(nl>mid)
	{
		if(rc[o]==rc[pre]) rc[o]=0;
		Modify(rc[o],rc[pre],mid+1,r,nl,nr,k);
	}
	else
	{
		if(lc[o]==lc[pre]) lc[o]=0;
		Modify(lc[o],lc[pre],l,mid,nl,mid,k);
		if(rc[o]==rc[pre]) rc[o]=0;
		Modify(rc[o],rc[pre],mid+1,r,mid+1,nr,k);
	}
	maintain(o,l,r);
}

TRD Query(int o,int l,int r,int nl,int nr,int d)
{
	if(l==nl&&r==nr) return TRD((sum[o]+1ll*d*(r-l+1))%ha,(dsum[o]+1ll*d*deep[o])%ha,(d2sum[o]+1ll*d*deep2[o])%ha);
	int mid=(l+r)/2;d=(d+tag[o])%ha;
	if(nr<=mid) return Query(lc[o],l,mid,nl,nr,d);
	else if(nl>mid) return Query(rc[o],mid+1,r,nl,nr,d);
	else
	{
		TRD res1=Query(lc[o],l,mid,nl,mid,d);
		TRD res2=Query(rc[o],mid+1,r,mid+1,nr,d);
		return res1+res2;
	}
}

void PathModify(int u,int v,int d)
{
	ptx++;
	while(top[u]!=top[v])
	{
		if(dep[top[u]]<dep[top[v]]) swap(u,v);
		Modify(root[ptx],root[now],1,n,dfn[top[u]],dfn[u],d);
		u=fa[top[u]];
	}
	if(dep[u]>dep[v]) swap(u,v);
	Modify(root[ptx],root[now],1,n,dfn[u],dfn[v],d);
	now=ptx;
}

int getlca(int u,int v)
{
	while(top[u]!=top[v])
	{
		if(dep[top[u]]<dep[top[v]]) swap(u,v);
		u=fa[top[u]];
	}
	if(dep[u]>dep[v]) swap(u,v);
	return u;
}

int PathQuery(int u,int v)
{
	int lca=getlca(u,v),t2=dep[v];
	int t1=(dep[v]-2*dep[lca]+ha)%ha;
	int ans=0;TRD res;
	while(top[u]!=top[v])
	{
		if(dep[top[u]]>=dep[top[v]])
		{
			res=Query(root[now],1,n,dfn[top[u]],dfn[u],0);
			ans=(ans+1ll*res.a*(1ll*t1*t1%ha+t1))%ha;
			ans=(ans+1ll*res.b*(2ll*t1+1)+res.c)%ha;
			u=fa[top[u]];
		}
		else
		{
			res=Query(root[now],1,n,dfn[top[v]],dfn[v],0);
			ans=(ans+1ll*res.a*(1ll*t2*t2%ha+t2))%ha;
			ans=(ans-1ll*res.b*(2ll*t2+1)%ha+res.c+ha)%ha;
			v=fa[top[v]];
		}
	}
	if(dep[u]>=dep[v])
	{
		res=Query(root[now],1,n,dfn[v],dfn[u],0);
		ans=(ans+1ll*res.a*(1ll*t1*t1%ha+t1))%ha;
		ans=(ans+1ll*res.b*(2ll*t1+1)+res.c)%ha;
	}
	else
	{
		res=Query(root[now],1,n,dfn[u],dfn[v],0);
		ans=(ans+1ll*res.a*(1ll*t2*t2%ha+t2))%ha;
		ans=(ans-1ll*res.b*(2ll*t2+1)%ha+res.c+ha)%ha;
	}
	return 10080251ll*ans%ha;
}

int main()
{
	int u,v,opt,x,y,d,lastans=0;
	n=read();m=read();
	for(int i=1;i<n;i++)
	{
		u=read();v=read();
		add_edge(u,v);
		add_edge(v,u);
	}
	for(int i=1;i<=n;i++) val[i]=read();
	dep[1]=1;dfs1(1,0);dfs2(1,1);
	Build(root[0],1,n);
	while(m--)
	{
		opt=read();x=read();
		if(opt==3) now=x^lastans;
		if(opt==1)
		{
			y=read();d=read();
			x^=lastans;
			y^=lastans;
			PathModify(x,y,d);
		}
		if(opt==2)
		{
			y=read();
			x^=lastans;
			y^=lastans;
			lastans=PathQuery(x,y);
			print(lastans);
		}
	}
	flush();
	return 0;
}
```
