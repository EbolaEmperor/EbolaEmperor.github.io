[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/FFT)

# 毛啸理论变换(MTT) 学习笔记

毛啸理论变换，又称拆系数FFT，是任意模数FFT的一种实现方式（另一种实现方式是多模数NTT），其英文全称是 Mathew Theoretic Transform ，因其由毛啸(Mathew，IOI2017中国国家队队员，现就读麻省理工)发明而得名

NTT运算过程中，可以边运算边取模，但只能针对特殊模数。FFT计算过程中，使用的是浮点数，因此不能边运算边取模，如果需要取模，只能在最后的IDFT步骤完成之后对结果取模

如果系数不大，FFT最后再来取模是没有问题的，但如果系数比较大，运算过程中就会不可避免地进行大数运算，这时浮点数的精度误差就会彻底地体现出来

MTT做的事，就是把大系数变小，取之而来的是精确的答案与较大的时间常数

考虑多项式系数的拆分。我们令多项式A(x)=m\*A1(x)+A2(x)，显然，只要A1各项系数等于A各项系数除m（下取整），A2各项系数等于A各项系数模m，这个等式就是成立的。那么只要我们的m取值恰当，A1与A2的系数就都不会太大

对相乘的两个多项式都进行这样的拆分，于是最终的结果C=A\*B=(m\*A1+A2)\*(m\*B1+B2)=A1\*B1\*m²+(A1\*B2+A2\*B1)\*m+A2\*B2，直接FFT即可，由于需要FFT的多项式由2个变成了4个，所以算法的常数大了一倍

```cpp
#include<bits/stdc++.h>
#define double long double
using namespace std;

typedef long long LL;
struct Comp
{
	double r,i;
	Comp(double x=0,double y=0):r(x),i(y){}
	friend Comp operator + (Comp a,Comp b){return Comp(a.r+b.r,a.i+b.i);}
	friend Comp operator - (Comp a,Comp b){return Comp(a.r-b.r,a.i-b.i);}
	friend Comp operator * (Comp a,Comp b){return Comp(a.r*b.r-a.i*b.i,a.r*b.i+a.i*b.r);}
};
const int N=400010,M=32768;
const double pi=acos(-1);
Comp a1[N],b1[N],c1[N],d1[N];
Comp a2[N],b2[N],c2[N],d2[N];
int n,m,len=1,p,l=0,r[N];
int A[N],B[N];
LL ans[N];

void FFT(Comp *a,int v)
{
	for(int i=0;i<len;i++) if(i<r[i]) swap(a[i],a[r[i]]);
	for(int i=1;i<len;i<<=1)
	{
		Comp wn=Comp(cos(pi/i),v*sin(pi/i));
		int p=(i<<1);
		for(int j=0;j<len;j+=p)
		{
			Comp w=Comp(1,0);
			for(int k=0;k<i;k++)
			{
				Comp x=a[j+k],y=a[i+j+k];
				a[j+k]=x+w*y;
				a[i+j+k]=x-w*y;
				w=w*wn; 
			} 
		}
	}
}

void MTT()
{
	for(int i=0;i<=n+m;i++)
	{
		a1[i]=Comp(A[i]/M,0);
		b1[i]=Comp(A[i]%M,0);
		c1[i]=Comp(B[i]/M,0);
		d1[i]=Comp(B[i]%M,0);
	}
	FFT(a1,1);FFT(b1,1);FFT(c1,1);FFT(d1,1);
	for(int i=0;i<len;i++)
	{
		a2[i]=a1[i]*c1[i];
		b2[i]=a1[i]*d1[i];
		c2[i]=b1[i]*c1[i];
		d2[i]=b1[i]*d1[i];
	}
	FFT(a2,-1);FFT(b2,-1);FFT(c2,-1);FFT(d2,-1);
	for(int i=0;i<len;i++)
	{
		ans[i]=(LL)round(a2[i].r/len)%p*M%p*M%p;
		ans[i]+=(LL)round(b2[i].r/len)%p*M%p;
		ans[i]+=(LL)round(c2[i].r/len)%p*M%p;
		ans[i]+=(LL)round(d2[i].r/len)%p;
		ans[i]%=p;
	}
}

int main()
{
	scanf("%d%d%d",&n,&m,&p);
	for(int i=0;i<=n;i++) scanf("%d",A+i);
	for(int i=0;i<=m;i++) scanf("%d",B+i);
	for(len=1;len<=n+m;len<<=1) l++;
	for(int i=0;i<len;i++) r[i]=(r[i/2]/2)|((i&1)<<(l-1));
	MTT();
	for(int i=0;i<=n+m;i++) printf("%lld ",ans[i]);
	return 0;
}
```
