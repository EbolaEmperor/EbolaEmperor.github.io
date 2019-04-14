[返回首页](https://EbolaEmperor.github.io)
# 【APIO2014】回文串 题解

### 题解

·只要会了回文自动机，这就是一道裸题

·在回文自动机每个结点上维护一个cnt数组，表示这个回文串出现了多少次

·每次加完字符，让cnt[last]++

·最后在fail树上把信息从底向上往父节点上传即可（不必建出fail树）

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int MAXN=300010;
char ss[MAXN];
int s[MAXN],n;
int ch[MAXN][26],sz;
int fail[MAXN],lst;
int len[MAXN],cnt[MAXN];

int newnode(int l){len[++sz]=l;return sz;}

void init()
{
	n=lst=0;
	sz=-1;
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

void insert(char c)
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

int main()
{
	scanf("%s",ss);
	init();
	int l=strlen(ss);
	for(int i=0;i<l;i++)
		insert(ss[i]-'a');
	for(int i=sz;i>=2;i--)
		cnt[fail[i]]+=cnt[i];
	LL ans=0;
	for(int i=2;i<=sz;i++)
		ans=max(ans,(LL)len[i]*cnt[i]);
	cout<<ans<<endl;
	return 0;
}
```