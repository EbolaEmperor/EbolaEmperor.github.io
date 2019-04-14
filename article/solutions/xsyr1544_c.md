# C. Flash

### 题意简述

一棵树上每个点都有一个任务，每个任务都需要一定时间，非相邻点的任务可以同时进行。最小化所有任务结束时刻之和的最小值。点数不超过2000

### 题解

大结论：任意一个点x对答案的贡献，都是从x出发的某条路径经过的点权之和

预处理从每个点出发的所有路径权值，并对其排序

设f\[u\]\[x\]表示点u选择了权值第x大的路径，此时u及其子树的最小答案是多少

转移时直接在子节点的路径权值数组上二分查找即可。为了保证复杂度，需要处理出f[u]的前缀后缀min

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const LL INF=1ll<<50;
const int N=2010;
vector<int> G[N];
int n,val[N];
LL dis[N][N];
LL f[N][N],pre[N][N],suf[N][N];

void dfs(int rt,int u,int fa)
{
    dis[rt][u]=dis[rt][fa]+val[u];
    for(int v : G[u]) if(v!=fa) dfs(rt,v,u);
}

void dp(int u,int fa)
{
    for(int i=1;i<=n;i++) f[u][i]=dis[u][i];
    for(int v : G[u])
    {
        if(v==fa) continue;
        dp(v,u);
        for(int j=1;j<=n;j++)
        {
            LL d=dis[u][j],ans=INF;
            ans=min(ans,pre[v][upper_bound(dis[v]+1,dis[v]+1+n,d-val[u])-dis[v]-1]);
            ans=min(ans,suf[v][lower_bound(dis[v]+1,dis[v]+1+n,d+val[v])-dis[v]]);
            f[u][j]+=ans;
        }
    }
    pre[u][0]=suf[u][n+1]=INF;
    for(int i=1;i<=n;i++) pre[u][i]=min(pre[u][i-1],f[u][i]);
    for(int i=n;i>=1;i--) suf[u][i]=min(suf[u][i+1],f[u][i]); 
}

int main()
{
    scanf("%d",&n);
    for(int i=1,u,v;i<n;i++)
    {
        scanf("%d%d",&u,&v);
        G[u].push_back(v);
        G[v].push_back(u);
    }
    for(int i=1;i<=n;i++) scanf("%d",val+i);
    for(int i=1;i<=n;i++) dfs(i,i,i),sort(dis[i]+1,dis[i]+1+n);
    dp(1,0);printf("%lld\n",suf[1][1]);
    return 0;
}
```

