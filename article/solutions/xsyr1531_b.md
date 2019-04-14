# B. 东非大裂谷

### 题意简述

给定一棵n个节点的树，你需要将其划分成若干条竖直链，每条链对答案的贡献是链上点权的最大最小值之差，最小化所有贡献之和

### 题解

考虑对于一条链，点权自底向上必然具有单调性（否则断一下肯定更优）

对于点u，设f\[u\]表示当u继承一条自底向上递增链时的信息，信息包含两个内容：链底元素的值、与该条链相连的所有子树答案之和。g\[u\]则维护当u继承一条自底向上递减链时的信息，内容同上

然后树形dp就完事了，节点u继承的链应该是能使整棵子树答案最大的那条链

```cpp
#include<bits/stdc++.h>
#define FR first
#define SE second
using namespace std;

typedef long long LL;
typedef pair<LL,LL> pll;
const int N=100010;
vector<int> G[N];
pll f[N],g[N];
LL h[N],w[N],n;

void dfs(int u)
{
    for(int v : G[u]) dfs(v),h[u]+=h[v];
    f[u]=g[u]=pll(w[u],h[u]);
    for(int v : G[u])
    {
        LL rest=h[u]-h[v];
        if(w[u]>=w[v])
        {
            pll pfv(f[v].FR,rest+f[v].SE);
            if(pfv.SE-pfv.FR>f[u].SE-f[u].FR) f[u]=pfv;
        }
        if(w[u]<=w[v])
        {
            pll pgv(g[v].FR,rest+g[v].SE);
            if(pgv.SE+pgv.FR>g[u].SE+g[u].FR) g[u]=pgv;
        }
    }
    h[u]=max(f[u].SE-f[u].FR+w[u],g[u].SE+g[u].FR-w[u]);
}

int main()
{
    scanf("%lld",&n);
    for(int i=1;i<=n;i++)
        scanf("%lld",w+i);
    for(int i=1,u,v;i<n;i++)
    {
        scanf("%d%d",&u,&v);
        G[u].push_back(v);
    }
    dfs(1);
    printf("%lld\n",h[1]);
    return 0;
}
```