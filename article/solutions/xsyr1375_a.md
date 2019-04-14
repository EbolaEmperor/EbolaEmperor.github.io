# A. 王子

### 题意简述

你每天可以做选择，第i天选A会得到Ai的收益，选B会得到Bi的收益。连续k天必须至少P天选A、至少Q天选B。求最大收益

### 题解

费用流经典题

首先假设每天都选B，然后初始答案为Bi之和。接下来考虑哪几天改选A

建模：建立n个点，排成一排，每个点x向x+1连边，称之为“横边”，同时向x+k连边，称之为“桥边”。横边表示保留选B，桥边表示改选A

对于每个点x，在他的右边画一条纵截线，与纵截线相交的桥边流量，就是第x-k+1到第x天改选A的天数

考虑改变题目限制的含义。至少Q天选B，就是至多k-Q天选A，我们记r=k-Q

于是与任意一条纵截线相交的桥边流量都应该至少为P，至多为r，剩余的流量都走横边。上界的控制可以从源点控制，控制源点流出的流量为r即可。下界的控制可以将横边流量设为r-P，这样超过了r-P的流量就不得不走桥边过，也就控制了桥边流量至少为P

由于每天只能做一次选择，也就是说每个点i只能进行一次改选A的操作，取得的收益是Ai-Bi。所以点i连出去的桥边容量为1，费用为Ai-Bi。横边表示保留选择，因此没有收益，所以容量为r-P，费用为0

源汇点的连边没什么好说的，看代码吧。最后跑一遍最大费用最大流即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=20010;
const int INF=0x3f3f3f3f;
struct Edge{int from,to,capa,flow,cost,next;} e[N];
int h[N],sum=-1,n,k,l,r,s,t;
int dis[N],pre[N],able[N];
int v1[N],v2[N],ans=0;
bool inq[N];

void add_edge(int u,int v,int w,int c)
{
    e[++sum]={u,v,w,0,c,h[u]};h[u]=sum;
    e[++sum]={v,u,0,0,-c,h[v]};h[v]=sum;
}

bool spfa(int &cost)
{
    memset(inq,0,sizeof(inq));
    for(int i=0;i<=n+5;i++) dis[i]=-INF;
    queue<int> q;q.push(s);
    dis[s]=0;inq[s]=1;able[s]=INF;
    while(!q.empty())
    {
        int u=q.front();
        for(int tmp=h[u];~tmp;tmp=e[tmp].next)
        {
            int v=e[tmp].to;
            if(dis[u]+e[tmp].cost>dis[v]&&e[tmp].capa>e[tmp].flow)
            {
                dis[v]=dis[u]+e[tmp].cost;
                able[v]=min(able[u],e[tmp].capa-e[tmp].flow);
                pre[v]=tmp;
                if(inq[v]) continue;
                q.push(v);inq[v]=1;
            }
        }
        q.pop();inq[u]=0;
    }
    if(dis[t]==-INF) return 0;
    cost+=able[t]*dis[t];
    for(int u=t;u!=s;u=e[pre[u]].from)
    {
        e[pre[u]].flow+=able[t];
        e[pre[u]^1].flow-=able[t];
    }
    return 1;
}

int main()
{
    memset(h,-1,sizeof(h));
    scanf("%d%d%d%d",&n,&k,&l,&r);
    for(int i=1;i<=n;i++)
        scanf("%d%d",v1+i,v2+i),ans+=v2[i];
    s=0;t=n+1;r=k-r;
    add_edge(s,n+2,r,0);
    for(int i=1;i<=k;i++)
        add_edge(n+2,i,1,0);
    for(int i=1;i<=n;i++)
    {
        add_edge(i,i+1,r-l,0);
        add_edge(i,i+k<t?i+k:t,1,v1[i]-v2[i]);
    }
    while(spfa(ans));
    printf("%d\n",ans);
    return 0;
}
```