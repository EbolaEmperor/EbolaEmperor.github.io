# 【CODECHEF】Chef and Graph Queries

#### 题目链接：[Chef and Graph Queries  -  CODECHEF](https://www.codechef.com/problems/GERALD07)

说到图的连通性问题，不难想到并查集。其实如果加的是树边，那用莫队套一个可撤销并查集就完成了，然而这是图边，比树的情况复杂了很多

考虑新加入一条边e会造成什么样的影响。有两种可能：

1. 加入后图中出现一个环

2. 连接了两个原本不连通的块

如果是第1种情况，那么我们找到环中最早加入的边p，然后把p断开，再把e连上。显然e会对所有左端点大于p、右端点大于等于e的询问造成影响（让联通块数量减一）。

至于第二种情况，显然e会对所有右端点大于等于e的询问造成影响

那么对于每条边e，我们求出它的p。可以把边转化为点，点权就是边的编号，而原来点的点权就是正无穷，然后用LCT维护路径最小点权就行了

得到所有边的p之后，就变成了这样一个问题：求序列的\[l,r\]区间中，小于l的数有多少个。这就是主席树的经典应用了

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
        putc('\n');
    }
}

using namespace IO;
const int N=400010;
int fa[N],ch[N][2];
int val[N],mn[N];
bool rev[N];
int stk[N],top;
int e[N][2],n,m;

inline void maintain(int x){mn[x]=min(min(mn[ch[x][0]],mn[ch[x][1]]),val[x]);}
inline void pushr(int x){swap(ch[x][0],ch[x][1]);rev[x]^=1;}
inline bool nrt(int x){return ch[fa[x]][0]==x||ch[fa[x]][1]==x;}
inline void pushdown(int x)
{
    if(!rev[x]) return;
    if(ch[x][0]) pushr(ch[x][0]);
    if(ch[x][1]) pushr(ch[x][1]);
    rev[x]^=1;
}

void rotate(int x)
{
    int y=fa[x],z=fa[y];
    int k=(ch[y][1]==x),w=ch[x][k^1];
    if(nrt(y)) ch[z][ch[z][1]==y]=x;
    ch[y][k]=w;ch[x][k^1]=y;
    if(w) fa[w]=y;
    fa[y]=x;fa[x]=z;
    maintain(y);
    maintain(x);
}
void splay(int x)
{
    int y=x,z;
    stk[top=1]=x;
    while(nrt(y)) stk[++top]=fa[y],y=fa[y];
    while(top) pushdown(stk[top--]);
    while(nrt(x))
    {
        y=fa[x];z=fa[y];
        if(nrt(y)) rotate((ch[y][0]==x)^(ch[z][0]==y)?x:y);
        rotate(x);
    }
    maintain(x);
}

void access(int x)
{
    for(int y=0;x;y=x,x=fa[x])
    {
        splay(x);
        ch[x][1]=y;
        maintain(x);
    }
}
int find(int x)
{
    access(x);splay(x);
    while(ch[x][0]) x=ch[x][0];
    return x;
}
void makeroot(int x){access(x);splay(x);pushr(x);}
void link(int x,int y){makeroot(x);fa[x]=y;}
void link(int eid)
{
    int x=e[eid][0],y=e[eid][1];
    link(x,eid+n);link(eid+n,y);
}
void cut(int x,int y)
{
    makeroot(x);access(y);splay(y);
    fa[x]=0;ch[y][0]=0;
    maintain(y);
}
void cut(int eid)
{
    int x=e[eid][0],y=e[eid][1];
    cut(x,eid+n);cut(y,eid+n);
}
int Link(int eid)
{
    int x=e[eid][0],y=e[eid][1];
    if(x==y) return eid;
    makeroot(x);
    if(find(y)!=x)
        return link(eid),0;
    int a=mn[y];
    cut(a);link(eid);
    return a;
}

const int P=N<<4;
int rt[N],lc[P],rc[P],sum[P],tot=0;

void insert(int &o,int p,int l,int r,int k)
{
    sum[o=++tot]=sum[p]+1;
    if(l==r) return;
    int mid=(l+r)/2;
    lc[o]=lc[p];rc[o]=rc[p];
    if(k<=mid) insert(lc[o],lc[p],l,mid,k);
    else insert(rc[o],rc[p],mid+1,r,k);
}
int query(int L,int R,int l,int r,int k)
{
    if(l==r) return sum[R]-sum[L];
    int mid=(l+r)/2;
    if(k<=mid) return query(lc[L],lc[R],l,mid,k);
    return sum[lc[R]]-sum[lc[L]]+query(rc[L],rc[R],mid+1,r,k);
}

void Init()
{
    tot=0;
    memset(fa,0,sizeof(fa));
    memset(ch,0,sizeof(ch));
    memset(lc,0,sizeof(lc));
    memset(rc,0,sizeof(rc));
    memset(rt,0,sizeof(rt));
    memset(sum,0,sizeof(sum));
    memset(rev,0,sizeof(rev));
    for(int i=0;i<=n;i++) val[i]=mn[i]=INT_MAX;
    for(int i=n+1;i<=n+m;i++) val[i]=mn[i]=i-n;
}

int main()
{
    int T,Q,tmp;
    for(T=read();T;T--)
    {
        n=read();m=read();Q=read();Init();
        for(int i=1;i<=m;i++) e[i][0]=read(),e[i][1]=read();
        for(int i=1;i<=m;i++) insert(rt[i],rt[i-1],0,m,Link(i));
        while(Q--)
        {
            int l=read(),r=read();
            int t=query(rt[l-1],rt[r],0,m,l-1);
            print(n-t);flush();
        }
    }
    flush();
    return 0;
}
```

