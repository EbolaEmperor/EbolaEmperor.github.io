# 【HEOI2016】求和  题解

首先你要有信仰，你要相信原式中的j是可以枚举到n的。原因很简单，当j>i时，S(i,j)=0，所以对答案不造成贡献。为什么要这么做，后面就会知道

你要知道第二类斯特林数展开这种东西。也就是$\left\{\begin{aligned}n\\m\end{aligned}\right\}m!=\sum\limits_{i=0}^m(-1)^i\left(\begin{aligned}m\\i\end{aligned}\right)(m-i)^n$

然后把它代入原式，得到：$f(n)=\sum\limits_{i=0}^n\sum\limits_{j=0}^n2^j\sum\limits_{k=0}^j(-1)^k\left(\begin{aligned}j\\k\end{aligned}\right)(j-k)^i$

把组合数公式代入，得到：$f(n)=\sum\limits_{i=0}^n\sum\limits_{j=0}^n2^j\sum\limits_{k=0}^j(-1)^k\frac{j!}{k!(j-k)!}(j-k)^i$

把$j!$提到前面来，把$\sum\limits_{i=0}^n$丢到后面去（这就是为什么一开始要把j枚举到n，如果不这么干，枚举i是不能丢到后面去的），得到：$\sum\limits_{j=0}^n2^jj!\sum\limits_{k=0}^j\frac{(-1)^k}{k!}\times\frac{\sum\limits_{i=0}^n(j-k)^i}{(j-k)!}$

接下来我们定义两个函数：$A(x)=\frac{(-1)^k}{k!}\qquad B(x)=\frac{\sum_{i=0}^nx^i}{k!}$

代入原式，得到：$f(n)=\sum\limits_{j=0}^n2^jj!\sum\limits_{k=0}^jA(k)B(j-k)$

哇，后面这个求和，居然是美妙的卷积！于是我们定义$C(k)=\sum\limits_{x+y=k}A(x)B(y)$，然后原式就可以化为$f(n)=\sum\limits_{j=0}^n2^jj!C(j)$

预处理阶乘及阶乘逆元，$A$就可以在$O(n)$的时间内算出。至于$B$的话，用一下等比数列求和公式，也可以在$O(n)$的时间内算出。然后用$FFT$对$A,B$做一下多项式乘法，可以在$O(n\;\log\;n)$的时间内求出$C$。再预处理出$2$的所有次幂在模意义下的值，最后就可以$O(n)$算出$f(n)$

时间复杂度瓶颈在于$FFT$，最终总的复杂度就是$O(n\;\log\;n)$

```cpp
#include<bits/stdc++.h>
#define ha 998244353
using namespace std;

const int N=270010;
int A[N],B[N],C[N],r[N];
int fac[N],ifac[N],pw[N];

int Pow(int a,int b)
{
	int ans=1;
	for(;b;b>>=1,a=1ll*a*a%ha)
		if(b&1) ans=1ll*ans*a%ha;
	return ans;
}

void NTT(int *A,int n,bool IDFT)
{
	for(int i=0;i<n;i++) r[i]=(r[i>>1]>>1)|((i&1)*(n/2));
	for(int i=0;i<n;i++) if(i<r[i]) swap(A[i],A[r[i]]);
	for(int i=1;i<n;i<<=1)
	{
		int wn=Pow(3,(ha-1)/(i<<1));
		if(IDFT) wn=Pow(wn,ha-2);
		for(int j=0;j<n;j+=(i<<1))
			for(int k=0,w=1;k<i;k++)
			{
				int x=A[j+k],y=1ll*w*A[i+j+k]%ha;
				A[j+k]=(x+y)%ha;
				A[i+j+k]=(x-y+ha)%ha;
				w=1ll*w*wn%ha;
			}
	}
	int inv=IDFT?Pow(n,ha-2):0;
	if(IDFT) for(int i=0;i<n;i++) A[i]=1ll*A[i]*inv%ha;
}

void InitFac(int n)
{
	fac[0]=1;
	for(int i=1;i<=n;i++)
		fac[i]=1ll*fac[i-1]*i%ha;
	ifac[n]=Pow(fac[n],ha-2);
	for(int i=n-1;i>=0;i--)
		ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
}

void InitABC(int n)
{
	for(int i=0;i<=n;i++)
		A[i]=1ll*(i&1?ha-1:1)*ifac[i]%ha;
	B[0]=1;B[1]=n+1;
	for(int i=2;i<=n;i++)
		B[i]=1ll*(Pow(i,n+1)-1)*Pow(i-1,ha-2)%ha*ifac[i]%ha;
	int len;
	for(len=1;len<=(n<<1);len<<=1);
	NTT(A,len,0);NTT(B,len,0);
	for(int i=0;i<len;i++)
		C[i]=1ll*A[i]*B[i]%ha;
	NTT(C,len,1);
}

int main()
{
	int n,ans=0;
	scanf("%d",&n);
	InitFac(n);
	InitABC(n);
	pw[0]=1;
	for(int i=1;i<=n;i++)
		pw[i]=2ll*pw[i-1]%ha;
	for(int i=0;i<=n;i++)
		ans=(ans+1ll*pw[i]*fac[i]%ha*C[i])%ha;
	printf("%d\n",ans);
	return 0;
}
```

