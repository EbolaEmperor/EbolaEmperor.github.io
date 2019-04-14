[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/KMP)

# 【NOI2014】动物园 题解

这题正解的时间复杂度是O(Tn)，然而我的O(Tn log n)居然过去了……（某巨佬：哪来的log？？？）

我们看一下这个num数组的含义到底是什么

首先忽略“不可重叠”这个限制，那么就是“与某个前缀相同的后缀的数量（注意是数量，不是长度）”。发现这就是连续失配的最大次数，也就是沿next指针跳到0所需步数

那么我们可以先沿next指针预处理出num数组。具体地，对于每一个位置p，我们设num[p]=num[p->next]+1

接下来我们考虑“不可重叠”这个限制

不难想到，“不可重叠”的含义，其实就是失配位置不能超过当前位置的一半

那么我们可以沿next指针上跳，跳到第一个不超过当前位置p一半的位置q时停止，那么就有num[p]=num[q]+1

这样只能拿50分，因为上跳过程过于暴力，容易被卡成O(Tn²)

那么我们就对这个上跳过程进行优化，不难想到使用倍增

将next数组建成一个倍增数组，这样往上跳就是O(log n)的了，总时间复杂度变成了O(Tn log n)

可是只能拿80分？？？按理说T和n的这个范围，带log的算法应该也是能A的呀？

我们考虑利用二维数组的特性，进行一些底（xuán）层（xué）优化

多维数组的一个特性就是：将长度较短的维度放在前面，会比放在后面要快，因为这样就会有更多的数据存储在寄存器中，而不是内存中，使得访问速度大大提升

于是我们将倍增数组的倍增维度放到点维度的前面，也就是next[k][p]表示从位置p上跳2^k次

然后就AC了！原来TLE的点只要500+ms就过了！

```cpp
#include<bits/stdc++.h>
#define Mod 1000000007
using namespace std;

typedef long long LL;
const int N=1000010;
int nxt[20][N],num[N];
char s[N];

int main()
{
	int n;
	scanf("%d",&n);
	while(n--)
	{
		scanf("%s",s+1);
		int l=strlen(s+1),j=0;
		LL ans=1;
		memset(nxt,0,sizeof(nxt));
		memset(num,0,sizeof(num));
		num[0]=0;num[1]=1;
		for(int i=2;i<=l;i++)
		{
			while(j&&s[i]!=s[j+1]) j=nxt[0][j];
			if(s[i]==s[j+1]) j++;
			nxt[0][i]=j;num[i]=num[j]+1;
		}
		for(int k=1;k<=19;k++)
			for(int i=1;i<=l;i++)
				nxt[k][i]=nxt[k-1][nxt[k-1][i]];
		for(int i=1;i<=l;i++)
		{
			int p=i;
			for(int k=19;k>=0;k--)
				if(nxt[k][p]>i/2) p=nxt[k][p];
			if(p!=i) ans=ans*((LL)num[nxt[0][p]]+1)%Mod;
			else ans=ans*(LL)num[i]%Mod;
		}
		printf("%lld\n",ans);
	}
	return 0;
}
```
