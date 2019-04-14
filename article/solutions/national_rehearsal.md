[返回首页](https://EbolaEmperor.github.io)
# 【国家集训队】啦啦队排练 题解

[我是题目链接](https://www.lydsy.com/JudgeOnline/problem.php?id=2160)

### 题解

·这题还是比较简单的

·造一台回文自动机，然后O(n)求出每个回文串出现的次数，求法在之前的文章里面已经提过无数遍了

·设数组f[l]，表示长度为l的回文串出现次数之和，利用cnt数组和len数组求出f，然后就可以O(n)求解了

·注意题目只要奇数长度的回文串

·还有，k很大，求乘积时记得快速幂优化

```cpp
#include<bits/stdc++.h>
#define Mod 19930726
using namespace std;

typedef long long LL;
const int MAXN=1000010;
int s[MAXN],n;
int ch[MAXN][26],sz;
int fail[MAXN],lst;
int len[MAXN];
char ss[MAXN];
LL f[MAXN],cnt[MAXN];

int newnode(int l){len[++sz]=l;return sz;}

void init()
{
	n=lst=0;
	s[0]=-1;
	sz=-1;
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
		ch[cur][c]=now;
	}
	lst=ch[cur][c];
	cnt[lst]++;
}

LL QuickPow(LL a,LL b)
{
	LL ans=1;
	while(b)
	{
		if(b&1) ans=ans*a%Mod;
		a=a*a%Mod;
		b>>=1;
	}
	return ans;
}

int main()
{
	int l;LL k;
	scanf("%d%lld%s",&l,&k,ss);
	init();
	for(int i=0;i<l;i++)
		insert(ss[i]-'a');
	LL sum=0;
	for(int i=sz;i>=2;i--)
	{
		cnt[fail[i]]+=cnt[i];
		f[len[i]]+=cnt[i];
		sum+=cnt[i];
	}
	if(sum<k){puts("-1");return 0;}
	LL ans=1;
	for(LL i=l;i>=1;i--)
	{
		if(i%2==0) continue;
		if(f[i]>=k){ans=ans*QuickPow(i,(LL)k)%Mod;break;}
		else ans=ans*QuickPow(i,f[i])%Mod,k-=f[i];
	}
	cout<<ans<<endl;
	return 0;
}
```