[返回首页](https://EbolaEmperor.github.io)
# 【HDU5785】Interesting 题解

[我是题目链接](http://acm.hdu.edu.cn/showproblem.php?pid=5785)

### 题解

·官方题解给了manacher，然后各种毒瘤区间处理，非常烦人，根本不想写

·于是我决定使用回文自动机

·对于回文自动机上的每个结点p，维护两个信息，num[p]表示以p结尾的回文子串数量，sum[p]表示以p结尾的回文子串总长度，它们可以从fail[p]转移过来：

	num[p]=num[fail[p]]+1

	sum[p]=sum[fail[p]]+len[p]

·那么，我们正着搞一遍回文自动机，对于每个位置i，就可以求出以i结尾的回文子串起始坐标之和：

	f[i]=(i+2)*num[last]-sum[last]

·然后再倒着搞一遍回文自动机，那么对于每个位置i，我们有：

	ans+=f[i-1]*(i*num[last]+sum[last])

·就这样求出来答案，根本不用写什么繁琐的区间处理

·不要开long long，因为回文自动机承受不住这个空间，可以中间过程用long long计算，取模后赋值回int里面去

```cpp
#include<bits/stdc++.h>
#define Mod 1000000007
using namespace std;

typedef long long LL;
const int MAXN=1000010;
int ch[MAXN][26],fail[MAXN];
int len[MAXN],num[MAXN],sum[MAXN];
int lst,sz,n;
int s[MAXN];
char ss[MAXN];
int f[MAXN];

int newnode(int l)
{
	len[++sz]=l;
	memset(ch[sz],0,sizeof(ch[sz]));
	num[sz]=sum[sz]=0;
	return sz;
}

void init()
{
	sz=-1;
	lst=n=0;
	s[0]=-1;
	newnode(0);
	newnode(-1);
	fail[0]=1;
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
		num[now]=num[fail[now]]+1;
		sum[now]=(sum[fail[now]]+len[now])%Mod;
		ch[cur][c]=now;
	}
	lst=ch[cur][c];
}

int main()
{
	while(~scanf("%s",ss))
	{
		init();
		int l=strlen(ss);
		for(LL i=0;i<l;i++)
		{
			insert(ss[i]-'a');
			f[i+1]=((i+2)*num[lst]%Mod-sum[lst]+Mod)%Mod;
		}
		init();
		LL ans=0;
		for(LL i=l-1;i>=0;i--)
		{
			insert(ss[i]-'a');
			ans=(ans+f[i]*(i*num[lst]%Mod+sum[lst])%Mod)%Mod;
		}
		printf("%lld\n",ans);
	}
	return 0;
}
```