[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/FWT)

# 快速沃尔什变换(FWT) 学习笔记

FWT的算法，我并不是非常了解，但我知道它的作用，而且它代码很短，可以……（后面的话不说都懂吧）

### 提出问题

定义卷积：

![](http://latex.codecogs.com/svg.latex?c_k=\sum_{i\;xor\;j=k}a_ib_j)

现在给定数组a、b，他们的长度分别为n、m，下标从0开始，请你求出c数组

设k是2的次幂中第一个大于等于max(n,m)的数，易证c只可能有前k项非0，故输出c数组的前k项即可

### 解决问题

FWT是一类奇妙的变换，经过FWT之后，a与b可以对应位直接相乘，然后再IFWT回去就是答案

不同的卷积定义，写出的FWT代码也是不一样的，经过前人的试验，已经得出了xor、or、and的FWT代码，下面的模板是xor的，也就是上面那个问题的代码，or和and的代码也写在了下面模板的注释中，经过暴力对拍都是没问题的。~~背吧~~

```cpp
#include<bits/stdc++.h>
#define Mod 1000000007
#define inv2 500000004
using namespace std;

const int N=400010;
int a[N],b[N];

void FWT(int *a,int n,bool flag)
{
	for(int i=1;i<n;i<<=1)
		for(int j=0;j<n;j+=(i<<1))
			for(int k=0;k<i;k++)
			{
				int x=a[j+k],y=a[i+j+k];
				if(flag) a[j+k]=(x+y)%Mod,a[i+j+k]=(x-y+Mod)%Mod;
				//and: a[j+k]=x+y   or: a[i+j+k]=x+y
				else a[j+k]=1ll*(x+y)*inv2%Mod,a[i+j+k]=1ll*(x-y+Mod)*inv2%Mod;
				//and: a[j+k]=x-y   or: a[i+j+k]=y-x
			}
}

int main()
{
	int n,m;
	scanf("%d%d",&n,&m);
	for(int i=0;i<n;i++) scanf("%d",a+i);
	for(int i=0;i<m;i++) scanf("%d",b+i);
	int len;for(len=1;len<n||len<m;len<<=1);
	FWT(a,len,1);FWT(b,len,1);
	for(int i=0;i<len;i++) a[i]=1ll*a[i]*b[i]%Mod;
	FWT(a,len,0);
	for(int i=0;i<len;i++) printf("%d ",a[i]);
	return 0;
}
```
