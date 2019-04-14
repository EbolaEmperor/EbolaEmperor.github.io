[返回首页](https://EbolaEmperor.github.io)
# 【URAL2040】Palindromes and Super Abilities 2 题解

### 题解

·题目问你每加入一个新字符，字符串中新添加的本质不同的回文串的个数

·可以发现每次的答案只能是0或1，证明略

·回文自动机上每个结点都是一段回文子串，它结点的总数就是本质不同的回文串的个数

·所以只要会回文自动机，这题就非常简单了

·每次加入新字符判断回文自动机的结点数有没有增加，若增加了，本次答案即为1，反之为0

```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAXN=5000010;
char ss[MAXN];
int s[MAXN],n;
int ch[MAXN][2],sz;
int fail[MAXN],lst;
int len[MAXN];

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

bool insert(int c)
{
	bool flag=0;
	s[++n]=c;
	int cur=find(lst);
	if(!ch[cur][c])
	{
		flag=1;
		int now=newnode(len[cur]+2);
		fail[now]=ch[find(fail[cur])][c];
		ch[cur][c]=now;
	}
	lst=ch[cur][c];
	return flag;
}

int main()
{
	scanf("%s",ss);
	init();
	int l=strlen(ss);
	for(int i=0;i<l;i++)
		printf("%d",insert(ss[i]-'a'));
	return 0;
}
```