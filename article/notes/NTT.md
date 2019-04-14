[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/FFT)

# 快速数论变换(NTT) 学习笔记

在OI中，我们常常需要对一些结果取模，由于FFT运算过程使用的是浮点数，故并不支持运算过程中取模。NTT就是一种支持在运算过程中取模的东西，但需要注意的是，它只针对特殊模数有效。

NTT模数形如：m=a\*2^x+1，其中a是奇数，且m是质数。一般题目中用到的都是998244353(=7\*17\*2^23+1)，也有些题目会用1004535809(=479\*2^21+1)

了解一个概念：原根。它是指一个数g，满足g的0~p-1次方在模p意义下互不相同

上面提到的两个典型NTT模数的原根均为3

它就可以代替FFT中的单位复根

因为给定多项式都被我们填充成了2的幂次项，故(m-1)总是可以整除2i(这里的i是FFT中枚举的长度)，那么我们就可以依据这个原理，把(m-1)给分成2i份，这就像FFT中把复平面单位圆分成2i份一样

具体也不太好解释，反正把NTT的原根类比FFT的单位复根处理就好了

与FFT相比而言，NTT全程使用int，没有浮点误差，常数小

### 模板

```cpp
#include<bits/stdc++.h>
#define Mod 998244353
using namespace std;

const int g=3;
const int N=400010;
int a[N],b[N],r[N];
int l=0,n,m,len,inv;

int Pow(int a,int b)
{
	int ans=1;
	while(b)
	{
		if(b&1) ans=1ll*ans*a%Mod;
		a=1ll*a*a%Mod;
		b>>=1;
	}
	return ans;
}

void NTT(int *a,bool INTT)
{
	for(int i=0;i<len;i++) if(i<r[i]) swap(a[i],a[r[i]]);
	for(int i=1;i<len;i<<=1)
	{
		int p=(i<<1);
		int wn=Pow(g,(Mod-1)/p);
		if(INTT) wn=Pow(wn,Mod-2);
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

int main()
{
	int fake;
	scanf("%d%d%d",&n,&m,&fake);
	for(int i=0;i<=n;i++) scanf("%d",a+i);
	for(int i=0;i<=m;i++) scanf("%d",b+i);
	for(len=1;len<=n+m;len<<=1) l++;
	inv=Pow(len,Mod-2);
	for(int i=0;i<len;i++) r[i]=(r[i/2]/2)|((i&1)<<(l-1));
	NTT(a,0);NTT(b,0);
	for(int i=0;i<len;i++) a[i]=1ll*a[i]*b[i]%Mod;
	NTT(a,1);
	for(int i=0;i<=n+m;i++) printf("%d ",a[i]);
	return 0;
}
```
