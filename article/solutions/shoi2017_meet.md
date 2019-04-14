[返回首页](https://EbolaEmperor.github.io)
[返回专题](hrttps://EbolaEmperor.github.io/special/Eular)

# 【SHOI2017】相逢是问候 题解

欢迎收看《一个等号引发的悲剧》系列节目。扩展欧拉定理记成了>phi(p)才要+phi(p)，呵呵

首先，我们需要知道[扩展欧拉定理](https://blog.csdn.net/ez_yww/article/details/76176970)。那么利用这条定理，我们就可以知道幂需要模多少

先预处理出p的欧拉函数嵌套，base[i]表示phi(phi(...phi(p)...))(i个phi)

再预处理出c的一些次方，p[i][j]表示c^i对base[j]取模的值。但由于p比较大，我们需要预处理出c的10^8次方出来，显然时空爆炸。于是我们分块，p1[i][j]表示c^i对base[j]取模的值,p2[i][j]表示c^(i*10000)对base[j]取模的值，那么c^x mod base[y]就可以表示为p2[x/10000][y]*p1[x%10000][y]%base[y]。预处理时间复杂度O(10000 log p)

于是我们就可以暴力更新

每次更新操作，我们在每个位置上都进行一次重新计算。放心，log p次之后的计算是可以忽略的，因为它会在log p次计算之后趋于一个定值，因此时间复杂度有保证

求和操作需要一个线段树来维护一下。那之前的更新操作就变成了在线段树对应的的叶子结点上进行重新计算。

注意，扩展欧拉定理有两条，计算是需要根据情况选择使用哪一条。具体请看代码。

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=50010;
int pc[60];
int p1[10010][30];  // c^i mod base[j]
int p2[10010][30];  // c^(10000*i) mod base[j]
int base[30];  //phi[phi[...phi[p]...]]
int n,m,p,c;
int a[N];

inline int phi(int x)
{
	int s=x;
	for(int i=2;i*i<=x;i++)
		if(x%i==0)
		{
			s=s/i*(i-1);
			while(x%i==0) x/=i;
		}
	if(x!=1) s=s/x*(x-1);
	return s;
}

inline int Pow(int a,int b,int p)
{
	int ans=1;
	while(b)
	{
		if(b&1) ans=1ll*ans*a%p;
		a=1ll*a*a%p;
		b>>=1;
	}
	return ans;
}

inline void Init()
{
	base[0]=p;
	for(int i=1;i<=28;i++)
		base[i]=phi(base[i-1]);
	pc[0]=1;
	for(int i=1;i<=50;i++)
		if(pow(1ll*c,1ll*i)<=1e9) pc[i]=pc[i-1]*c;
		else pc[i]=-1;
	for(int i=0;i<=28;i++)
	{
		p1[0][i]=1%base[i];
		for(int j=1;j<=10000;j++)
			p1[j][i]=1ll*p1[j-1][i]*c%base[i];
		p2[0][i]=1%base[i];
		for(int j=1;j<=10000;j++)
			p2[j][i]=1ll*p2[j-1][i]*p1[10000][i]%base[i];
	}
}

inline int powc(int b,int x){return 1ll*p2[b/10000][x]*p1[b%10000][x]%base[x];}

int val[N<<2],power[N<<2];

inline void maintain(int o)
{
	val[o]=(val[o*2]+val[o*2+1])%p;
	power[o]=min(power[o*2],power[o*2+1]);
}

void build(int o,int l,int r)
{
	if(l==r)
	{
		val[o]=a[l];
		power[o]=0;
		return;
	}
	int mid=(l+r)/2;
	build(o*2,l,mid);
	build(o*2+1,mid+1,r);
	maintain(o);
}

void work(int o,int x)
{
	int now=a[x],s=a[x];
	power[o]++;
	for(int i=power[o]-1;i>=0;i--)
		if(now<0||now>=base[i+1])
		{
			now=-1;
			s=powc(s%base[i+1]+base[i+1],i);
		}
		else
		{
			now=(now<40)?pc[now]:-1;
			s=powc(s%base[i+1],i);
		}
	val[o]=s;
}

void Modify(int o,int l,int r,int nl,int nr)
{
	if(power[o]>27) return;
	if(l==r){work(o,l);return;}
	int mid=(l+r)/2;
	if(nl<=mid) Modify(o*2,l,mid,nl,nr);
	if(nr>mid) Modify(o*2+1,mid+1,r,nl,nr);
	maintain(o);
}

int Query(int o,int l,int r,int nl,int nr)
{
	if(l>=nl&&r<=nr) return val[o];
	int mid=(l+r)/2,res=0;
	if(nl<=mid) res+=Query(o*2,l,mid,nl,nr);
	if(nr>mid) res+=Query(o*2+1,mid+1,r,nl,nr);
	return res%p;
}

int main()
{
	scanf("%d%d%d%d",&n,&m,&p,&c);
	Init();
	for(int i=1;i<=n;i++) scanf("%d",a+i);
	build(1,1,n);
	int opt,l,r;
	for(int i=1;i<=m;i++)
	{
		scanf("%d%d%d",&opt,&l,&r);
		if(l>r) swap(l,r);
		if(opt==0) Modify(1,1,n,l,r);
		else printf("%d\n",Query(1,1,n,l,r));
	}
	return 0;
}
```
