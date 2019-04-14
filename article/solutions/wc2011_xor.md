[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/LinearBasis)

# 【WC2011】最大XOR和路径 题解

我们把选出的路径看做一条1到n的简单路径+一些环

简单路径可以任取一条，就算我们选出的这条不是最优解的路径，我们也可以认为，我们走这条路径到了n，又走最优解的路径回到1，然后再走这条路径到n，这样其实就是这条路径+一个环，异或一下就抵消了这条路径

那么对于一个不直接与这条路径联通的环，我们也可以认为这个环可以异或到答案里面。因为从这条路一条分岔出去到这个环，然后再原路返回，那从这条路到环的那段分岔异或一下就抵消掉了，所以可以直接异或这个环

那么解题思路就是，预处理出所有的环，将其加入线性基中。然后再计算出1到n的任意一条简单路径，长度记为d，我们在线性基中询问与d异或的最大值即可

预处理环与计算简单路径可以一并在dfs中完成

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;

const int N=50010,M=100010;
struct Edge{int to,next;LL capa;} e[M<<1];
int h[N],sum=0,n,m;
bool vis[N];
LL dis[N],base[70];

void add_edge(int u,int v,LL w)
{
	e[++sum].to=v;
	e[sum].capa=w;
	e[sum].next=h[u];
	h[u]=sum;
}

void insert(LL x)
{
	for(int i=60;i>=0;i--)
		if(x&(1ll<<i))
		{
			if(base[i]) x^=base[i];
			else{base[i]=x;break;}
		}
}

void dfs(int u)
{
	vis[u]=1;
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
	{
		int v=e[tmp].to;
		if(vis[v]) insert(dis[u]^dis[v]^e[tmp].capa);
		else dis[v]=dis[u]^e[tmp].capa,dfs(v);
	}
}

LL query(LL x)
{
	for(int i=60;i>=0;i--)
		if((x^base[i])>x) x^=base[i];
	return x;
}

int main()
{
	int u,v;LL w;
	scanf("%d%d",&n,&m);
	for(int i=1;i<=m;i++)
	{
		scanf("%d%d%lld",&u,&v,&w);
		add_edge(u,v,w);
		add_edge(v,u,w);
	}
	dfs(1);
	printf("%lld\n",query(dis[n]));
	return 0;
}
```
