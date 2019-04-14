# 【CODECHEF】Children Trips

##### 题目链接：[Children Trips - CODECHEF](https://www.codechef.com/problems/TRIPS)

一道比较有意思的分类讨论题，我们先从暴力说起。

我们定义一个SingleJump(x, p)，它可以计算出从x往上跳到最远的那个与x距离不超过p的点。这个函数非常好实现，只要倍增预处理一下，然后再处理出每个点到根的距离就可以实现了，单次调用复杂度O(log n)

于是我们考虑一个非常暴力的做法：先求出两个询问点的LCA，然后对两个点反复进行SingleJump，两个点都一直跳到最靠近LCA的点（不能跳到LCA以及它的上方），然后记录SingleJump的次数，再对剩余的距离（即最后跳到的位置与LCA的距离）简单处理一下就能得到答案了

这样的暴力，最坏情况下可能需要跳 2n/p 次，因此单次询问复杂度是O(n/p log n)，总复杂度O(m \* n/p log n)。如果p很大，我们会惊奇地发现，这个复杂度是对的！那么，中央已经决定了，当p很大（假设大于T，下面再来分析T的取值）的时候，就把这种方法请到中南海来！

现在来看看p小于T时怎么办。咦？我们是不是可以倍增！如果我们钦定p是一个常量，那么我们可以设jump\[k\]\[x\]表示从x开始，进行2^k次SingleJump到达的点。那么初始时，jump\[0\]\[x\]=SingleJump(x, p)，然后用倍增预处理的套路去搞出整个数组就行了。然后单次询问就可以O(log n)了！

等等，上面地分析似乎是建立在p是常量的基础上的，可是询问给的p不全相等啊？那好办，我们对不超过T的p都处理出这个数组，然后让询问根据p对号入座就行了！考虑到空间复杂度O(T n log n)无法承受，我们选择将p<T的询问按p的大小排序，每做一个p与之前不同的询问时，对当前的p进行一次预处理。预处理是瓶颈，一共要进行T次，故复杂度为O(T n log n)

那么，总复杂度就是O(m \* n/T log n + T n log n)。哎呀，还有什么好考虑的，一看这个T就应该取sqrt(n)，大概取个315就行了。不过复杂度似乎仍然高达5e8，但没关系，我们没有常数（逃

```cpp
#include<bits/stdc++.h>
#define FR first
#define SE second
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
typedef pair<int,int> pii;
const int N=100010,B=315;
struct Edge{int to,capa,next;} e[N<<1];
int h[N],sum=0,n,ans[N];
int fa[18][N],dis[N],dep[N];
int jump[18][N];

struct Query
{
    int s,t,p,id;
    Query(int _s=0,int _t=0,int _p=0,int _id=0):s(_s),t(_t),p(_p),id(_id){}
    bool operator < (const Query &b) const{return p<b.p;}
} qs[N];

void add_edge(int u,int v,int w)
{
    e[++sum].to=v;
    e[sum].capa=w;
    e[sum].next=h[u];
    h[u]=sum;
}

void dfs(int u,int la)
{
    for(int i=1;i<=16;i++)
        fa[i][u]=fa[i-1][fa[i-1][u]];
    for(int t=h[u];t;t=e[t].next)
    {
        int v=e[t].to;
        if(v==la) continue;
        dis[v]=dis[u]+e[t].capa;
        dep[v]=dep[u]+1;
        fa[0][v]=u;dfs(v,u);
    }
}

int LCA(int x,int y)
{
    if(dep[x]<dep[y]) swap(x,y);
    for(int i=16;i>=0;i--)
        if(dep[fa[i][x]]>=dep[y])
            x=fa[i][x];
    if(x==y) return x;
    for(int i=16;i>=0;i--)
        if(fa[i][x]!=fa[i][y])
            x=fa[i][x],y=fa[i][y];
    return fa[0][x];
}

int SingleJump(int x,int p)
{
    int tmp=dis[x];
    for(int i=16;i>=0;i--)
        if(tmp-dis[fa[i][x]]<=p)
            x=fa[i][x];
    return x;
}

pii LargeJump(int x,int aim,int p)
{
    int step=0,rest=0;
    while(x!=aim)
    {
        int y=SingleJump(x,p);
        if(dis[y]>dis[aim]) x=y,step++;
        else rest=dis[x]-dis[aim],x=aim;
    }
    return pii(step,rest);
}

int SolveLarge(int s,int t,int p)
{
    if(s==t) return 0;
    int lca=LCA(s,t);
    pii tmp1=LargeJump(s,lca,p);
    pii tmp2=LargeJump(t,lca,p);
    int ans=tmp1.FR+tmp2.FR;
    return (tmp1.SE+tmp2.SE<=p)?ans+1:ans+2;
}

void Rebuild(int p)
{
    for(int i=1;i<=n;i++)
        jump[0][i]=SingleJump(i,p);
    for(int k=1;k<=16;k++)
        for(int i=1;i<=n;i++)
            jump[k][i]=jump[k-1][jump[k-1][i]];
}

pii SmallJump(int x,int aim,int p)
{
    int step=0;
    for(int i=16;i>=0;i--)
        if(dis[jump[i][x]]>dis[aim])
            x=jump[i][x],step|=1<<i;
    return pii(step,dis[x]-dis[aim]);
}

int SolveSmall(int s,int t,int p)
{
    if(s==t) return 0;
    int lca=LCA(s,t);
    pii tmp1=SmallJump(s,lca,p);
    pii tmp2=SmallJump(t,lca,p);
    int ans=tmp1.FR+tmp2.FR;
    return tmp1.SE+tmp2.SE<=p?ans+1:ans+2;
}

int main()
{
    n=read();
    for(int i=1;i<n;i++)
    {
        int u=read(),v=read(),w=read();
        add_edge(u,v,w);
        add_edge(v,u,w);
    }
    dep[1]=1;dfs(1,0);
    int Q=read(),cnt=0;
    for(int i=1;i<=Q;i++)
    {
        int s=read(),t=read(),p=read();
        if(p<=B) qs[++cnt]=Query(s,t,p,i);
        else ans[i]=SolveLarge(s,t,p);
    }
    sort(qs+1,qs+1+cnt);
    for(int i=1;i<=cnt;i++)
    {
        if(qs[i].p!=qs[i-1].p) Rebuild(qs[i].p);
        ans[qs[i].id]=SolveSmall(qs[i].s,qs[i].t,qs[i].p);
    }
    for(int i=1;i<=Q;i++) print(ans[i]);
    flush();
    return 0;
}
```

