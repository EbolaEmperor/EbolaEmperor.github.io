[返回首页](https://EbolaEmperor.github.io)
# 【SCOI2012】喵星球上的点名 题解

[我是题目链接](https://www.lydsy.com/JudgeOnline/problem.php?id=2754)

### 题解

·我的代码在广州二中直接排到了rank 1，真是爽

·其实就是一个非常暴力的AC自动机，只不过儿子指针非常多，这个我们可以用散列表map来解决

·用点名串造一台AC自动机，然后用名字在AC自动机上匹配一遍，对所有途经的结点打上访问标记，同时每到一个点就沿它的fail指针往上走，走过的结点也打上访问标记。打标记的同时把结点的词尾信息加上，就是喵星人的点名次数。词尾结点的标记数量，就是点到喵星人的数量。

·注意喵星人的姓和名是分离的，因此我们在读取他们的名字的时候，可以在姓和名中间插一个无关紧要的数字，比如-1

·还需要注意老师的点名串可能有重复

```cpp
#include<bits/stdc++.h>
using namespace std;

map<int,int> ch[100010];
int fail[100010],sz=0;
int word[100010];
int tag[100010];
vector<int> ans[100010];
vector<int> miao[50010];
int s[100010];
int sum[100010];

void insert(int *s,int len,int k)
{
	int p=0;
	for(int i=0;i<len;i++)
	{
		int j=s[i];
		if(!ch[p].count(j)) ch[p][j]=++sz;
		p=ch[p][j];
	}
	tag[p]++;word[k]=p;
}

void getfail()
{
	queue<int> q;
	q.push(0);
	while(!q.empty())
	{
		int u=q.front();
		map<int,int>::iterator it;
		for(it=ch[u].begin();it!=ch[u].end();it++)
		{
			int i=it->first,v=it->second,k=fail[u];
			while(ch[k].count(i)==0&&k) k=fail[k];
			if(u) fail[v]=ch[k].count(i)==0?0:ch[k][i];
			q.push(v);
		}
		q.pop();
	}
}

int serch(vector<int> s,int k)
{
	int p=0,len=s.size(),res=0;
	for(int i=0;i<len;i++)
	{
		int j=s[i];
		while(p&&ch[p].count(j)==0) p=fail[p];
		if(ch[p].count(j)) p=ch[p][j];
		for(int f=p;f;f=fail[f])
			if(tag[f]&&(ans[f].size()==0||ans[f][ans[f].size()-1]!=k))
				ans[f].push_back(k),res+=tag[f];
	}
	return res;
}

int read()
{
	int x=0;
	char ch=getchar();
	while(ch<'0'||ch>'9') ch=getchar();
	while(ch>='0'&&ch<='9') x=x*10+ch-'0',ch=getchar();
	return x;
}

int main()
{
	int n,m,l;
	n=read();m=read();
	for(int i=1;i<=n;i++)
	{
		l=read();
		for(int j=1;j<=l;j++) miao[i].push_back(read());
		miao[i].push_back(-1);
		l=read();
		for(int j=1;j<=l;j++) miao[i].push_back(read());
	}
	for(int i=1;i<=m;i++)
	{
		l=read();
		for(int j=0;j<l;j++) s[j]=read();
		insert(s,l,i);
	}
	getfail();
	for(int i=1;i<=n;i++) sum[i]=serch(miao[i],i);
	for(int i=1;i<=m;i++) printf("%d\n",ans[word[i]].size());
	for(int i=1;i<=n;i++) printf("%d ",sum[i]);
	return 0;
}
```