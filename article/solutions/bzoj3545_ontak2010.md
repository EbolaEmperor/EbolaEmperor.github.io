[返回首页](https://EbolaEmperor.github.io)
# 【BZOJ3545】【ONTAK2010】Peaks 题解

### 闲谈

·这道题是BZOJ上的权限题，没办法发链接，反正我是在广州二中的OJ上做的

·然后广州二中的OJ特别恶心，卡常能力惊人，再加上yww大佬的疯狂hack，导致数据异常强大

·所以要用尽各种奇技淫巧进行鬼畜优化

·不过我都还没用读入优化，居然还是A了

### 题解

·还是权值线段树的合并

·这题比较好的是它不强制在线，因此我们可以考虑离线做法

·把边按边权从小到大排序，询问按x从小到大排序，这样我们就可以顺次加边

·每处理一个询问时，把边权比该组询问的x小的边给加上就行

·反正路径都是双向的，因此跟图论就一点关系都没有了，只要用并查集维护联通关系就好了

·初始时，我们对每个点建立一棵权值线段树，动态开点，否则MLE

·然后每次连边，我们先用并查集找到“祖父”，然后如果它们原来是联通的，就不管它，否则就将两颗权值线段树合并

·询问只要对相应的权值线段树进行kth操作就行了，注意题目要求第k大，而不是第k小

·这样会TLE最后一个点

·因此我们进行一步重要的优化——离散化

·把题目中的h数组离散化，这样权值线段树的规模就大大缩小了，数量级从10^9缩减到了10^5

·细节参考代码

```cpp
#include<bits/stdc++.h>
#define newnode(x) x=new Node,x->ls=x->rs=null
#define MAXN 100000
#define MAXM 500000
using namespace std;

struct Node
{
	Node *ls,*rs;
	int sum;
	Node(){ls=rs=NULL;sum=0;}
};
Node *root[MAXN+10],*null;
struct Edge{int from,to,capa;} t[MAXM+10];
struct Query{int u,x,k,id;} Q[MAXM+10];
int fa[MAXN+10];
int ans[MAXM+10];
int Hash[MAXN+10];
int val[MAXN+10];

int find(int x){return x==fa[x]?x:fa[x]=find(fa[x]);}

bool cmpE(const Edge &x,const Edge &y){return x.capa<y.capa;}
bool cmpQ(const Query &x,const Query &y){return x.x<y.x;}

void maintain(Node *o){o->sum=o->ls->sum+o->rs->sum;}

void insert(Node* &o,int l,int r,int x)
{
	newnode(o);
	o->sum=1;
	if(l==r) return;
	int mid=(l+r)/2;
	if(x<=mid) insert(o->ls,l,mid,x);
	else insert(o->rs,mid+1,r,x);
}

Node *merge(Node *p1,Node *p2)
{
	if(p1==null) return p2;
	if(p2==null) return p1;
	if(p1->ls==null&&p1->rs==null)
	{
		p1->sum=p1->sum+p2->sum;
		return p1;
	}
	p1->ls=merge(p1->ls,p2->ls);
	p1->rs=merge(p1->rs,p2->rs);
	maintain(p1);
	return p1;
}

int kth(Node *o,int l,int r,int x)
{
	if(l==r) return l;
	int size=o->ls->sum;
	int mid=(l+r)/2;
	if(x<=size) return kth(o->ls,l,mid,x);
	else return kth(o->rs,mid+1,r,x-size);
}

int main()
{
	null=new Node;
	null->ls=null->rs=null;
	int n,m,q,u,x,k;
	cin>>n>>m>>q;
	for(int i=1;i<=n;i++) scanf("%d",val+i),Hash[i]=val[i];
	for(int i=1;i<=m;i++) scanf("%d%d%d",&t[i].from,&t[i].to,&t[i].capa);
	for(int i=1;i<=q;i++) scanf("%d%d%d",&Q[i].u,&Q[i].x,&Q[i].k),Q[i].id=i;
	sort(Hash+1,Hash+1+n);
	int hs=unique(Hash+1,Hash+1+n)-(Hash+1);
	for(int i=1;i<=n;i++)
	{
		val[i]=lower_bound(Hash+1,Hash+1+hs,val[i])-Hash;
		insert(root[i],1,hs,val[i]);
	}
	sort(t+1,t+1+m,cmpE);
	sort(Q+1,Q+1+q,cmpQ);
	int pos=1;
	for(int i=1;i<=n;i++) fa[i]=i;
	for(int i=1;i<=q;i++)
	{
		u=Q[i].u;x=Q[i].x;k=Q[i].k;
		while(pos<=m&&t[pos].capa<=x)
		{
			int fr=t[pos].from,to=t[pos].to,cap=t[pos].capa;
			pos++;
			int ffr=find(fr),fto=find(to);
			if(ffr==fto) continue;
			fa[ffr]=fto;
			root[fto]=merge(root[ffr],root[fto]);
		}
		int fu=find(u);
		if(root[fu]->sum<k) ans[Q[i].id]=-1;
		else ans[Q[i].id]=Hash[kth(root[fu],1,hs,root[fu]->sum-k+1)];
	}
	for(int i=1;i<=q;i++) printf("%d\n",ans[i]);
	return 0;
}
```