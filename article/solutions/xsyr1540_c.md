# C. 二〇四八

### 题意简述

你有一个区间，你每次可以选择两个相邻且相同的数$k$，将它们合并成一个$k+1$，或者你可以删去任意一个数。操作到区间只剩最后一个数为止

现在有多组询问，每次询问给定$l,r,k$，求$[l,r]$的子区间中，有多少个区间能通过上述操作使得最后剩下的数为$k$

序列长度、询问次数、值域范围均为2e5级别

### 题解

设$f_{k,i}$表示以$i$为左端点，合成$k$所需的最小右端点，为了方便后面的处理，我们让$f_{k,i}$的值在原意义的基础上加$1$

那么，对于一组询问$[l,r,k]$，它的答案就是$\sum\limits_{i=l}^r\max(r-f_{k,i}+2,0)$

转移时，考虑右边的第一个$k+1$以及右边的第二个$k$，不难得出转移方程：$f_{k+1,i}=\min(f_{k,f_{k,i}},next_{i,k+1}+1)$。其中$next_{i,k+1}$表示位置$i$右侧第一个$k+1$的位置

不妨考虑离线。将所有询问按$k$离线，再将原序列的所有位置按值离线出来。然后设当前正在考虑的值为$c$，用一棵线段树维护$f_{c,i=1\to n}$的区间最小值、区间最大值、区间和

那么对于一组询问$(l,r,k)$，答案的式子等价于：对于所有内部$f$值相等且这个$f$值不大于$(r+1)$的区间$[l_i,r_i]$，求：$(r+2)(r_i-l_i+1)-\sum\limits_{j=l_i}^{r_i}f_{k,j}$。于是我们可以使用势能线段树来实现。具体地，若区间内$f$值相等，则直接按上式计算，否则递归下去。若整个区间的最小值都大于$(r+1)$了，则返回

现在来考虑dp的更新。我们现在手上握着$f_{c,i=1\to n}$的线段树，我们要让它快速变成$f_{c+1,i=1\to n}$的线段树

先来考虑转移的第一个部分：$f_{k+1,i}=f_{k,f_{k,i}}$

从右往左遍历线段树，将内部$f$值相等的区间顺次存下来。然后对于一个$f$值相等的区间，在我们存下的所有区间中，找到这个区间的$f$指向的那个区间，将这个区间的$f$值更新为指向区间的$f$值。因为从右往左遍历时，$f$值是单调递减的，所以可以用一个单调左移的指针来快速查找$f$值指向的区间

接下来考虑第二部分，在第一部分的基础上，我们要求：$f_{k,i}=\min(f_{k,i},next_{i,k}+1)$

考虑一个值为$k$的位置$p$，它有可能是$[1,p-1]$中某些位置的$next$值。所以$[1,p-1]$区间中的$f$值都要对$p$取一个$\min$。区间取$\min$也可以使用势能线段树来实现

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
    template <class I>inline void print(I x)
    {
        if(!x) putc('0');
        while(x) qu[++qr]=x%10+'0',x/=10;
        while(qr) putc(qu[qr--]);
    }
}

using namespace IO;
typedef long long LL;
typedef pair<int,int> pii;
struct QRY{int l,r,id;};
const int N=200010;
LL mn[N<<2],mx[N<<2],sum[N<<2],tag[N<<2];
vector<int> a[N];
vector<QRY> qry[N];
int n,m,q,cnt,now;
LL ans[N],val[N];
pii b[N];

inline void modify(int o,int l,int r,LL x)
{
    tag[o]=mn[o]=mx[o]=x;
    sum[o]=x*(r-l+1);
}

inline void maintain(int o)
{
    mn[o]=min(mn[o<<1],mn[o<<1|1]);
    mx[o]=max(mx[o<<1],mx[o<<1|1]);
    sum[o]=sum[o<<1]+sum[o<<1|1];
}

inline void pushdown(int o,int l,int r)
{
    if(tag[o]==-1) return;
    int mid=(l+r)/2;
    modify(o<<1,l,mid,tag[o]);
    modify(o<<1|1,mid+1,r,tag[o]);
    tag[o]=-1;
}

void build(int o,int l,int r)
{
    tag[o]=-1;
    if(l==r){mx[o]=mn[o]=n+2;return;}
    int mid=(l+r)/2;
    build(o<<1,l,mid);
    build(o<<1|1,mid+1,r);
    maintain(o);
}

void update(int o,int l,int r,int nl,int nr,LL x)
{
    if(mx[o]<=x) return;
    if(l>=nl&&r<=nr&&mn[o]>=x)
    {
        modify(o,l,r,x);
        return;
    }
    int mid=(l+r)/2;pushdown(o,l,r);
    if(nl<=mid) update(o<<1,l,mid,nl,nr,x);
    if(nr>mid) update(o<<1|1,mid+1,r,nl,nr,x);
    maintain(o);
}

LL query(int o,int l,int r,int nl,int nr,LL x)
{
    if(l>=nl&&r<=nr)
    {
        if(mn[o]>x+1) return 0;
        if(mx[o]<=x+1) return (x+2)*(r-l+1)-sum[o];
    }
    pushdown(o,l,r);
    int mid=(l+r)/2;LL res=0;
    if(nl<=mid) res+=query(o<<1,l,mid,nl,nr,x);
    if(nr>mid) res+=query(o<<1|1,mid+1,r,nl,nr,x);
    return res;
}

void jump(int o,int l,int r)
{
    if(mn[o]==mx[o])
    {
        b[++cnt]=pii(l,r);
        val[cnt]=mn[o];
        while(mn[o]<b[now].first) now++;
        modify(o,l,r,val[now]);
        return;
    }
    pushdown(o,l,r);
    int mid=(l+r)/2;
    jump(o<<1|1,mid+1,r);
    jump(o<<1,l,mid);
    maintain(o);
}

int main()
{
    n=read();m=read();q=read();
    for(int i=1;i<=n;i++) a[read()].push_back(i);
    for(int i=1,k,l,r;i<=q;i++)
    {
        l=read();r=read();k=read();
        qry[k].push_back({l,r,i});
    }
    build(1,1,n+2);
    for(int i=1;i<=m;i++)
    {
        cnt=0;now=1;jump(1,1,n+2);
        for(int p : a[i]) update(1,1,n+2,1,p,p+1);
        for(QRY p : qry[i]) ans[p.id]=query(1,1,n+2,p.l,p.r,p.r);
    }
    for(int i=1;i<=q;i++)
        print(ans[i]),putc('\n');
    return flush(),0;
}
```

### 关于复杂度

复杂度是两个log的

吉老师说复杂度证了半个黑板……

那我们也没什么办法了，吉老师既然证了是对的，我们写就完事了

这里给一下第一部分更新的证明

假如我们将位置$i$的$f$指向的点看作它在树上的父亲。那么，对于一次更新，所有点的新父亲会变成它原来的祖父，于是这棵树层数减半，在log次操作后就没了