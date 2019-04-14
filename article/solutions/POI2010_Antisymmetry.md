[返回首页](https://EbolaEmperor.github.io)
# 【POI2010】Antisymmetry 题解

### 题解

·题中给的定义与回文有相似之处，可以考虑把它转化为回文的问题

·显然“对称串”一定是偶数长度，因为奇数长度的串中间位置一定不匹配

·可以推出，我们的“对称串”条件是，串中所有关于中点对称的位置上两数相加为1

·这就可以用manacher算法了，匹配条件改为s[i-p]+s[i+p]=1即可

·但这样的话我们的填充位置怎么办？

·有一个比较好的办法，就是把原串中的1改为2，然后填充1，匹配条件改为s[i+p]+s[i-p]=2

```cpp
#include<bits/stdc++.h>
using namespace std;

char s[500010];
int ss[1000010];
int p[1000010];

int manacher(int l)
{
	int len=0;
	ss[len++]=3;ss[len++]=1;
	for(int i=0;i<l;i++)
	{
		if(s[i]=='0') ss[len++]=0;
		else ss[len++]=2;
		ss[len++]=1;
	}
	ss[len]=3;
	int id=0,mx=0;
	for(int i=1;i<len;i+=2)
	{
		if(i<mx) p[i]=min(p[2*id-i],mx-i);
		else p[i]=1;
		while(ss[i-p[i]]+ss[i+p[i]]==2) p[i]++;
		if(i+p[i]>mx) mx=i+p[i],id=i;
	}
	return len;
}

int main()
{
	int n;
	long long ans=0;
	scanf("%d%s",&n,s);
	int len=manacher(n);
	for(int i=1;i<len;i+=2) ans+=p[i]/2;
	cout<<ans<<endl;
	return 0;
}
```