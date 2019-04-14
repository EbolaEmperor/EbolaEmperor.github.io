[返回首页](https://EbolaEmperor.github.io)
# 【HNOI2006】最短母串 题解

[我是题目链接](https://www.lydsy.com/JudgeOnline/problem.php?id=1195)

### 题解

·这题据说可以用状压DP，但思维难度有点大，我们考虑在AC自动机上状压

·将给定串构造出AC自动机，每个词尾结点记录一个state，对于第p个单词，其词尾结点的state=1<<(p-1)，当然如果有重复，我们就把state或到一起

·然后在AC自动机上进行BFS，因为我们总是优先走更小的字母，所以不用担心字典序的问题

·每走到一个结点，沿它的fail指针把沿途结点的state或上来，因为fail指针指向的一段字符总是被自己的前缀包含的

·BFS的过程中记录当前状态是由哪一步过来的，以便最后的答案输出

·当前状态=(1<<n)-1时得到答案，可以输出。因为我们是BFS，所以第一个得到的答案一定是最短的，字典序的问题上面已经说过了

```cpp
#include<bits/stdc++.h>
using namespace std;

struct AC
{
	int ch[26];
	int state;
	int fail;
	AC(){memset(ch,0,sizeof(ch));state=fail=0;}
} t[610];
int sz=0,n;
char s[60];
bool vis[610][4100];
short fromu[610][4100],fromc[610][4100];
int froms[610][4100];

void insert(char *s,int v)
{
	int p=0,len=strlen(s);
	for(int i=0;i<len;i++)
	{
		int j=s[i]-'A';
		if(!t[p].ch[j]) t[p].ch[j]=++sz;
		p=t[p].ch[j];
	}
	t[p].state|=1<<(v-1);
}

void getfail()
{
	queue<int> q;q.push(0);
	while(!q.empty())
	{
		int o=q.front();
		for(int i=0;i<26;i++)
			if(t[o].ch[i])
			{
				int v=t[o].ch[i];
				if(o) t[v].fail=t[t[o].fail].ch[i];
				q.push(v);
			}
			else t[o].ch[i]=t[t[o].fail].ch[i];
		q.pop();
	}
}

void print(int u,int s)
{
	if(!u) return;
	print(fromu[u][s],froms[u][s]);
	printf("%c",fromc[u][s]+'A');
}

void bfs()
{
	queue<int> q1,q2;
	q1.push(0);q2.push(0);
	vis[0][0]=1;
	while(!q1.empty())
	{
		int u=q1.front(),s=q2.front();
		if(s==(1<<n)-1) print(u,s),exit(0);
		for(int i=0;i<26;i++)
		{
			int v=t[u].ch[i],ss=s;
			for(int j=v;j;j=t[j].fail) ss|=t[j].state;
			if(!vis[v][ss])
			{
				vis[v][ss]=1;
				fromu[v][ss]=u;
				froms[v][ss]=s;
				fromc[v][ss]=i;
				q1.push(v);
				q2.push(ss);
			}
		}
		q1.pop();q2.pop();
	}
}

int main()
{
	cin>>n;
	for(int i=1;i<=n;i++) scanf("%s",s),insert(s,i);
	getfail();
	bfs();
	return 0;
}
```