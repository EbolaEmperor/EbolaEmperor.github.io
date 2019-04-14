[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/FFT)

# 快速傅里叶变换(FFT) 学习笔记

很早以前就想学这个东西，但一直学不懂，后来去研究了一下复数的性质，一下子就感觉明朗了很多

我们定义多项式A表示为![](http://latex.codecogs.com/svg.latex?A(x)=\sum_{i=0}^nc_ix^i)，快速傅里叶变换就是用来求两个多项式乘积的工具

为了表述方便，下面用n表示A的项数，m表示B的项数，s表示A*B的项数

暴力求两个多项式的乘积就是模拟，时间复杂度O(n²)

我们考虑用另一种方式来进行多项式乘法

多项式的另一种表达形式是点值表示法，一个n项的多项式可以由n个点唯一确定，这点可以用高斯消元来进行

假如我们把多项式A、B转化成点值表示法，那么它们的相乘就可以直接点值相乘，最后再变回系数表示法即可。注意A、B转换成点值表示法时需要取n+m个点，因为A*B是n+m项的

怎么把系数表示法转化成点值表示法？取s个值带入计算？那不就O(n²)了？糟糕！

怎么把点值表示法变回系数表示法？高斯消元？那不就是O(n³)了！糟糕透顶！

那么我们介绍一种新的方法——FFT

用FFT求两个多项式的乘积分为三步：DFT（系数表示法转点值表示法）、点值相乘、IDFT（点值表示法转系数表示法）

### First Step: DFT(离散傅里叶变换)

首先我们要知道复数相关的运算 [看这里](https://blog.csdn.net/qq_39670434/article/details/79678103)

DFT要求我们取的s个数满足![](http://latex.codecogs.com/svg.latex?x_k=\omega_s^k)

其中![](http://latex.codecogs.com/svg.latex?\omega_s^k)表示s单位复根的第k个（即把复平面单位圆上平均划分成s个位置，从(1,0)开始逆时针数第k个）

单位复根有两个奇妙的引理：倍增引理与折半引理。可以形象地理解为“约分”。就是说，单位复根上下标同时乘或除2，它的值不变

于是我们可以用它进行一些推导

在此之前，我们需要对多项式进行划分。试想将多项式A的奇次方项和偶次方项分开，分别记作A1与A2,不难得到一个关系：A(x)=x*A1(x²)+A2(x²)

考虑A的第k个点的x坐标，代入关系，根据折半引理得到：![](http://latex.codecogs.com/svg.latex?A(\omega_n^k)=A_2(\omega_{n/2}^k)+\omega_n^kA_1(\omega_{n/2}^k))

再考虑A的第k+n/2个点的x坐标：![](http://latex.codecogs.com/svg.latex?\omega_n^{k+n/2}=\omega_n^k*\omega_n^{n/2}=-\omega_n^k)

以及它的平方：![](http://latex.codecogs.com/svg.latex?(\omega_n^{k+n/2})^2=\omega_n^{2k+n}=\omega_n^{2k}*\omega_n^n=\omega_n^{2k}=\omega_{n/2}^k)

将其代入关系，得到：![](http://latex.codecogs.com/svg.latex?A(\omega_n^{k+n/2})=A_2(\omega_{n/2}^k)-\omega_n^kA_1(\omega_{n/2}^k))

发现第k个点代入关系得到的表达式非常相似，就是一个正负号的问题！

那么假如我们已经计算出来了A1与A2的点值表达，我们就可以枚举划分出的多项式的所有点（它的数量只有A的点的一半），然后代入上式计算，就可以得到A的所有点，非常划算！

那么我们递归求解就好了！

退出条件就是项数为1时，那一项系数即为y坐标值

需要注意，这样的划分要求一开始多项式的项数是2的幂次，我们可以在一开始的时候把缺的项系数补上0

### Second Step: Point Multiplication(点值乘法)

直接把多项式A、B所有点的y坐标乘起来即可

### Third Step: IDFT(逆离散傅里叶变换)

这一步非常奇妙，具体推导过程我也不会，但我知道最终推出的表达式：

![](http://latex.codecogs.com/svg.latex?c_i=\frac{1}{n}\sum_{k=0}^{n-1}y_k\omega_n^{-ki})

发现与DFT那步的柿子很像，只是前面多了个1/n，然后单位复根的上标多了个负号

这就允许我们把DFT和IDFT写到同一个函数里面，做IDFT的时候，单位复根的虚部乘上-1就好了，最后出来再把实部都除掉n

### 模板(递归版，简洁易懂，常数较大)

```cpp
#include<bits/stdc++.h>
#define vec vector<Comp>
#define pb push_back
using namespace std;

const double pi=acos(-1);
struct Comp
{
	double r,i;
	Comp(double x=0,double y=0):r(x),i(y){}
	friend Comp operator + (Comp a,Comp b){return Comp(a.r+b.r,a.i+b.i);}
	friend Comp operator - (Comp a,Comp b){return Comp(a.r-b.r,a.i-b.i);}
	friend Comp operator * (Comp a,Comp b){return Comp(a.r*b.r-a.i*b.i,a.r*b.i+a.i*b.r);}
};

vec A,B,C;

vec FFT(vec a,int v)
{
	int n=a.size();
	if(n==1) return a;
	vec a1,a2;
	for(int i=0;i<n;i++)
		if(i&1) a2.pb(a[i]);
		else a1.pb(a[i]);
	a1=FFT(a1,v);a2=FFT(a2,v);
	Comp wn=Comp(cos(2*pi/n),v*sin(2*pi/n));
	Comp w=Comp(1,0);
	for(int i=0;i<n/2;i++)
	{
		Comp x=a1[i],y=a2[i];
		a[i]=x+w*y;
		a[n/2+i]=x-w*y;
		w=w*wn;
	}
	return a;
}

int main()
{
	int n,m,fake;double x;
	scanf("%d%d%d",&n,&m,&fake);
	for(int i=0;i<=n;i++)
	{
		scanf("%lf",&x);
		A.pb(Comp(x,0));
	}
	for(int i=0;i<=m;i++)
	{
		scanf("%lf",&x);
		B.pb(Comp(x,0));
	}
	int now=1;
	while(now<=n+m) now<<=1;
	for(int i=n+1;i<now;i++) A.pb(Comp(0,0));
	for(int i=m+1;i<now;i++) B.pb(Comp(0,0));
	A=FFT(A,1);B=FFT(B,1);
	for(int i=0;i<now;i++) C.pb(A[i]*B[i]);
	C=FFT(C,-1);
	for(int i=0;i<=n+m;i++) printf("%d ",(int)round(C[i].r/now));
	return 0;
}
```

### 模板(非递归版，晦涩难懂，常数小，代码量小~~可以背~~)

```cpp
#include<iostream>
#include<cstdio>
#include<cmath>
#include<complex>
#include<cstdlib>
using namespace std;

const double pi=acos(-1);
struct Comp
{
	double r,i;
	Comp(double x=0,double y=0):r(x),i(y){}
	friend Comp operator + (Comp a,Comp b){return Comp(a.r+b.r,a.i+b.i);}
	friend Comp operator - (Comp a,Comp b){return Comp(a.r-b.r,a.i-b.i);}
	friend Comp operator * (Comp a,Comp b){return Comp(a.r*b.r-a.i*b.i,a.r*b.i+a.i*b.r);}
};
int n,m,l=0,r[3000000];
Comp a[3000000],b[3000000];

void FFT(Comp *a,int v)
{
	for(int i=0;i<n;i++) if(i<r[i]) swap(a[i],a[r[i]]);
	for(int i=1;i<n;i*=2)
	{
		Comp wn=Comp(cos(pi/i),v*sin(pi/i));
		int p=i*2;
		for(int j=0;j<n;j+=p)
		{
			Comp w=Comp(1,0);
			for(int k=0;k<i;k++)
			{
				Comp x=a[j+k],y=w*a[i+j+k];
				a[j+k]=x+y;a[i+j+k]=x-y;
				w=w*wn;
			}
		}
	}
}

int main()
{
	cin>>n>>m;
	for(int i=0;i<=n;i++) scanf("%lf",&a[i].r);
	for(int i=0;i<=m;i++) scanf("%lf",&b[i].r);
	m+=n;for(n=1;n<=m;n*=2) l++;
	for(int i=0;i<n;i++) r[i]=(r[i/2]/2)|((i&1)<<(l-1));
	FFT(a,1);FFT(b,1);
	for(int i=0;i<=n;i++) a[i]=a[i]*b[i];
	FFT(a,-1);
	for(int i=0;i<=m;i++) printf("%d ",(int)round(a[i].r/n));
	return 0;
}
```
