[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Du)

# 杜教筛 学习笔记

### 导入

这种筛法首次出现于外国的什么鬼网站，由于杜瑜皓是将其引入中国的第一人，故得名“杜教筛”

网上介绍这种筛法往往扯上狄利克雷卷积，然后说一大堆没用的东西，我实在看不懂，就问了yxq巨佬，巨佬开口就先鄙视了一番网上的博客，然后花了不到5分钟就把我教会了qwq，太神了！

### 正文

嗯，来做一道题：求φ(n)的前缀和

什么？你觉得很简单？线性筛一下就做出来了？

那我告诉你n的范围是10^9呢……O(n)爆炸了！

于是我们要用复杂度更低的方法。

先来画柿子！

![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/06/%E5%9B%BE%E7%89%872-1-350x488.png)

然后就可以递归求解了！

什么？递归？i=2~n的枚举不是O(n)的吗？这复杂度明显更高了呀！

看到整除还不整除分块！

递归的退出条件？

线性筛出前N个的T(n)，然后当n<N时返回T(n)即可

至于要筛前多少个，可以自己调一下参数，调到你觉得最快就行了

如果你参数调的好，时间复杂度是可以达到O(n^(2/3))的

需要加一个优化。因为杜教筛的过程中会计算很多重复的东西，所以可以用一个蛤希表来记录已经计算过的值，以后算到直接调用就可以了。如果像我一样比较懒，蛤希表开map也可以

### 代码实现

下面的代码并没有加蛤希表优化

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=7000000;
int prime[N],tot=0;
LL phi[N+10];
bool mark[N+10];

void Init()
{
	phi[1]=1;
	for(int i=2;i<=N;i++)
	{
		if(!mark[i]) prime[++tot]=i,phi[i]=i-1;
		for(int j=1;j<=tot&&i*prime[j]<=N;j++)
		{
			mark[i*prime[j]]=1;
			if(i%prime[j]) phi[i*prime[j]]=(prime[j]-1)*phi[i];
			else phi[i*prime[j]]=prime[j]*phi[i];
		}
	}
	for(int i=1;i<=N;i++) phi[i]+=phi[i-1];
}

LL Sphi(LL n)
{
	if(n<=N) return phi[n];
	LL res=n*(n+1)/2,div;
	for(LL i=2;i<=n;i=div+1)
	{
		div=n/(n/i);
		res-=(div-i+1)*Sphi(n/i);
	}
	return res;
}

int main()
{
	Init();
	LL n;
	cin>>n;
	cout<<Sphi(n)<<endl;
	return 0;
}

/*
Description : 求sigma{phi(i)},i=1~n
Sample Input : 1000000000
Sample Output : 303963551173008414
HINT : n<=10^9
Time Limit : 1s
Memory Limit : 512MB
*/
```
