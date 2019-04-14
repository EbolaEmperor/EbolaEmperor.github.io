# 【NOIP2018】赛道修建

首先二分答案显然

那么二分到答案之后，我们想要求得最多能分出多少条长度不小于mid的路径

考虑贪心的策略。对于一个节点u，如果存在某个子树节点到u的路径长度不小于mid，则将该条路径删除并让答案加一。如果存在某两条子树节点到u的路径长度加起来不小于mid，则将这两条路径删除并让答案加一。那么剩下的路径中，我们取最长的那条往上传就好了

上述过程可以使用multiset优化，当然排序后二分什么的也可以。我还是用了multiset，具体实现细节见代码

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int N=50010;
struct Edge{int to,capa,next;} e[N<<1];
int h[N],n,m,sum=0;
int f[N],cnt,mid;
 
void add_edge(int u,int v,int w)
{
    e[++sum].to=v;
    e[sum].capa=w;
    e[sum].next=h[u];
    h[u]=sum;
}
 
void dp(int u,int fa)
{
    multiset<int> S;
    multiset<int>::iterator it;
    for(int t=h[u];t;t=e[t].next)
    {
        int v=e[t].to;
        if(v==fa) continue;
        dp(v,u);
        S.insert(f[v]+e[t].capa);
    }
    while(!S.empty())
    {
        it=S.end();
        int x=*(--it);
        if(x>=mid) cnt++,S.erase(it);
        else break;
    }
    while(!S.empty())
    {
        it=S.begin();
        int x=*it;S.erase(it);
        it=S.lower_bound(mid-x);
        if(it!=S.end()) cnt++,S.erase(it);
        else f[u]=max(f[u],x);
    }
}
 
bool check()
{
    memset(f,0,sizeof(f));
    cnt=0;dp(1,0);
    return cnt>=m;
}
 
int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1,u,v,w;i<n;i++)
    {
        scanf("%d%d%d",&u,&v,&w);
        add_edge(u,v,w);
        add_edge(v,u,w);
    }
    int l=0,r=n*10000;
    while(l<=r)
    {
        mid=(l+r)/2;
        if(check()) l=mid+1;
        else r=mid-1;
    }
    mid=l;
    printf("%d\n",check()?l:l-1);
    return 0;
}
```