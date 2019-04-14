# 斯坦纳树  学习笔记

### 问题模型

给定一个无向图，图中有若干个关键点，你需要选出一些边使得所有关键点能连通，最小化边权之和。图比较大，但关键点的数量极少（通常不超过10）

典例：[5180. Cities  -  BZOJ](https://www.lydsy.com/JudgeOnline/problem.php?id=5180)

### 解决方案

考虑到关键点数量极少，我们可以将关键点的连通性进行状压，然后dp。设f\[s\]\[x\]表示在连通状态为s，并且包含点x的前提下，最小的花费是多少。二进制状态s的第i位为1，表示点i与点x连通

那么第一种转移就是将自己的两个子集合并。即：![](http://latex.codecogs.com/svg.latex?f_{s,x}=\min_{t\subset&space;s}(f_{t,x}+f_{\complement_st,x}))

第二种就是从别的点转移过来，不改变连通状态，即：![](http://latex.codecogs.com/svg.latex?f_{s,x}=\min_{e=(u,x)}(f_{s,u}+capa_e))

对于第一种转移，可以直接枚举子集，然后转移。对于第二种转移，它其实就是一个最短路，直接对于每种连通状态都跑一遍spfa就行了。当然BZOJ5180那道题，它会卡spfa，换成dijkstra就行了

枚举子集有一个小技巧：

```cpp
for(int sub=(s-1)&s;sub;sub=(sub-1)&s)
```

这样恰好能不重不漏地枚举s的所有子集

### 代码实现

这里就放一下BZOJ5180这道题的代码，非常简明易懂

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
typedef pair<LL,int> pli;
const int N=100010;
const LL INF=0x3f3f3f3f3f3f3f3fLL;
struct Edge{int to,capa,next;} e[N<<2];
int h[N],sum=0,n,m,tot;
priority_queue<pli> q;
bool vis[N];
LL f[35][N];

void add_edge(int u,int v,int w)
{
    e[++sum].to=v;
    e[sum].capa=w;
    e[sum].next=h[u];
    h[u]=sum;
}

void dijkstra(int s)
{
    memset(vis,0,sizeof(vis));
    while(!q.empty())
    {
        int u=q.top().second;q.pop();
        if(vis[u]) continue;vis[u]=1;
        for(int t=h[u];t;t=e[t].next)
        {
            int v=e[t].to;
            if(f[s][u]+e[t].capa<f[s][v])
            {
                f[s][v]=f[s][u]+e[t].capa;
                q.push(pli(-f[s][v],v));
            }
        }
    }
}

int main()
{
    int x,u,v,w;
    scanf("%d%d%d",&n,&tot,&m);
    memset(f,0x3f,sizeof(f));
    for(int i=0;i<tot;i++)
    {
        scanf("%d",&x);
        f[1<<i][x]=0;
    }
    for(int i=1;i<=m;i++)
    {
        scanf("%d%d%d",&u,&v,&w);
        add_edge(u,v,w);
        add_edge(v,u,w);
    }
    for(int s=1;s<(1<<tot);s++)
    {
        for(int i=1;i<=n;i++)
        {
            for(int t=(s-1)&s;t;t=(t-1)&s)
                f[s][i]=min(f[s][i],f[t][i]+f[s^t][i]);
            if(f[s][i]<INF) q.push(pli(-f[s][i],i));
        }
        dijkstra(s);
    }
    LL ans=INF;
    for(int i=1;i<=n;i++)
        ans=min(ans,f[(1<<tot)-1][i]);
    printf("%lld\n",ans);
    return 0;
}
```

