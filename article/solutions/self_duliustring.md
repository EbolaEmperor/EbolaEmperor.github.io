[返回首页](https://EbolaEmperor.github.io)
# 【原创题】毒瘤串 题解

### 题目描述

一串数字被定义为“毒瘤串”，需要满足：

1. 顺着读和反着读一样（即回文）

2. 串的左半部分单调不减（若串长为奇数，中间位置也要算入“左半部分”）

比如“1 2 2 3 2 2 1”就是一个“毒瘤串”

再比如“1 3 2 2 3 1”虽然满足条件1，但不满足条件2，所以不是“毒瘤串”

现在给定一个串S，请你求出串S的子串中所有的“毒瘤串”，并将它们按长度升序排序。

你需要回答Q组询问，询问格式如下：

1926 x ，你需要回答排序后排名为x的“毒瘤串”长度为多少

817 x ，你需要回答排序后长度为x的“毒瘤串”排名是多少（如果有多个排名，请输出最小的）

### 数据范围

对于30%的数据，满足 Q≤1000

对于40%的数据，满足 n≤1000

对于另10%的数据，只有第一类询问

对于100%的数据，满足 n≤3\*10^6 , Q≤10^5 ，保证询问合法

提交地址：[毒瘤串](https://www.luogu.org/problemnew/show/U26892)

### 题解

### 40分做法

·这题给的暴力分比较多，40分应该是比较容易的

·我们可以暴力处理出“毒瘤串”，即从每个位置出发，向两端扩展搜索，处理时间复杂度O(n^2)

·对于询问，只要设一个数组f\[x\]，表示长度为x的“毒瘤串”有多少个就可以了。

·对于第一类操作，遍历f数组，如果f\[i\]>=x，则说明找到了解，输出i即可。若f\[i\]<x，说明解还在后面，此时让x减去f\[i\]，继续遍历下一个i

·对于第二类操作，直接将f\[i\](i<x)累加，输出累加结果+1即可

·理论询问时间复杂度O(Q\*n)，由于40%的数据不强，是可以过掉的

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=3000010;
int s[2*N],ss[2*N];
long long f[N];
int main()
{
	int n,len=0;
	scanf("%d",&n);
	for(int i=1;i<=n;i++) scanf("%d",ss+i);
	s[len]=5;s[++len]=-1;
	for(int i=1;i<=n;i++)
		s[++len]=ss[i],s[++len]=-1;
	s[len+1]=6;
	for(int i=1;i<=len;i++)
	{
		int p=1;
		while(s[i+p]==s[i-p]&&(p<2||s[i-p]<=s[i-p+2])) p++;
		if(s[i]==-1)
			for(int j=1;j<p;j+=2) f[j+1]++;
		else
			for(int j=0;j<p;j+=2) f[j+1]++;
	}
	int Q,opt,x;
	cin>>Q;
	for(int i=1;i<=Q;i++)
	{
		scanf("%d%d",&opt,&x);
		if(opt==1926)
		{
			for(int i=1;i<=n;i++)
				if(f[i]>=x){printf("%d\n",i);break;}
				else x-=f[i];
		}
		else
		{
			long long k=0;
			for(int i=1;i<x;i++) k+=f[i];
			printf("%lld\n",k+1);
		}
	}
	return 0;
}
```

### 100分做法

·n的这个范围，其实暗示了正解的处理要达到O(n)，使用manacher算法

·我们先弄一个f数组，含义和40分做法是一样的，其维护方式是每求出一个长度为p的回文子串，就让f[p]+1 ，跑完需要处理一下，因为manacher跑出来的长度为x的回文，必定包含了一个长度为x-2的回文，所以要对f分奇偶做一遍后缀和，具体可以看代码

·为了达到查询的目的，我们再对f数组求前缀和，对于第二类询问，输出f\[x-1\]+1即可；对于第一类询问，二分查找一个k，满足f\[k-1\]<x≤f\[k\]，输出k即为答案。时间复杂度O(Q log n)

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=3000010;
int s[2*N],ss[2*N];
int p[2*N];
long long f[N];

int main()
{
	int n,len=0;
	scanf("%d",&n);
	for(int i=1;i<=n;i++) scanf("%d",ss+i);
	s[len]=5;s[++len]=-1;
	for(int i=1;i<=n;i++)
		s[++len]=ss[i],s[++len]=-1;
	s[len+1]=6;
	int mx=0,id=0,mxa=0;
	for(int i=1;i<=len;i++)
	{
		if(i<mx) p[i]=min(p[2*id-i],mx-i);
		else p[i]=1;
		while(s[i+p[i]]==s[i-p[i]]&&(p[i]<2||s[i-p[i]]<=s[i-p[i]+2])) p[i]++;
		f[p[i]-1]++;
		mxa=max(mxa,p[i]-1);
		if(i+p[i]>mx) mx=i+p[i],id=i;
	}
	for(int i=mxa;i>=2;i--) f[i-2]+=f[i];
	for(int i=1;i<mxa;i++) f[i+1]+=f[i];
	f[0]=0;
	int Q,opt,x;
	for(cin>>Q;Q;Q--)
	{
		scanf("%d%d",&opt,&x);
		if(opt==1926) printf("%d\n",lower_bound(f+1,f+1+mxa,x)-f);
		else printf("%lld\n",f[x-1]+1);
	}
	return 0;
}
```

当然，使用回文自动机可以大大降低思维难度，这点在专题的“知识梳理里面是有提到的”

```cpp
#include<bits/stdc++.h>
#define New(x) x=new Node,x->lc=x->rc=null
using namespace std;

typedef long long LL;
const int MAXN=3000010;
int ch[MAXN][4],sz;
int fail[MAXN],lst;
int len[MAXN],cnt[MAXN];
int s[MAXN],n;
LL f[MAXN];

int newnode(int l){len[++sz]=l;return sz;}

void init()
{
	lst=n=0;
	sz=-1;
	newnode(0);
	newnode(-1);
	fail[0]=1;
	s[0]=5;
}

int find(int p)
{
	while(s[n]!=s[n-len[p]-1]) p=fail[p];
	return p;
}

void insert(int c)
{
	s[++n]=c;
	int cur=s[n]>s[n-1]?1:find(lst);
	if(!ch[cur][c])
	{
		int now=newnode(len[cur]+2);
		fail[now]=ch[find(fail[cur])][c];
		ch[cur][c]=now;
	}
	lst=ch[cur][c];
	cnt[lst]++;
}

int read()
{
	int x=0;
	char ch=getchar();
	while(ch<'0'||ch>'9') ch=getchar();
	while(ch>='0'&&ch<='9') x=x*10+ch-'0',ch=getchar();
	return x;
}

struct Node
{
	Node *lc,*rc;
	LL v,cnt,size;
	Node(){lc=rc=NULL;v=cnt=size=0;}
};
Node *rt,*null;

void build(Node* &o,int l,int r)
{
	New(o);
	LL mid=(l+r)/2;
	o->v=mid;
	o->cnt=f[mid];
	o->size=f[mid];
	if(l==r) return;
	if(l<mid) build(o->lc,l,mid-1);
	if(r>mid) build(o->rc,mid+1,r);
	o->size=o->lc->size+o->rc->size+o->cnt;
}

LL Kth(Node *o,LL k)
{
	LL sz=o->lc->size;
	if(k>sz&&k<=sz+o->cnt) return o->v;
	else if(k<=sz) return Kth(o->lc,k);
	else return Kth(o->rc,k-sz-o->cnt);
}

LL Rank(Node *o,LL x)
{
	LL sz=o->lc->size;
	if(o->v==x) return sz+1;
	else if(o->v>x) return Rank(o->lc,x);
	else return Rank(o->rc,x)+sz+o->cnt;
}

int main()
{
	New(null);
	int l=read();
	init();
	for(int i=0;i<l;i++)
		insert(read());
	int mx=0;
	for(int i=sz;i>=2;i--)
	{
		cnt[fail[i]]+=cnt[i];
		f[len[i]]+=cnt[i];
		mx=max(mx,len[i]);
	}
	build(rt,1,mx);
	int Q=read(),opt,x;
	for(int i=1;i<=Q;i++)
	{
		opt=read();x=read();
		if(opt==1926) printf("%lld\n",Kth(rt,x));
		else printf("%lld\n",Rank(rt,x));
	}
	return 0;
}
```
