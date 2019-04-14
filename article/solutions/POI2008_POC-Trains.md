[返回首页](https://EbolaEmperor.github.io)
# 【POI2008】POC-Trains 题解

[我是题目链接](https://www.lydsy.com/JudgeOnline/problem.php?id=1125)

### 题解

·发现字符串都很短，因此哈希冲突的概率不大，我们维护这些串的哈希值

·那么我们对每个哈希值，要用一个数据结构来维护它，这个数据结构需要支持：插入、删除、下放标记（因为这个数据结构的大小，对里面所有元素的答案都可能有贡献）

·线段树（动态开点）、平衡树都是可以的，这里我选择了Treap

·开若干个Treap，与哈希值一一对应，其键值的含义是字符串的编号

·由于哈希值是非常大的，不可能将根数组的下标开到那么大，因此我们使用一个map，将哈希值散列到一个小数组中，这样根数组就不会超过n+m了。当然也可以直接将map作为根数组

·每当一个字符串的哈希值发生改变，我们将这个字符串从原来那个哈希值对应的Treap中删去，然后加入到新哈希值对应的Treap中，并更新根节点的lazy标记

·lazy表示的是自己所在的Treap中，曾经达到过的最大结点数，这就是题目中说的“在交换车厢的某个时刻，与其颜色完全相同的火车最多有多少”

·那么在插入、删除、旋转的过程中，都要先进行标记下放，并顺便更新答案。由于同时有标记下放与信息上传，因此一定要特别注意顺序。下放是访问一个结点之前进行的，上传是访问完一个结点进行的

·注意加上a=c的特判，否则会导致RE

```cpp
#include<bits/stdc++.h>
#define newnode(x) x=new Node,x->son[0]=x->son[1]=null
using namespace std;

int read()
{
	int x=0;char c=getchar();
	while(c<'0'||c>'9') c=getchar();
	while(c>='0'&&c<='9') x=x*10+c-'0',c=getchar();
	return x;
}

typedef unsigned long long ULL;
const int N=1010;
ULL f[N],p[N];
char s[N][110];
int tot=0;
int ans[N];

struct Node
{
	Node *son[2];
	int sz,v,tag,r;
	Node(){son[0]=son[1]=NULL;}
};
Node *null;
map<ULL,Node*> rt;

void down(Node* &o,int x)
{
	ans[o->v]=max(ans[o->v],x);
	o->tag=max(o->tag,x);
}

void pushdown(Node* &o)
{
	if(!o->tag) return;
	Node *lc=o->son[0];
	Node *rc=o->son[1];
	if(lc!=null) down(lc,o->tag);
	if(rc!=null) down(rc,o->tag);
	o->tag=0;
}

void maintain(Node* &o){o->sz=o->son[0]->sz+o->son[1]->sz+1;}

void rotate(Node* &o,int d)
{
	Node *k=o->son[d^1];
	pushdown(o);
	pushdown(k);
	o->son[d^1]=k->son[d];
	k->son[d]=o;
	maintain(o);
	maintain(k);
	o=k;
}

void insert(Node* &o,int x)
{
	if(o==null)
	{
		newnode(o);
		o->sz=1;
		o->v=x;
		o->tag=0;
		o->r=rand();
		return;
	}
	pushdown(o);
	if(x<o->v)
	{
		insert(o->son[0],x);
		if(o->son[0]->r>o->r) rotate(o,1);
		else maintain(o);
	}
	else
	{
		insert(o->son[1],x);
		if(o->son[1]->r>o->r) rotate(o,0);
		else maintain(o);
	}
}

void remove(Node* &o,int x)
{
	if(o==null) return;
	pushdown(o);
	if(o->v==x)
	{
		if(o->son[0]==null) o=o->son[1];
		else if(o->son[1]==null) o=o->son[0];
		else
		{
			if(o->son[0]->r>o->son[1]->r) rotate(o,1),remove(o->son[1],x);
			else rotate(o,0),remove(o->son[0],x);
		}
	}
	else if(x<o->v) remove(o->son[0],x);
	else remove(o->son[1],x);
	if(o!=null) maintain(o);
}

void insert(int x)
{
	if(!rt.count(f[x])) rt[f[x]]=null;
	insert(rt[f[x]],x);
	Node *o=rt[f[x]];
	down(o,o->sz);
}

int main()
{
	newnode(null);
	int n=read(),l=read(),m=read();
	p[0]=1;for(int i=1;i<=100;i++) p[i]=131*p[i-1];
	for(int i=1;i<=n;i++)
	{
		scanf("%s",s[i]+1);
		for(int j=1;j<=l;j++)	
			f[i]=f[i]*131+s[i][j];
		insert(i);
	}
	for(int i=1;i<=m;i++)
	{
		int a=read(),b=read(),c=read(),d=read();
		if(a==c)
		{
			remove(rt[f[a]],a);
			f[a]+=p[l-b]*(s[a][d]-s[a][b]);
			f[a]+=p[l-d]*(s[a][b]-s[a][d]);
			swap(s[a][b],s[a][d]);
			insert(a);
			continue;
		}
		remove(rt[f[a]],a);remove(rt[f[c]],c);
		f[a]+=p[l-b]*(s[c][d]-s[a][b]);
		f[c]+=p[l-d]*(s[a][b]-s[c][d]);
		swap(s[a][b],s[c][d]);
		insert(a);insert(c);
	}
	for(int i=1;i<=n;i++) remove(rt[f[i]],i);
	for(int i=1;i<=n;i++) printf("%d\n",ans[i]);
	return 0;
}
```