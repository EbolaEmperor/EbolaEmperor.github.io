# 【IOI2018】高速公路收费

首先将所有边设为A，求出s到t之间的距离dis

考虑二分去找一条s到t最短路径上的边rte。设tot为边的总数，将\[mid,tot\]设为B，其余设为A，若s到t之间的距离仍为dis，则这条边在\[l,mid\]中，否则在\[mid,r\]中

设rte的两个顶点为u和v，则将所有点按到u和v的距离划分成两个点集，点集X中所有点距离u较近，点集Y中所有点距离v较近。由于rte位于s到t的最短路中，于是不难发现，s和t一定在不同点集中

这么一来，我们可以从u出发进行bfs，对点集X中所有点求出bfs序。容易发现，一定存在一条u到s的最短路径使得s的bfs序是路径上所有点中最大的，于是可以二分求s。具体地，对于bfs序位于\[mid,size_X\]中的所有点，将与之相邻的边全部设为B，若s到t之间的距离仍为dis，则s的bfs序在\[l,mid\]中，否则在\[mid,r\]中。求t也是类似的

```cpp
#include<bits/stdc++.h>
#include "highway.h"
using namespace std;

typedef long long LL;
const int N=90010,M=130010;
struct Edge{int u,v;} e[M];
vector<int> g[N],stu;
int n,lena,lenb,tot;
int depx[N],depy[N];
int bfnx[N],bfny[N];
int idxx[N],idxy[N];
LL dis;

int findroot()
{
    int l=0,r=tot-1,mid,res=0;
    while(l<=r)
    {
        mid=(l+r)/2;
        for(int i=0;i<tot;i++) stu[i]=(i>=mid);
        LL tmp=ask(stu);
        if(tmp==dis) r=mid-1;
        else l=mid+1,res=mid;
    }
    return res;
}

void bfs(int rt,int *dep,int *bfn,int *idx)
{
    int bfc=0;
    static bool vis[N];
    queue<int> q;q.push(rt);
    memset(vis,0,sizeof(vis));
    vis[rt]=1;
    while(!q.empty())
    {
        int u=q.front();q.pop();
        idx[bfn[u]=bfc]=u;bfc++;
        for(int v : g[u])
        {
            if(vis[v]) continue;
            dep[v]=dep[u]+1;
            q.push(v);vis[v]=1;
        }
    }
}

int gao(int *depx,int *depy,int *bfn,int *idx)
{
    int cnt=0;
    static int p[N],idp[N];
    static bool in[N];
    memset(p,0,sizeof(p));
    memset(in,0,sizeof(in));
    for(int i=0;i<n;i++)
        if(depx[idx[i]]<depy[idx[i]]) in[idx[i]]=1,idp[p[idx[i]]=cnt]=idx[i],cnt++;
    int l=0,r=cnt-1,mid,res=0;
    while(l<=r)
    {
        mid=(l+r)/2;
        for(int i=0;i<tot;i++)
            stu[i]=(in[e[i].u]&&p[e[i].u]>=mid||in[e[i].v]&&p[e[i].v]>=mid);
        LL tmp=ask(stu);
        if(tmp==dis) r=mid-1;
        else l=mid+1,res=mid;
    }
    return idp[res];
}

void find_pair(int tmpn,vector<int> U,vector<int> V,int tmpa,int tmpb)
{
    n=tmpn;lena=tmpa;lenb=tmpb;
    stu.resize(tot=U.size());
    for(int i=0;i<tot;i++) stu[i]=0;
    dis=ask(stu);
    for(int i=0;i<tot;i++)
    {
        e[i]=(Edge){U[i],V[i]};
        g[U[i]].push_back(V[i]);
        g[V[i]].push_back(U[i]);
    }
    int rt=findroot();
    int rtx=e[rt].u,rty=e[rt].v;
    bfs(rtx,depx,bfnx,idxx);
    bfs(rty,depy,bfny,idxy);
    int s=gao(depx,depy,bfnx,idxx);
    int t=gao(depy,depx,bfny,idxy);
    answer(s,t);
}
```