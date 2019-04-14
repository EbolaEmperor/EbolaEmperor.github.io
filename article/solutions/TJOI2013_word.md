[返回首页](https://EbolaEmperor.github.io)
# 【TJOI2013】单词 题解

[我是题目链接](https://www.lydsy.com/JudgeOnline/problem.php?id=3172)

### 题解

·某巨佬：“这题有什么可做的吗？”

·看来还是我太菜了，步入正题

·一个简单的想法就是，用给定的字符串造一台AC自动机，然后把全部字符串一个一个丢进去匹配，匹配过程中把沿途经过的结点sum+1，然后对所有经过的结点，都沿它的fail指针上跳，把经过的结点都sum+1，答案就是字符串词尾结点的sum值

·这样复杂度显然不对

·观察这个过程，沿fail指针上跳其实就是一个信息上传的过程，由于fail指针反转过来可以构成一棵树，因此我们可以考虑在树上解决这个问题

·发现构造fail指针时，fail树的子节点总是比父节点后处理，因此构造fail指针的那个队列，如果反过来，就是我们要的信息上传顺序

·一开始插入字符串时把Trie树上每个结点的sum++，然后按上述顺序去上传信息，那么每个字符串的答案就是它词尾结点的sum值

```cpp
#include<bits/stdc++.h>
using namespace std;

int ch[1000010][26],fail[1000010],sz=0;
int sum[1000010];
int q[1000010],head=0,tail=0;
char s[1000010];
int word[1000010];

void insert(char *s,int k)
{
	int p=0,len=strlen(s);
	for(int i=0;i<len;i++)
	{
		int j=s[i]-'a';
		if(!ch[p][j]) ch[p][j]=++sz;
		p=ch[p][j];
		sum[p]++;
	}
	word[k]=p;
}

void getfail()
{
	q[tail++]=0;
	while(head<tail)
	{
		int u=q[head++];
		for(int i=0;i<26;i++)
			if(ch[u][i])
			{
				int v=ch[u][i];
				if(u) fail[v]=ch[fail[u]][i];
				q[tail++]=v;
			}
			else ch[u][i]=ch[fail[u]][i];
	}
	for(int i=tail-1;i>=0;i--)
		sum[fail[q[i]]]+=sum[q[i]];
}

int main()
{
	int n;cin>>n;
	for(int i=1;i<=n;i++)
		scanf("%s",s),insert(s,i);
	getfail();
	for(int i=1;i<=n;i++)
		printf("%d\n",sum[word[i]]);
	return 0;
}
```