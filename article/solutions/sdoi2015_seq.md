[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/FFT)

# 【SDOI2015】序列统计 题解

如果集合s里的数相加，我们是可以搞出生成函数来的，但现在问题是相乘

将乘变成加的运算只有一种：对数

那么我们可以求出M的原根g，然后利用离散对数的定义把乘变成加

约定一下，下面所说的log都指以g为底的离散对数

生成函数：![](http://latex.codecogs.com/svg.latex?F(x)=\sum_{i=0}^mc_ix^i)，其中c[i]表示log(i)是否在集合S中

那么该生成函数n次幂的第log(x)项系数就是答案

注意运算过程中不仅系数要取模，次数也要取模。具体地，我们把所有次数为r(r>=m)的项的系数加到r-m里面去

其实本来可以毒瘤一下搞什么多项式求ln、exp什么的，但因为次数要取模，所以只能用快速幂

```cpp
#include<bits/stdc++.h>
#define Mod 1004535809
using namespace std;

const int N=40010;
int a[N],c[N],ans[N],r[N],l=0,len,inv;
int q[8010],cnt=0,lg[8010],m;

int Pow(int a,int b,int p)
{
	int ans=1;
	for(;b;b>>=1,a=1ll*a*a%p)
		if(b&1) ans=1ll*ans*a%p;
	return ans;
}

void NTT(int *a,bool INTT)
{
	for(int i=0;i<len;i++) if(i<r[i]) swap(a[i],a[r[i]]);
	for(int i=1;i<len;i<<=1)
	{
		int p=(i<<1);
		int wn=Pow(3,(Mod-1)/p,Mod);
		if(INTT) wn=Pow(wn,Mod-2,Mod);
		for(int j=0;j<len;j+=p)
		{
			int w=1;
			for(int k=0;k<i;k++)
			{
				int x=a[j+k],y=1ll*w*a[i+j+k]%Mod;
				a[j+k]=(x+y)%Mod;
				a[i+j+k]=(x-y+Mod)%Mod;
				w=1ll*w*wn%Mod;
			}
		}
	}
	if(INTT) for(int i=0;i<len;i++) a[i]=1ll*a[i]*inv%Mod;
}

int Root(int s)
{
	for(int i=2;i<=s-2;i++)
		if((s-1)%i==0) q[++cnt]=i;
	for(int i=2;;i++)
	{
		bool flag=1;
		for(int j=1;j<=cnt&&flag;j++)
			if(Pow(i,q[j],s)==1) flag=0;
		if(flag) return i;
	}
	return -1;
}

void Mult(int *a,int *b)
{
	for(int i=0;i<len;i++) c[i]=b[i];
	NTT(a,0);NTT(c,0);
	for(int i=0;i<len;i++)
		a[i]=1ll*a[i]*c[i]%Mod;
	NTT(a,1);
	for(int i=len-1;i>=m-1;i--)
		a[i-m+1]=(a[i-m+1]+a[i])%Mod,a[i]=0;
}

int main()
{
	int n,x,S,k;
	scanf("%d%d%d%d",&n,&m,&x,&S);
	for(len=1;len<(m<<1);len<<=1) l++;
	for(int i=0;i<len;i++) r[i]=(r[i/2]/2)|((i&1)<<(l-1));
	inv=Pow(len,Mod-2,Mod);
	int g=Root(m),pw=1;
	for(int i=0;i<=m-2;i++)
		lg[pw]=i,pw=pw*g%m;
	for(int i=0;i<S;i++)
	{
		scanf("%d",&k);
		if(k) a[lg[k]]++;
	}
	int b=n;ans[0]=1;
	while(b)
	{
		if(b&1) Mult(ans,a);
		Mult(a,a);
		b>>=1;
	}
	printf("%d\n",ans[lg[x]]);
	return 0;
}
```
