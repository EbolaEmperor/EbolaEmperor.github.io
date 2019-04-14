# 【CF97E】Leaders

根据题意不难想到一个具有一定正确性的错误做法

先任意建出一棵生成树，然后对于一组询问，如果两个点深度的奇偶性不同，则存在长度为奇数的路径

那么两个点深度的奇偶性相同呢？

如果图中存在奇环，则凡是经过奇环的路径，都可以利用这个奇环去改变路径长度的奇偶性。于是只要一条路径经过了奇环，那无论如何它都可以变成奇数长度了

于是我们可以对于一条非树边，若它连接的两个点深度的奇偶性相同，则给这两个点的路径上打标记。对于一组两个点奇偶性相同的询问，看路径上是否存在打了标记的边，存在则答案为Yes

为了支持路径打标记+询问，我们需要写一个树剖套线段树

这样的做法在我们的模拟赛上可以拿到85分的高分，但并不能AC

上述做法存在的问题是非常明显的，随便手完一组数据都能把它hack掉

那在说能AC的做法之前，我们把之前的做法优化一下。发现所有修改操作都在询问之前，于是可以用树上差分来解决。修改操作用差分实现，然后两遍还原操作得到前缀和（注意两遍还原操作的方向不一样，具体看代码），然后询问直接用前缀和差分询问就行了。复杂度只有求lca的一个log

为什么要优化原本的算法呢？因为我们要用这样的做法反复做很多次！没错，随机！每次把遍历顺序随机打乱，再把边表随机打乱，然后根据随机的顺序来做上面的操作。随机10次，既能保证正确性，又能保证时间

注意到我们的做法只会把Yes误判成No，因此每次随机，处理询问时只处理当前答案是No的即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010;
vector<int> G[N];
int n,m,Q,dep[N],fa[N],ffa[N];
int hson[N],top[N],sz[N],sum[N];
int dfn[N],dfc=0;
bool vis[N],ans[N];
int qu[N],qv[N];
int idx[N];

int find(int x){return ffa[x]==x?x:ffa[x]=find(ffa[x]);}

void dfs1(int u)
{
    vis[u]=1;sz[u]=1;
    for(int v : G[u])
    {
        if(vis[v]) continue;
        dep[v]=dep[u]+1;
        fa[v]=u;dfs1(v);
        sz[u]+=sz[v];
        if(sz[v]>sz[hson[u]]) hson[u]=v;
    }
}

void dfs2(int u,int tp)
{
    dfn[u]=++dfc;top[u]=tp;
    if(hson[u]) dfs2(hson[u],tp);
    for(int v : G[u])
        if(fa[v]==u&&v!=hson[u])
            dfs2(v,v);
}

int LCA(int x,int y)
{
    while(top[x]!=top[y])
    {
        if(dep[top[x]]<dep[top[y]]) swap(x,y);
        x=fa[top[x]];
    }
    return dep[x]<dep[y]?x:y;
}

bool query(int x,int y)
{
    if(find(x)!=find(y)) return 0;
    if((dep[x]&1)!=(dep[y]&1)) return 1;
    return sum[x]+sum[y]-2*sum[LCA(x,y)];
}

void build(int u)
{
    vis[u]=1;
    for(int v : G[u])
    {
        if(fa[v]==u||fa[u]==v) continue;
        if((dep[u]&1)!=(dep[v]&1)) continue;
        sum[u]++;sum[v]++;sum[LCA(u,v)]-=2;
    }
    for(int v : G[u])
        if(!vis[v]&&fa[v]==u) build(v);
}

void reduction1(int u)
{
    for(int v : G[u])
    {
        if(fa[v]!=u) continue;
        reduction1(v);
        sum[u]+=sum[v];
    }
}

void reduction2(int u)
{
    for(int v : G[u])
    {
        if(fa[v]!=u) continue;
        sum[v]+=sum[u];
        reduction2(v);
    }
}

void clear()
{
    memset(vis,0,sizeof(vis));
    memset(top,0,sizeof(top));
    memset(hson,0,sizeof(hson));
    memset(sz,0,sizeof(sz));
    memset(dep,0,sizeof(dep));
    memset(sum,0,sizeof(sum));
    memset(fa,0,sizeof(fa));
    dfc=0;memset(dfn,0,sizeof(dfn));
}

int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++) ffa[i]=i;
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        G[u].push_back(v);
        G[v].push_back(u);
        if(find(u)!=find(v))
            ffa[find(u)]=find(v);
    }
    scanf("%d",&Q);
    for(int i=1;i<=Q;i++)
        scanf("%d%d",qu+i,qv+i);
    for(int i=1;i<=n;i++) idx[i]=i;
    for(int tms=7;tms;tms--)
    {
        random_shuffle(idx+1,idx+1+n);
        for(int i=1;i<=n;i++) random_shuffle(G[i].begin(),G[i].end());
        clear();
        for(int i=1;i<=n;i++)
            if(!vis[idx[i]]) dfs1(idx[i]),dfs2(idx[i],idx[i]);
        memset(vis,0,sizeof(vis));
        for(int i=1;i<=n;i++)
            if(!vis[idx[i]])
            {
                build(idx[i]);
                reduction1(idx[i]);
                reduction2(idx[i]);
            }
        for(int i=1;i<=Q;i++)
            if(!ans[i]) ans[i]=query(qu[i],qv[i]);
    }
    for(int i=1;i<=Q;i++)
        puts(ans[i]?"Yes":"No");
    return 0;
}
```

