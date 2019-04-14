[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Mobius)

# 【国家集训队】Crash的数字表格 题解

先来画柿子！

![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/06/%E5%9B%BE%E7%89%873-1-768x502.png)

上面的sum(x)表示1+2+3+…+x

嗯这柿子这么大一定很好吃（划掉）

首先，由于F(T)是一个积性函数，因此，如果我们让一个数x乘上一个它原本没有的质因子p，记T=xp，则有F(T)=F(x)*F(p)

如果x乘上的质因子p是它本身有的，那么枚举质因子q时，除非q=p，否则q对F(T)的贡献一定为0，因为T/q必定包含了至少两个p，导致μ(T/q)=0。于是又得到一个结论：当i mid p=0，其中p是质数时，有F(T)=F(x)

有了上面两个结论，就可以线性筛预处理出F函数，sum可以用等差数列求和公式，当然也可以选择预处理出来，反正空间足够。然后就可以O(n)直接做了！

```cpp
#include<bits/stdc++.h>
#define Mod 20101009
using namespace std;

typedef long long LL;
const int N=10000000;
int prime[664579+10],tot=0;
bool mark[N+10];
LL f[N+10],sum[N+10];

void Init(int n)
{
	f[1]=1;
	for(int i=2;i<=n;i++)
	{
		if(!mark[i]) prime[++tot]=i,f[i]=((1-i)%Mod+Mod)%Mod;
		for(int j=1;j<=tot&&i*prime[j]<=n;j++)
		{
			mark[i*prime[j]]=1;
			if(i%prime[j]) f[i*prime[j]]=f[i]*f[prime[j]]%Mod;
			else f[i*prime[j]]=f[i];
		}
	}
	for(int i=1;i<=N;i++) f[i]=(f[i-1]+(LL)i*f[i]%Mod)%Mod;
	for(int i=1;i<=N;i++) sum[i]=(sum[i-1]+(LL)i)%Mod;
}

int main()
{
	int n,m,j;cin>>n>>m;
	if(n>m) swap(n,m);
	LL ans=0;Init(n);
	for(int i=1;i<=n;i=j+1)
	{
		j=min(n/(n/i),m/(m/i));
		ans=(ans+(sum[n/i]*sum[m/i]%Mod)*(f[j]-f[i-1]+Mod)%Mod)%Mod;
	}
	cout<<ans<<endl;
	return 0;
}
```
