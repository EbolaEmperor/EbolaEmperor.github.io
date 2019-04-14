[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/LinearBasis)

# 【SCOI2016】幸运数字 题解

一句话题意：给定一棵带点权的树，求给定路径经过的所有点权的异或最大值

异或最大值，显然的线性基

于是不难想到倍增，维护所有点向上跳2的次方步路径上的点权构成的线性基

那么询问，就是把路径上的线性基合并，然后询问最大值

线性基的合并是一个非常暴力的过程，其实就是把一个线性基里面的所有基一个一个插入另一个线性基中，理论时间复杂度应该是log² n

那么再套上一层倍增，最终时间复杂度O(Q log³ n)？？？

然后加了一个玄学优化就A掉了？？？（玄学优化见代码注释）

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=20010;
struct Edge{int to,next;} e[N<<1];
int h[N],sum=0;
int mul[16][N],dep[N];

struct Base
{
	LL b[65];
	Base(){memset(b,0,sizeof(b));}
	void insert(LL x)
	{
		for(int i=60;i>=0;i--)
			if(x&(1ll<<i))
			{
				if(b[i]) x^=b[i];
				else{b[i]=x;break;}
			}
	}
	void merge(Base &a)
	{
		for(int i=60;i>=0;i--)
			if(a.b[i]) insert(a.b[i]);
			//The "if" made my program from 60s+ down to 30s-??? 
	}
	LL query()
	{
		LL ans=0;
		for(int i=60;i>=0;i--)
			if((ans^b[i])>ans) ans^=b[i];
		return ans;
	}
} num[16][N];

void add_edge(int u,int v)
{
	e[++sum].to=v;
	e[sum].next=h[u];
	h[u]=sum;
}

void dfs(int u,int fa)
{
	for(int j=1;j<=14;j++)
	{
		mul[j][u]=mul[j-1][mul[j-1][u]];
		num[j][u]=num[j-1][u];
		num[j][u].merge(num[j-1][mul[j-1][u]]);
	}
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
	{
		int v=e[tmp].to;
		if(v==fa) continue;
		mul[0][v]=u;
		dep[v]=dep[u]+1;
		dfs(v,u);
	}
}

Base LCA(int p1,int p2)
{
	Base ans;
	if(dep[p1]<dep[p2]) swap(p1,p2);
	for(int j=14;j>=0;j--)
		if(dep[mul[j][p1]]>=dep[p2])
		{
			ans.merge(num[j][p1]);
			p1=mul[j][p1];
		}
	for(int j=14;j>=0;j--)
		if(mul[j][p1]!=mul[j][p2])
		{
			ans.merge(num[j][p1]);
			p1=mul[j][p1];
			ans.merge(num[j][p2]);
			p2=mul[j][p2];
		}
	if(p1!=p2)
	{
		ans.merge(num[1][p1]);
		ans.merge(num[0][p2]);
	}
	else ans.merge(num[0][p1]);
	return ans;
}

int main()
{
	int n,q,u,v;LL x;
	scanf("%d%d",&n,&q);
	for(int i=1;i<=n;i++)
	{
		scanf("%lld",&x);
		num[0][i].insert(x);
	}
	for(int i=1;i<n;i++)
	{
		scanf("%d%d",&u,&v);
		add_edge(u,v);
		add_edge(v,u);
	}
	dep[1]=1;dfs(1,0);
	for(int i=1;i<=q;i++)
	{
		scanf("%d%d",&u,&v);
		Base lca=LCA(u,v);
		printf("%lld\n",lca.query());
	}
	return 0;
}
```
