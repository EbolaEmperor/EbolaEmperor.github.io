[返回首页](https://EbolaEmperor.github.io)
# 【NOI2011】阿狸的打字机 题解

### 题解

·这题有点毁天灭地啊（对于初学者而言）

·先说说40分做法

·造一台AC自动机，注意保存父指针，因为有“B”这个鬼畜操作

·然后每次询问把y走一遍，沿途经过的所有结点sum+1，对于所有结点，沿fail指针上跳，经过的结点sum+1，答案就是x的词尾结点的sum值

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<queue>
using namespace std;
int ch[100010][30];
int father[100010];
int val[100010];
int p[100010];
int f[100010];
char s[100010];
int sum=0,sz=0;
void insert()
{
	scanf(“%s”,s);
	int n=strlen(s),u=0;
	for(int i=0;i<n;i++)
	{
		if(s[i]>=’a’&&s[i]<=’z’)
		{
			int idx=s[i]-‘a’;
			if(ch[u][idx]==0)
			{
				sz++;
				memset(ch[sz],0,sizeof(ch[sz]));
				val[sz]=0;
				father[sz]=u;
				ch[u][idx]=sz;
			}
			u=ch[u][idx];
		}
		if(s[i]==’B’) u=father[u];
		if(s[i]==’P’) val[u]=1,p[++sum]=u;
	}
}
void GetFail()
{
	queue<int> q;
	f[0]=0;
	for(int i=0;i<26;i++)
		if(ch[0][i]!=0) q.push(ch[0][i]),f[ch[0][i]]=0;
	while(!q.empty())
	{
		int o=q.front();
		for(int i=0;i<26;i++)
		{
			int u=ch[o][i];
			if(u==0) continue;
			q.push(u);
			int v=f[o];
			while(v!=0&&ch[v][i]==0) v=f[v];
			f[u]=ch[v][i];
		}
		q.pop();
	}
}
int search(int x,int y)
{
	int u=p[y],cnt=0;
	while(u!=0)
	{
		for(int i=u;i!=0;i=f[i])
			if(i==p[x]) {cnt++;break;}
		u=father[u];
	}
	return cnt;
}
int main()
{
	insert();
	GetFail();
	int T,a,b;
	cin>>T;
	for(int tt=1;tt<=T;tt++)
	{
		scanf(“%d%d”,&a,&b);
		printf(“%d\n”,search(a,b));
	}
	return 0;
}
```

·接下来我们考虑怎么用一些奇技淫巧把分卡高一点

·注意到询问的y是有重复的，这样我们相同的y就会反复匹配很多次，这显然不是我们希望的

·所以把询问离线，把y相同的询问一并处理掉，可以卡到70分

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<queue>
#include<algorithm>
using namespace std;
struct Type_XY{int x,y,id,ans;} xy[100010];
int ch[100010][30];
int father[100010],val[100010],f[100010];
int p[100010],cnt[100010],ans[100010];
char s[100010];
int sum=0,sz=0;
bool operator < (Type_XY t1,Type_XY t2) {return t1.y<t2.y;}
void insert()
{
	scanf(“%s”,s);
	int n=strlen(s),u=0;
	for(int i=0;i<n;i++)
	{
		if(s[i]>=’a’&&s[i]<=’z’)
		{
			int idx=s[i]-‘a’;
			if(ch[u][idx]==0)
			{
				sz++;
				memset(ch[sz],0,sizeof(ch[sz]));
				val[sz]=0;
				father[sz]=u;
				ch[u][idx]=sz;
			}
			u=ch[u][idx];
		}
		if(s[i]==’B’) u=father[u];
		if(s[i]==’P’) p[++sum]=u,val[u]=sum;
	}
}
void GetFail()
{
	queue<int> q;
	f[0]=0;
	for(int i=0;i<26;i++)
		if(ch[0][i]!=0) q.push(ch[0][i]),f[ch[0][i]]=0;
	while(!q.empty())
	{
		int o=q.front();
		for(int i=0;i<26;i++)
		{
			int u=ch[o][i];
			if(u==0) continue;
			q.push(u);
			int v=f[o];
			while(v!=0&&ch[v][i]==0) v=f[v];
			f[u]=ch[v][i];
		}
		q.pop();
	}
}
void search(int y)
{
	memset(cnt,0,sizeof(cnt));
	int u=p[y];
	while(u!=0)
	{
		for(int i=u;i!=0;i=f[i])
		{
			if(val[i]!=0) cnt[val[i]]++;
		}
		u=father[u];
	}
}
int main()
{
	insert();
	GetFail();
	int T;
	cin>>T;
	for(int tt=1;tt<=T;tt++)
	{
		scanf(“%d%d”,&xy[tt].x,&xy[tt].y);
		xy[tt].id=tt;
	}
	sort(xy+1,xy+1+T);
	int pos=1;
	for(int i=1;i<=T;i=pos)
	{
		search(xy[i].y);
		while(xy[pos].y==xy[i].y)
		{
			xy[pos].ans=cnt[xy[pos].x];
			pos++;
		}
	}
	for(int i=1;i<=T;i++) ans[xy[i].id]=xy[i].ans;
	for(int i=1;i<=T;i++) printf(“%d\n”,ans[i]);
	return 0;
}
```

·我们看我们时间复杂度的瓶颈在哪里

·发现我们沿fail指针一步一步跳极其傻逼，而且每次重走y也是非常傻逼

·由于fail可以构成一棵树，我们可以搞出dfs序来，然后用树状数组在dfs序上维护信息

·计算dfs序时存储in和out，使得dfs序在in[p]和out[p]之间的都是p的子节点，这样我们求子树和就不用傻逼一样一步一步从子节点上传了，就可以在树状数组上用1个log的时间求出sum(out[p])-sum(in[p]-1)

·其实那一串字符就是一个给定好了的过程，读一个小写字母就在树状数组里加上信息，读到一个“B”就抹去信息，读到一个“P”就把y等于当前序号的询问处理一下

```cpp
#include<bits/stdc++.h>
#define lowbit(x) (x&-x)
using namespace std;

struct Type_XY{int x,y,id,ans;} qry[100010];
int ch[100010][30],sz=0;
int father[100010],val[100010],fail[100010];
int word[100010],cnt[100010],ans[100010];
char s[100010];
int bit[100010];
struct Edge{int to,next;} e[200010];
int h[100010],sum=0;
int in[100010],out[100010],dfn=0;

void add_edge(int u,int v)
{
	sum++;
	e[sum].to=v;
	e[sum].next=h[u];
	h[u]=sum;
}

void add(int p,int k)
{
	for(int i=p;i<=dfn;i+=lowbit(i))
		bit[i]+=k;
}

int query(int p)
{
	int res=0;
	for(int i=p;i>0;i-=lowbit(i))
		res+=bit[i];
	return res;
}

bool operator < (const Type_XY &t1,const Type_XY &t2) {return t1.y<t2.y;}

void insert()
{
	scanf("%s",s);
	int n=strlen(s),u=0,cnt=0;
	for(int i=0;i<n;i++)
	{
		if(s[i]>='a'&&s[i]<='z')
		{
			int idx=s[i]-'a';
			if(ch[u][idx]==0)
			{
				ch[u][idx]=++sz;
				father[sz]=u;
			}
			u=ch[u][idx];
		}
		if(s[i]=='B') u=father[u];
		if(s[i]=='P') word[++cnt]=u,val[u]=cnt;
	}
}

void GetFail()
{
	queue<int> q;
	fail[0]=0;
	for(int i=0;i<26;i++)
		if(ch[0][i]!=0) q.push(ch[0][i]),fail[ch[0][i]]=0;
	while(!q.empty())
	{
		int o=q.front();
		for(int i=0;i<26;i++)
			if(ch[o][i])
			{
				int v=ch[o][i];
				if(o) fail[v]=ch[fail[o]][i];
				q.push(v);
			}
			else ch[o][i]=ch[fail[o]][i];
		q.pop();
	}
}

void dfs(int u,int la)
{
	in[u]=++dfn;
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
	{
		if(e[tmp].to==la) continue;
		dfs(e[tmp].to,u);
	}
	out[u]=dfn;
}

void solve()
{
	int p=0,len=strlen(s),k=1,cnt=0;
	for(int i=0;i<len;i++)
	{
		if(s[i]>='a'&&s[i]<='z')
		{
			p=ch[p][s[i]-'a'];
			add(in[p],1);
		}
		if(s[i]=='B')
		{
			add(in[p],-1);
			p=father[p];
		}
		if(s[i]=='P')
		{
			cnt++;
			for(int j=k;qry[j].y==cnt;j++,k++)
			{
				int ans1=query(out[word[qry[j].x]]);
				int ans2=query(in[word[qry[j].x]]-1);
				qry[j].ans=ans1-ans2;
			}
		}
	}
}

int main()
{
	insert();
	GetFail();
	for(int i=1;i<=sz;i++)
		add_edge(i,fail[i]),add_edge(fail[i],i);
	dfs(0,0);
	int T;
	cin>>T;
	for(int tt=1;tt<=T;tt++)
	{
		scanf("%d%d",&qry[tt].x,&qry[tt].y);
		qry[tt].id=tt;
	}
	sort(qry+1,qry+1+T);
	solve();
	for(int i=1;i<=T;i++) ans[qry[i].id]=qry[i].ans;
	for(int i=1;i<=T;i++) printf("%d\n",ans[i]);
	return 0;
}
```