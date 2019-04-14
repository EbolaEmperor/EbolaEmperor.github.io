# C. 求和

### 题意简述

给定一个长度为n(n≤1e6)的序列，求下标相差不超过k的两个数之和的最大值

需要在线支持单点修改操作(不超过1e6次)，每次修改后输出当前序列的答案

### 题解

用一棵线段树来维护每个区间取得最大值的位置（值相同则认为位置靠后的更大）

定义“位置p的答案”为钦定位置p必选时的答案。注意位置p的答案值只有当位置p是\[p-k,p+k\]中的最大值时才是上述定义，否则是0。再用一棵线段树来维护区间中所有位置的答案最大值。则总的答案就是这棵线段树根节点的值

现在考虑如何进行单点修改操作

在第一棵线段树上要做的事情是显然的。在第二棵线段树上，需要更新位置p的答案。此外，对于\[p-k,p-1\]中取得最大值的位置，也要更新答案，右边也是一样的

现在解释一下为什么p不是\[p-k,p+k\]中的最大值，位置p的答案要设为0。我们现在假设不这么做。令a\[p\]远大于其它值，显然\[p-k,p+k\]中任何位置q的答案都是a\[q\]+a\[p\]。此时把a\[p\]变成非常小的数，我们只修改了p位置，以及左边右边取得最大值的位置，其它位置q的答案仍然是a\[q\]+原a\[p\].因为一开始a\[p\]是远大于其它值的，所以此时a\[q\]+原a\[p\]必然是最大的，于是答案线段树的根节点就存了一个根本不存在的答案，就错了

```cpp
#include<bits/stdc++.h>
using namespace std;

namespace IO
{
    const int S=(1<<20)+5;
    char buf[S],*H,*T;
    inline char Get()
    {
        if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
        if(H==T) return -1;return *H++;
    }
    inline int read()
    {
        int x=0;char c=Get();
        while(!isdigit(c)) c=Get();
        while(isdigit(c)) x=x*10+c-'0',c=Get();
        return x;
    }
    char obuf[S],*oS=obuf,*oT=oS+S-1,c,qu[55];int qr;
    inline void flush(){fwrite(obuf,1,oS-obuf,stdout);oS=obuf;}
    inline void putc(char x){*oS++ =x;if(oS==oT) flush();}
    inline void print(int x)
    {
        if(!x) putc('0');
        while(x) qu[++qr]=x%10+'0',x/=10;
        while(qr) putc(qu[qr--]);
        putc('\n');
    }
}

using namespace IO;
const int N=1000010;
int tr1[N<<2],tr2[N<<2],leaf;
int n,k,q,op,a[N];

inline int cmp(int x,int y){return a[x]==a[y]?x<y:a[x]<a[y];}
inline int maxp(int x,int y){return cmp(x,y)?y:x;}

void build()
{
    for(leaf=1;leaf<=n+3;leaf<<=1);
    for(int i=1;i<=n;i++) tr1[i+leaf]=i;
    for(int i=leaf;i;i--) tr1[i]=maxp(tr1[i<<1],tr1[i<<1|1]);
}

void modify1(int p){for(p=(p+leaf)>>1;p;p>>=1) tr1[p]=maxp(tr1[p<<1],tr1[p<<1|1]);}

int query(int l,int r)
{
    int res=0;
    for(l=leaf+l-1,r=leaf+r+1;l^r^1;l>>=1,r>>=1)
    {
        if(~l&1) res=maxp(res,tr1[l^1]);
        if(r&1) res=maxp(res,tr1[r^1]);
    }
    return res;
}

void modify2(int p)
{
    int L=p>1?query(max(p-k,1),p-1):0;
    int R=p<n?query(p+1,min(p+k,n)):0;
    int ans=maxp(L,R);
    tr2[leaf+p]=cmp(ans,p)?a[p]+a[ans]:0;
    for(p=(p+leaf)>>1;p;p>>=1)
        tr2[p]=max(tr2[p<<1],tr2[p<<1|1]);
}

int main()
{
    n=read();k=read();q=read();op=read();
    for(int i=1;i<=n;i++) a[i]=read();
    build();for(int i=1;i<=n;i++) modify2(i);
    print(tr2[1]);
    while(q--)
    {
        int x=read(),y=read();
        if(op) x^=tr2[1],y^=tr2[1];
        a[x]=y;modify1(x);modify2(x);
        if(x>1) modify2(query(max(1,x-k),x-1));
        if(x<n) modify2(query(x+1,min(n,x+k)));
        print(tr2[1]);
    }
    flush();
    return 0;
}
```