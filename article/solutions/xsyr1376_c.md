# C. 钦点

### 题意简述

有n个点，每个点都有点权。两个点之间有连边，当且仅当这两个点权的gcd为合数。现在你可以删去任意一个点，求删去后的最大联通块最小是多少

点数范围10^5，点权范围10^7

### 重要提醒

题目标题不是读作"qīn diǎn"，而是读作"yìn diǎn"。原因不明

### 题解

删去的点必然要是割点，否则删了之后毫无意义，这个非常显然

因为只删一个点，所以我们可以考虑将一个点双“化简”成一个环，对答案不会造成任何影响，但可以大大减少边的规模

于是我们可以认为，含有某个相同合数因子的所有点，构成一个环，环的编号就是这个合数。一开始把环当成链，链头固定，链尾指针随着环中新增的点而移动，最后再把头尾接成环。然后对于一个点，我们分解质因数，枚举两个质数相乘得到该点可以进入的环编号，然后把这个点加入相应的环中

最后要删的点显然位于最大的联通块中，联通块大小用并查集维护一下。于是求最大联通块的割点时，顺便对每个割点求出删去该点对答案的贡献，取最小值，最后再与一开始的次大联通块取个max即可

复杂度瓶颈似乎在于每个点权的质因数分解……

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010;
const int V=10000010;
int head[V],tail[V],tp[V];
int val[V],vcnt=0;
vector<int> G[V];
int a[N],n,T,ans;
int ffa[N],fsz[N];
int dfn[N],low[N],dfc;
int siz[N],all;

int find(int x){return ffa[x]==x?x:ffa[x]=find(ffa[x]);}

void add_edge(int u,int v)
{
    G[u].push_back(v);
    G[v].push_back(u);
    u=find(u);v=find(v);
    if(u!=v) ffa[u]=v,fsz[v]+=fsz[u];
}

void link(int u,int x)
{
    static int p[100],cnt[100];
    int tt=0,num;
    for(int v=2;v*v<=x&&x>1;v++)
    {
        if(x%v) continue;
        cnt[++tt]=0;p[tt]=v;
        while(x%v==0) x/=v,cnt[tt]++;
    }
    if(x>1) cnt[++tt]=1,p[tt]=x;
    for(int i=1;i<=tt;i++)
        for(int j=1;j<=i;j++)
        {
            if(i==j&&cnt[i]<2) continue;
            int v=p[i]*p[j];
            if(tp[v]!=T) tp[v]=T,tail[v]=0;
            if(!tail[v]) head[v]=u,val[++vcnt]=v;
            else add_edge(u,tail[v]);
            tail[v]=u;
        }
}

void tarjan(int u,int fa)
{
    siz[u]=1;
    dfn[u]=low[u]=++dfc;
    int mxsz=0,ssz=1;
    for(int v : G[u])
    {
        if(v==fa) continue;
        if(!dfn[v])
        {
            tarjan(v,u);
            siz[u]+=siz[v];
            low[u]=min(low[u],low[v]);
            if(low[v]>=dfn[u]) mxsz=max(mxsz,siz[v]),ssz+=siz[v];
        }
        else low[u]=min(low[u],dfn[v]);
    }
    mxsz=max(mxsz,all-ssz);
    ans=min(ans,mxsz);
}

int main()
{
    for(scanf("%d",&T);T;T--)
    {
        scanf("%d",&n);vcnt=0;
        for(int i=1;i<=n;i++) G[i].clear(),ffa[i]=i,fsz[i]=1;
        for(int i=1;i<=n;i++) scanf("%d",a+i);
        for(int i=1;i<=n;i++) link(i,a[i]);
        for(int i=1;i<=vcnt;i++)
        {
            int x=val[i];
            if(head[x]==tail[x]) continue;
            add_edge(head[x],tail[x]);
        }
        int mx=0,mxs=0,rt;
        for(int i=1;i<=n;i++)
        {
            if(ffa[i]!=i) continue;
            if(fsz[i]>mx) mxs=mx,all=mx=fsz[i],rt=i;
            else if(fsz[i]>mxs) mxs=fsz[i];
        }
        memset(dfn,0,sizeof(dfn));
        ans=INT_MAX;dfc=0;tarjan(rt,0);
        printf("%d\n",max(ans,mxs));
    }
    return 0;
}
```