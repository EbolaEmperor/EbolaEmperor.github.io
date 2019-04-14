# 【NOI2018】屠龙勇士  题解

首先可以预处理出对每条龙的攻击力，我们只要一个能支持插入、查找、删除的数据结构即可。这个可以使用STL的multiset

考虑一条龙。根据题意可以列出不定方程![](http://latex.codecogs.com/svg.latex?Ax+py=a)，其中A表示对这条龙的攻击力，p表示这条龙的恢复能力，a表示这条龙的初始血量。如果有某条龙的不定方程无解，那么就输出-1。将第i条龙求出的x值记作x[i]

根据不定方程的通解公式，每条龙都可以列出这样的一个同余方程：

![](http://latex.codecogs.com/svg.latex?x\equiv&space;x_i\quad(mod\;\;\frac{p_i}{\gcd(A_i,p_i)}))

然后就是要解一个同余方程组。由于模数不都是质数，我们使用扩展中国剩余定理求解。如果同余方程组无解，那么就输出-1

解完同余方程组还没完。回归到问题本身，我们要把龙给打死，而我们求出的最小攻击次数，可能不能将龙全都打到负血。举一个例子来说，攻击力为3，恢复力为2，血量为7，此时根据我们的方法求出的答案就是1，这时不定方程的确满足了，但不能把龙打死。所以解完整个同余方程组后，要对每条龙都判定一下能不能打死，如果打不死，还得让x加上lcm的一个倍数使得能把它打死（这里的lcm指同余方程组所有模数的lcm，并不是数据范围里面的那个lcm）

计算过程中会出现爆long long的问题，所以要在某些地方使用快速乘

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=100010;
int gt[N],atk[N];
LL a[N],p[N];
multiset<LL> A;
multiset<LL>::iterator it;
struct Equ{LL b,m;} equ[N];

LL ExGcd(LL a,LL b,LL &x,LL &y)
{
	if(b==0){x=1;y=0;return a;}
	LL g=ExGcd(b,a%b,x,y);
	LL t=x;x=y;y=t-(a/b)*y;
	return g;
}

LL Mul(LL a,LL b,LL p)
{
	LL ans=0,fg=1;
	if(b<0) fg=-1,b=-b;
	for(;b;b>>=1,a=(a+a)%p)
		if(b&1) ans=(ans+a)%p;
	return ans*fg;
}

bool ExCRT(Equ &a1,Equ a2)
{
	LL k1,k2,g;
	g=ExGcd(a1.m,a2.m,k1,k2);
	LL c=a2.b-a1.b;
	if(c%g) return 0;
	LL t=a2.m/g;
	k1=(Mul(k1,c/g,t)+t)%t;
	LL lcm=a1.m/g*a2.m;
	LL x=Mul(k1,a1.m,lcm)+a1.b;
	a1=Equ{x%lcm,lcm};
	return 1;
}

LL gao(int n)
{
	for(int i=1;i<=n;i++)
	{
		LL x,y,g;
		g=ExGcd(atk[i],p[i],x,y);
		if(a[i]%g) return -1;
		LL m=p[i]/g;
		x=(Mul(x,a[i]/g,m)+m)%m;
		equ[i]=Equ{x,m};
	}
	Equ now=equ[1];
	for(int i=2;i<=n;i++)
	{
		bool ok=ExCRT(now,equ[i]);
		if(!ok) return -1;
	}
	LL x=now.b,add=now.m;
	for(int i=1;i<=n;i++)
		if(x*atk[i]<a[i]) x+=add*(((a[i]-x*atk[i]-1)/atk[i])/add+1);
	return x;
}

int main()
{
	int T,n,m,x;
	for(scanf("%d",&T);T;T--)
	{
		scanf("%d%d",&n,&m);A.clear();
		for(int i=1;i<=n;i++) scanf("%lld",a+i);
		for(int i=1;i<=n;i++) scanf("%lld",p+i);
		for(int i=1;i<=n;i++) scanf("%d",gt+i);
		for(int i=1;i<=m;i++) scanf("%d",&x),A.insert(x);
		for(int i=1;i<=n;i++)
		{
			it=A.lower_bound(a[i]);
			if(it!=A.begin()&&(it==A.end()||*it!=a[i])) it--;
			atk[i]=*it;
			A.erase(it);
			A.insert(gt[i]);
		}
		printf("%lld\n",gao(n));
	}
}
```

