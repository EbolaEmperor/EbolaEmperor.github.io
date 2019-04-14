[返回首页](https://EbolaEmperor.github.io)
# 【CF835D】Palindromic Characteristics 题解

[我是题目链接](http://codeforces.com/problemset/problem/835/D)

### 题解

·这题的数据范围非常小，因此是可以字符串hash乱搞一波的，但我觉得这样的话，这题就出的一点意义都没有了，为了发挥这题的价值，我们把它当做|s|≤5000000的题目来做

·把它开到了这个范围，那肯定就是要线性的做法了，我们考虑使用回文自动机

·对于回文自动机上每个结点p，计算其出现的次数cnt[p]，以及长度小于len[p]/2的最长回文后缀g[p]

·可以在每次插入完之后对cnt[last]+1，全部插入完了再沿fail树上传，这样可以O(n)求出cnt数组

·设f[p]表示回文串p是几阶的，那么我们可以得到如下转移方程：

	f[p]=1  ,  len[p]/2≠len[g[p]]

	f[p]=f[g[p]]+1  ,  len[p]/2=len[g[p]]

·转移顺序应该按照点编号递增，因为p总是出现在g[p]后面

·那么我们枚举p，让ans[f[p]]+=cnt[p]，由于k阶回文串必定是1~k-1阶回文串，所以我们对ans数组求一遍后缀和就得到了最终答案

```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAXN=5000010;
char ss[MAXN];
int s[MAXN],n;
int ch[MAXN][26],sz;
int fail[MAXN],lst;
int len[MAXN],cnt[MAXN];
int g[MAXN],f[MAXN];
long long ans[MAXN];

int newnode(int l){len[++sz]=l;return sz;}

void init()
{
	sz=-1;
	n=lst=0;
	newnode(0);
	newnode(-1);
	fail[0]=1;
	s[0]=-1;
}

int find(int p)
{
	while(s[n]!=s[n-len[p]-1]) p=fail[p];
	return p;
}

void insert(int c)
{
	s[++n]=c;
	int cur=find(lst);
	if(!ch[cur][c])
	{
		int now=newnode(len[cur]+2);
		fail[now]=ch[find(fail[cur])][c];
		if(len[now]<=2)
			g[now]=fail[now];
		else
		{
			int p=g[cur];
			while(s[n]!=s[n-len[p]-1]||(len[p]+2)*2>len[now]) p=fail[p];
			g[now]=ch[p][c];
		}
		ch[cur][c]=now;
	}
	lst=ch[cur][c];
	cnt[lst]++;
}

int main()
{
	scanf("%s",ss);
	init();
	int l=strlen(ss);
	for(int i=0;i<l;i++)
		insert(ss[i]-'a');
	for(int i=sz;i>=2;i--)
		cnt[fail[i]]+=cnt[i];
	for(int i=2;i<=sz;i++)
	{
		if(len[i]/2==len[g[i]])
			f[i]=f[g[i]]+1;
		else f[i]=1;
		ans[f[i]]+=cnt[i];
	}
	for(int i=l;i>=1;i--)
		ans[i]+=ans[i+1];
	for(int i=1;i<=l;i++)
		printf("%lld\n",ans[i]);
	return 0;
}
```