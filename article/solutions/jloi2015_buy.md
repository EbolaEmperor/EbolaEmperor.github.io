[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/LinearBasis)

# 【JLOI2015】装备购买 题解

嗯，吉林OI，基佬OI。搞不懂这吉林人跑去省选怎么就忽然变基佬了

简单的说，这题就是要你选择m维空间基底向量，选择每个向量都要付出一定代价，最小化这个代价

容易想到线性基+贪心
若代价小的向量可以加入线性基，我们就贪心地选择它

于是可以给定向量按代价从小到大排序，顺次枚举，高斯消元法插入即可

具体插入方法见 [线性基 学习笔记](https://EbolaEmperor.github.io/article/notes/LinearBasis)（虽然也不是很具体）

此外，据说这题卡精度，于是我写了模意义下的高斯消元，全程不用浮点数，爽！

``` cpp
#include<bits/stdc++.h>
#define Mod 998244353
using namespace std;

int inv(int a)
{
	int ans=1;
	for(int b=Mod-2;b;a=1ll*a*a%Mod,b>>=1)
		if(b&1) ans=1ll*ans*a%Mod;
	return ans;
}

int n,m,ans=0,tot=0;
struct Base
{
	int ba[510],w;
	Base(){memset(ba,0,sizeof(ba));}
	int& operator [] (const int &x){return ba[x];}
	void read(){for(int i=1;i<=m;i++)scanf("%d",ba+i);}
	bool empty()
	{
		bool flag=1;
		for(int i=1;i<=m;i++)
			if(ba[i]){flag=0;break;}
		return flag;
	}
} a[510],b[510];
bool operator < (const Base &x,const Base &y){return x.w<y.w;}

void insert(Base &a)
{
	for(int i=1;i<=m;i++)
	{
		if(!a[i]) continue;
		if(b[i].empty())
		{
			b[i]=a;tot++;
			ans+=a.w;return;
		}
		int k=1ll*a[i]*inv(b[i][i])%Mod;
		for(int j=1;j<=m;j++)
			a[j]=(a[j]-1ll*b[i][j]*k%Mod+Mod)%Mod;
	}
}

int main()
{
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;i++) a[i].read();
	for(int i=1;i<=n;i++) scanf("%d",&a[i].w);
	sort(a+1,a+1+n);
	for(int i=1;i<=n;i++) insert(a[i]);
	printf("%d %d\n",tot,ans);
	return 0;
}

```
