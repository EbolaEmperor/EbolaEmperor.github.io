# A. 挑战NPC

### 题意简述

给定一个n个点m条边的无向连通图，边权均为1，并给定一个参数p。你需要生成一个新图。新图中的两个点(u,v)有连边，当且仅当原图中u到v的最短路长度不超过p。你需要任意求出新图的一条哈密顿回路。数据保证p≥3

### 题解

考虑一棵树的情况。dfs这棵树，奇数层的点一遍历到就加入答案序列，偶数层的点在遍历完全部子节点后再加入答案序列

显然答案序列中两个相邻点距离不超过3，所以读进来的p根本没用

那对于一个图，我们任意生成一棵树就行了

具体实现时你甚至不需要把这棵树生成出来，只要遍历图的时候加上vis标记，你dfs的顺序实际上就构成了一棵树

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=1010;
vector<int> G[N],ans;
int n,m,p;
bool vis[N];

void dfs(int u,int d)
{
    vis[u]=1;
    if(d&1) ans.push_back(u);
    for(int v : G[u])
        if(!vis[v]) dfs(v,d^1);
    if(~d&1) ans.push_back(u);
}

int main()
{
    scanf("%d%d%d",&n,&m,&p);
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        G[u].push_back(v);
        G[v].push_back(u);
    }
    dfs(1,1);
    for(int x : ans)
        printf("%d ",x);
    return 0;
}
```