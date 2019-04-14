[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Mobius)

# 【SDOI2014】数表 题解

几天前写的题目忘记写题解了……倒回来思考了半天才想起来

先假设没有不大于a这条恶心的限制，设f(d)表示d的约数和，约定n<=m，则我们要求的就是：

![](http://latex.codecogs.com/gif.latex?\sum_{i=1}^n\sum_{j=1}^mf(gcd(i,j)))

![](http://latex.codecogs.com/gif.latex?=\sum_{d=1}^nf(d)\sum_{i=1}^n\sum_{j=1}^m[gcd(i,j)==d])

![](http://latex.codecogs.com/gif.latex?=\sum_{d=1}^nf(d)\sum_{i=1}^{\left%20\lfloor%20\frac{n}{d}%20\right%20\rfloor}\sum_{j=1}^{\left%20\lfloor%20\frac{m}{d}%20\right%20\rfloor}[gcd(i,j)==1])

![](http://latex.codecogs.com/gif.latex?=\sum_{d=1}^nf(d)\sum_{i=1}^{\left%20\lfloor%20\frac{n}{d}%20\right%20\rfloor}\sum_{j=1}^{\left%20\lfloor%20\frac{m}{d}%20\right%20\rfloor}\sum_{t%20\vert%20gcd(i,j)}\mu%20(t))

![](http://latex.codecogs.com/gif.latex?=\sum_{d=1}^nf(d)\sum_{t=1}^{\left%20\lfloor%20\frac{n}{d}%20\right%20\rfloor}\mu%20(t)%20\left\lfloor%20\frac{n}{dt}%20\right\rfloor%20\left\lfloor%20\frac{m}{dt}%20\right\rfloor)

设T=dt，得到：

![](http://latex.codecogs.com/gif.latex?\sum_{T=1}^n%20\left\lfloor%20\frac{n}{T}%20\right\rfloor%20\left\lfloor%20\frac{m}{T}%20\right\rfloor%20\sum_{d%20\vert%20T}\mu%20\left%20(\frac{T}{d}%20\right%20)%20f(d))

于是可以设 ![](http://latex.codecogs.com/gif.latex?h(T)=\sum_{d%20\vert%20T}\mu&space;\left&space;(\frac{T}{d}&space;\right&space;)&space;f(d)) ，那么求出h的前缀和就可以整除分块做了

f数组可以线性筛预处理出，当然你要是像我一样懒，写一个O(n log n)的暴力搞出f数组也行

现在来考虑不大于a这条限制

我们可以把所有询问离线，然后按a的大小排序，优先处理a较小的询问。再把f数组离线，按值升序排序，并保存下标。然后用一个树状数组来维护h的前缀和。每次处理一个询问，对于所有不大于当前a的f(d)，把u(T/d)f(d)给加到h(T)里面去。这一步骤用筛法来做就是：枚举i，对于id<N的情况，让h(id)加上u(i)f(d)。

然后整除分块时就可以直接在树状数组里求h的前缀和

时间复杂度 ![](http://latex.codecogs.com/gif.latex?O(Q\sqrt{n}log&space;n))

```cpp
#include<bits/stdc++.h>
#define INF 0x7fffffff
#define lowbit(x) (x&-x)
using namespace std;

const int N=100000;
struct QRY{int n,m,a,id;} qry[20010];
pair<int,int> f[N+10];
int prime[9592+10],tot=0;
int mu[N+10];
bool mark[N+10];
int bit[N],ans[N];

void Add(int p,int x){for(;p<=N;p+=lowbit(p))bit[p]+=x;}
int Sum(int p){int res=0;for(;p>0;p-=lowbit(p))res+=bit[p];return res;}
bool operator < (const QRY &a,const QRY &b){return a.a<b.a;}

void Init()
{
	mu[1]=1;
	for(int i=2;i<=N;i++)
	{
		if(!mark[i]) prime[++tot]=i,mu[i]=-1;
		for(int j=1;j<=tot&&i*prime[j]<=N;j++)
		{
			mark[i*prime[j]]=1;
			if(i%prime[j]) mu[i*prime[j]]=-mu[i];
			else mu[i*prime[j]]=0;
		}
	}
	for(int i=1;i<=N;i++)
		for(int j=i;j<=N;j+=i)
			f[j].first+=i;
	for(int i=1;i<=N;i++) f[i].second=i;
}

int main()
{
	Init();
	int Q;
	scanf("%d",&Q);
	for(int i=1;i<=Q;i++)
	{
		scanf("%d%d%d",&qry[i].n,&qry[i].m,&qry[i].a);
		qry[i].id=i;
	}
	sort(qry+1,qry+1+Q);
	sort(f+1,f+1+N);
	int pos=1;
	for(int T=1;T<=Q;T++)
	{
		while(pos<=N&&f[pos].first<=qry[T].a)
		{
			int k=f[pos].second;
			for(int j=k;j<=N;j+=k)
				Add(j,f[pos].first*mu[j/k]);
			pos++;
		}
		int div,n=qry[T].n,m=qry[T].m,res=0;
		if(n>m) swap(n,m);
		for(int i=1;i<=n;i=div+1)
		{
			div=min(n/(n/i),m/(m/i));
			res+=(n/i)*(m/i)*(Sum(div)-Sum(i-1));
		}
		ans[qry[T].id]=res&INF;
	}
	for(int i=1;i<=Q;i++) printf("%d\n",ans[i]);
	return 0;
}
```
