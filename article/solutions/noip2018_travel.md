# 【NOIP2018】旅行

比赛的时候写的错解，洛谷自测80分，官方数据AC，爽到

其实很简单的一道题。树就不多说，直接vector存边（方便排序），然后直接对所有点的出边进行一次sort操作，然后直接求dfs序即可

如果有环，我们可以枚举断边。选一条环上的边断开，然后按树去做，最后取字典序最小的那个答案即可

断边可以直接在邻接表上操作，也可以像我这个傻逼一样，到了断边就返回，把回第一个环上点的路上还没走过的子树走完，然后从第一个环上点走，再把第一个环上点到1的路上还没走过的子树走完。这种写法极其傻逼，考场上我甚至把上述最后一步给忘了……

找环就不用多说了吧，维护一个stack记录dfs的路径，引入visit标记，找到第一个标记过的点，就在stack上弹回它那里去

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int N=5010;
vector<int> G[N];
int n,m,ans[N],res[N],tt[N],dfc=0;
bool inlp[N],vis[N];
int dep[N],fa[N];
stack<int> pth;
int rnnn=0;
 
namespace Tree
{
    void dfs(int u,int fa)
    {
        ans[++dfc]=u;
        for(int v=0;v<G[u].size();v++)
            if(G[u][v]!=fa) dfs(G[u][v],u);
    }
     
    int main()
    {
        dfs(1,0);
        for(int i=1;i<=dfc;i++)
            printf("%d ",ans[i]);
        return 0;
    }
}
 
bool findloop(int u,int la)
{
    if(vis[u])
    {
        while(pth.top()!=u)
        {
            inlp[pth.top()]=1;
            pth.pop();
        }
        inlp[u]=1;
        return 1;
    }
    vis[u]=1;pth.push(u);
    for(int i=0;i<G[u].size();i++)
    {
        int v=G[u][i];
        if(v==la) continue;
        dep[v]=dep[u]+1;fa[v]=u;
        if(findloop(v,u)) return 1;
    }
    pth.pop();
    return 0;
}
 
bool dfs(int u,int la,int rtn)
{
    if(vis[u]) return 0;
    ans[++dfc]=u;vis[u]=1;
    if(!rnnn&&inlp[u]) rnnn=u;
    for(int i=0;i<G[u].size();i++)
    {
        int v=G[u][i];
        if(v==la) continue;
        if(u==rtn&&inlp[v]) continue;
        if(dfs(v,u,rtn)) return 1;
    }
    return u==rtn;
}
 
void gao(int rtn)
{
    dfc=0;
    memset(vis,0,sizeof(vis));
    bool fuck=dfs(1,0,rtn);
    int cnt=0;
    if(fuck)
    {
        int ffuck=0;
        for(int k=dfc;k>=1;k--)
        {
            int my=ans[k];
            if(my==rnnn){ffuck=k;break;}
            for(int i=0;i<G[my].size();i++)
                if(!inlp[G[my][i]]) dfs(G[my][i],my,-1);
        }
        for(int i=1;i<=dfc;i++) tt[++cnt]=ans[i];
        dfc=-1;vis[rnnn]=0;dfs(rnnn,0,-1);
        for(int i=1;i<=dfc;i++) tt[++cnt]=ans[i];
        dfc=0;
        for(int k=ffuck;k>=1;k--)
        {
            int my=tt[k];
            for(int i=0;i<G[my].size();i++)
                if(!inlp[G[my][i]]) dfs(G[my][i],my,-1);
        }
        for(int i=1;i<=dfc;i++) tt[++cnt]=ans[i];
    }
    else memcpy(tt,ans,sizeof(tt));
    for(int i=1;i<=n;i++)
    {
        if(tt[i]>res[i]) break;
        if(tt[i]<res[i])
        {
            memcpy(res,tt,sizeof(res));
            break;
        }
    }
}
 
int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        G[u].push_back(v);
        G[v].push_back(u);
    }
    for(int i=1;i<=n;i++)
        sort(G[i].begin(),G[i].end());
    if(m==n-1) return Tree::main();
    dep[1]=1;findloop(1,0);
    memset(res,0x3f,sizeof(res));
    for(int i=1;i<=n;i++)
        if(inlp[i]) gao(i);
    for(int i=1;i<=n;i++)
        printf("%d ",res[i]);
    return 0;
}
```