# A. 配对

### 题意简述

将树上的k个点两两配对，使得所有点对的距离之和最大

### 题解

结论题

将所有点按dfs序排序，排序后第 i (i≤k/2) 个点与第 i+k/2 个点配对即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010;
vector<int> G[N];
int dfn[N],n,k,dfc=0,p[N];

void dfs(int u,int fa)
{
    dfn[u]=++dfc;
    for(int v : G[u])
        if(v!=fa) dfs(v,u);
}

int main()
{
    scanf("%d%d",&n,&k);
    for(int i=1,u,v,w;i<n;i++)
    {
        scanf("%d%d%d",&u,&v,&w);
        G[u].push_back(v);
        G[v].push_back(u);
    }
    dfs(1,0);
    for(int i=1;i<=k;i++) scanf("%d",p+i);
    sort(p+1,p+1+k,[](int a,int b){return dfn[a]<dfn[b];});
    for(int i=1;i<=k/2;i++) printf("%d %d\n",p[i],p[k/2+i]);
    return 0;
}
```

