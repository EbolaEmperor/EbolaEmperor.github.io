考虑对每一个二进制位分别计算贡献。设$cnt_k$表示异或和的$2^k$位为$1$的子集数量，那么令$n$表示全集大小，$m_k$表示$2^k$位为$1$的数有几个。显然符合条件的子集应恰好包含奇数个$2^k$位为$1$的数，其它随意。于是有：

$$cnt_k=2^{n-m_k}\times\sum_{i=1}^{\left\lfloor\frac{m_k}{2}\right\rfloor}\binom{m_k}{2i-1}=2^{n-m_k}\times2^{m_k-1}=2^{n-1}$$

特别地，当$m_k=0$时，$cnt_k=0$

感觉似乎简洁的有点出乎意料？一个二进制位是否有贡献，只和$m$是否为$0$有关！

于是，一个集合的子异和就是：$2^{n-1}\times\sum\limits_{i=0}^k[m_i>0]2^i$。不难看出这个求和运算实际上是在计算所有数的二进制或，所以该式可写作：$2^{n-1}\times orsum$

好了，现在询问已经解决了，只要求出路径上点的数量以及点权或就行。现在考虑修改

如果我们知道了一个集合的与、或，现在这个集合所有数要异或$c$。那么，令$\cup_k$、$\cap_k$、$c_k$分别表示或的$2^k$位、与的$2^k$位、$c$的$2^k$位，则不难得出：

$$\cup_{k[c_k==1]}'=\begin{cases}1&\cap_k==0\\0&\cap_k==1\end{cases},\qquad\cap_{k[c_k==1]}'=\begin{cases}1&\cup_k==0\\0&\cup_k==1\end{cases}$$

进而得到：

$$\cup'=(\cup\;\&\;\sim c)\;|\;(c\;\&\;\sim\cap)$$

$$\cap'=(\cap\;\&\;\sim c)\;|\;(c\;\&\;\sim\cup)$$

所以说，我们需要维护路径与、路径或。树剖套线段树即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=1e9+7;
const int N=200010;
unsigned sand[N<<2],sor[N<<2],lazy[N<<2];
int dfn[N],idx[N],dfc=0;
int fa[N],hson[N],top[N];
int dep[N],siz[N];
vector<int> g[N];
int n,m,val[N],pw2[N];

void update(int o,unsigned c)
{
    unsigned a=sor[o],b=sand[o];
    sor[o]=(a&~c)|(c&~b);
    sand[o]=(b&~c)|(c&~a);
    lazy[o]^=c;
}

void pushdown(int o)
{
    if(!lazy[o]) return;
    update(o<<1,lazy[o]);
    update(o<<1|1,lazy[o]);
    lazy[o]=0;
}

void build(int o,int l,int r)
{
    if(l==r){sand[o]=sor[o]=val[idx[l]];return;}
    int mid=(l+r)/2;
    build(o<<1,l,mid);
    build(o<<1|1,mid+1,r);
    sand[o]=sand[o<<1]&sand[o<<1|1];
    sor[o]=sor[o<<1]|sor[o<<1|1];
}

void cxor(int o,int l,int r,int nl,int nr,unsigned c)
{
    if(l>=nl&&r<=nr){update(o,c);return;}
    int mid=(l+r)/2;pushdown(o);
    if(nl<=mid) cxor(o<<1,l,mid,nl,nr,c);
    if(nr>mid) cxor(o<<1|1,mid+1,r,nl,nr,c);
    sand[o]=sand[o<<1]&sand[o<<1|1];
    sor[o]=sor[o<<1]|sor[o<<1|1];
}

unsigned qor(int o,int l,int r,int nl,int nr)
{
    if(l>=nl&&r<=nr) return sor[o];
    int mid=(l+r)/2;unsigned res=0;pushdown(o);
    if(nl<=mid) res|=qor(o<<1,l,mid,nl,nr);
    if(nr>mid) res|=qor(o<<1|1,mid+1,r,nl,nr);
    return res;
}

void dfs1(int u)
{
    siz[u]=1;
    for(int v : g[u])
    {
        if(v==fa[u]) continue;
        dep[v]=dep[u]+1;
        fa[v]=u;dfs1(v);
        siz[u]+=siz[v];
        if(siz[v]>siz[hson[u]]) hson[u]=v;
    }
}

void dfs2(int u,int tp)
{
    top[u]=tp;
    idx[dfn[u]=++dfc]=u;
    if(hson[u]) dfs2(hson[u],tp);
    for(int v : g[u])
        if(v!=fa[u]&&v!=hson[u])
            dfs2(v,v);
}

void pathxor(int u,int v,unsigned c)
{
    while(top[u]!=top[v])
    {
        if(dep[top[u]]<dep[top[v]]) swap(u,v);
        cxor(1,1,n,dfn[top[u]],dfn[u],c);
        u=fa[top[u]];
    }
    if(dep[u]>dep[v]) swap(u,v);
    cxor(1,1,n,dfn[u],dfn[v],c);
}

int query(int u,int v)
{
    unsigned orsum=0,cnt=0;
    while(top[u]!=top[v])
    {
        if(dep[top[u]]<dep[top[v]]) swap(u,v);
        orsum|=qor(1,1,n,dfn[top[u]],dfn[u]);
        cnt+=dfn[u]-dfn[top[u]]+1;
        u=fa[top[u]];
    }
    if(dep[u]>dep[v]) swap(u,v);
    orsum|=qor(1,1,n,dfn[u],dfn[v]);
    cnt+=dfn[v]-dfn[u]+1;
    return 1ll*orsum*pw2[cnt-1]%ha;
}

int main()
{
    scanf("%d%d",&n,&m);
    pw2[0]=1;
    for(int i=1;i<=n;i++)
        pw2[i]=(pw2[i-1]<<1)%ha;;
    for(int i=1,u,v;i<n;i++)
    {
        scanf("%d%d",&u,&v);
        g[u].emplace_back(v);
        g[v].emplace_back(u);
    }
    for(int i=1;i<=n;i++)
        scanf("%d",val+i);
    dfs1(1);dfs2(1,1);
    build(1,1,n);
    int opt,u,v;unsigned c;
    while(m--)
    {
        scanf("%d%d%u",&opt,&u,&v);
        if(opt==1) printf("%d\n",query(u,v));
        else scanf("%u",&c),pathxor(u,v,c);
    }
    return 0;
}
```