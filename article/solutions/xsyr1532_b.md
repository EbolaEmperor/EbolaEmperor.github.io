# B. game

### 题意简述

给定一棵n(n≤1e5)个节点的树，每次等概率地选择一个连通块，将该连通块内的所有点都锤一遍，然后在选定连通块中随机删去一个点以及该点的所有连边，求所有点被锤次数的期望与n!的乘积。答案对1e9+7取模

### 题解

对每个点单独考虑计算被锤的期望。对于一个点u，它被锤的期望显然可以转化为另一个问题：在一棵n个节点以u为根的树上，每次随机删去一个子树，将整棵树删完为止，求删除次数的期望。这是一个经典问题，我们有一个结论，答案就是所有点深度+1的倒数之和。于是我们可以枚举所有点，将它设为根，然后求出答案，再全部加起来就是总的答案了

O(n²)的复杂度显然不能接受。仔细思考，其实可以发现，对于任何一对距离为d的点对，它们对答案造成的贡献都是d+1的倒数。于是我们可以对每个距离d，计算有多少距离为d的点对。统计经过某个节点u的点对数量，显然就是对于u的所有子树，求出所有点深度的生成函数，然后卷积起来。卷积可以FFT优化，然后外层套个点分治即可

```cpp
#include<bits/stdc++.h>
using namespace std;

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

typedef long long LL;
const int ha=1e9+7;
const int N=100010;
const double pi=acos(-1);
struct Edge{int to,next;} e[N<<1];
int h[N],esum=0,n;
int mxsz[N],sum,rt;
int sz[N],dep[N],son[N];
int pp[N],tt[N],pson[N];
bool done[N];
int inv[N],cnt[N];

struct Comp
{
    double r,i;
    Comp(double _r=0,double _i=0):r(_r),i(_i){}
    Comp operator + (const Comp &a){return Comp(r+a.r,i+a.i);}
    Comp operator - (const Comp &a){return Comp(r-a.r,i-a.i);}
    Comp operator * (const Comp &a){return Comp(r*a.r-i*a.i,r*a.i+i*a.r);}
} A[N<<2],B[N<<2];
int r[N<<2];

void add_edge(int u,int v)
{
    e[++esum].to=v;
    e[esum].next=h[u];
    h[u]=esum;
}

void FFT(Comp *A,int n,int v)
{
    for(int i=0;i<n;i++) if(i<r[i]) swap(A[i],A[r[i]]);
    for(int i=1;i<n;i<<=1)
    {
        Comp wn(cos(pi/i),v*sin(pi/i));
        for(int j=0;j<n;j+=i<<1)
        {
            Comp w(1,0);
            for(int k=0;k<i;k++,w=w*wn)
            {
                Comp x=A[j+k],y=w*A[i+j+k];
                A[j+k]=x+y;A[i+j+k]=x-y;
            }
        }
    }
}

void polymul(int *a,int n,int *b,int m,int *res)
{
    if(n<=60||m<=60)
    {
        memset(res,0,sizeof(int)*(n+m+1));
        for(int i=0;i<=n;i++)
            for(int j=0;j<=m;j++)
                res[i+j]=(res[i+j]+(LL)a[i]*b[j])%ha;
        return;
    }
    int len=1,l=0;
    for(;len<=n+m;len<<=1,l++);
    for(int i=0;i<len;i++) r[i]=(r[i/2]/2)|((i&1)<<l-1);
    for(int i=0;i<=n;i++) A[i]=Comp(a[i],0);
    for(int i=0;i<=m;i++) B[i]=Comp(b[i],0);
    for(int i=n+1;i<len;i++) A[i]=Comp(0,0);
    for(int i=m+1;i<len;i++) B[i]=Comp(0,0);
    FFT(A,len,1);FFT(B,len,1);
    for(int i=0;i<len;i++) A[i]=A[i]*B[i];
    FFT(A,len,-1);
    for(int i=0;i<=n+m;i++)
        res[i]=(LL)round(A[i].r/len)%ha;
}

void polyadd(int *a,int *b,int n)
{
    for(int i=0;i<=n;i++)
        a[i]=(a[i]+b[i]>=ha)?(a[i]+b[i]-ha):(a[i]+b[i]);
}

void find(int u,int fa)
{
    sz[u]=1;mxsz[u]=0;
    for(int t=h[u];t;t=e[t].next)
    {
        int v=e[t].to;
        if(v==fa||done[v]) continue;
        find(v,u);sz[u]+=sz[v];
        mxsz[u]=max(mxsz[u],sz[v]);
    }
    mxsz[u]=max(mxsz[u],sum-sz[u]);
    if(mxsz[u]<mxsz[rt]) rt=u;
}

void getsz(int u,int fa)
{
    sz[u]=1;dep[u]=0;
    for(int t=h[u];t;t=e[t].next)
    {
        int v=e[t].to;
        if(v==fa||done[v]) continue;
        getsz(v,u);sz[u]+=sz[v];
        dep[u]=max(dep[u],dep[v]+1);
    }
}

void getpoly(int u,int fa,int d)
{
    pson[d]++;
    for(int t=h[u];t;t=e[t].next)
    {
        int v=e[t].to;
        if(v==fa||done[v]) continue;
        getpoly(v,u,d+1);
    }
}

void solve(int u)
{
    done[u]=1;getsz(u,0);
    int pplen=0,sn=0;pp[0]=1;
    for(int t=h[u];t;t=e[t].next)
    {
        int v=e[t].to;
        if(done[v]) continue;
        son[++sn]=v;
    }
    sort(son+1,son+1+sn,[](int x,int y){return dep[x]<dep[y];});
    for(int i=1;i<=sn;i++)
    {
        int v=son[i];
        getpoly(v,u,1);
        polymul(pp,pplen,pson,dep[v]+1,tt);
        polyadd(cnt,tt,pplen+dep[v]+1);
        pplen=max(pplen,dep[v]+1);
        polyadd(pp,pson,pplen);
        memset(pson,0,sizeof(int)*(dep[v]+2));
    }
    memset(pp,0,sizeof(int)*(dep[u]+2));
    for(int t=h[u];t;t=e[t].next)
    {
        int v=e[t].to;
        if(done[v]) continue;
        rt=0;sum=sz[v];find(v,u);
        solve(rt);
    }
}

int main()
{
    n=read();inv[1]=1;
    for(int i=2;i<=n;i++)
        inv[i]=(ha-(LL)(ha/i)*inv[ha%i]%ha)%ha;
    for(int i=1,u,v;i<n;i++)
    {
        u=read();v=read();
        add_edge(u,v);
        add_edge(v,u);
    }
    mxsz[0]=n+1;sum=n;rt=0;
    find(1,0);solve(rt);
    int ans=n,fac=n;
    for(int i=1;i<n;i++)
    {
        cnt[i]=(cnt[i]<<1)%ha;
        ans=(ans+(LL)cnt[i]*inv[i+1])%ha;
        fac=(LL)fac*i%ha;
    }
    ans=(LL)ans*fac%ha;
    printf("%d\n",ans);
    return 0;
}
```

