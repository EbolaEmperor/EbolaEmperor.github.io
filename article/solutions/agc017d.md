# 【AGC017D】Game on Tree

题目链接：[AGC017 D. Game on Tree  -  AtCoder](https://agc017.contest.atcoder.jp/tasks/agc017_d)

讲真这种在国内OI界早已被奉为经典的题，拿出来比赛，不是纯粹欺负外国人吗……

经典结论：树上节点的sg值等于所有每个子树sg值加1的异或和

```cpp
#include<bits/stdc++.h>
using namespace std;

const int S=(1<<20)+5;
char buf[S],*H,*T;
inline char Get()
{
    if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
    if(H==T) return -1;return *H++;
}
inline int read()
{
    int x=0;char c=Get();
    while(!isdigit(c)) c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x;
}

const int N=100010;
struct Edge{int to,next;} e[N<<1];
int h[N],sum=0,n;

void add_edge(int u,int v)
{
    e[++sum].to=v;
    e[sum].next=h[u];
    h[u]=sum;
}

int dfs(int u,int fa)
{
    int res=0;
    for(int tmp=h[u];tmp;tmp=e[tmp].next)
        if(e[tmp].to!=fa) res^=dfs(e[tmp].to,u)+1;
    return res;
}

int main()
{
    int u,v;n=read();
    for(int i=1;i<n;i++)
    {
        u=read();v=read();
        add_edge(u,v);
        add_edge(v,u);
    }
    puts(dfs(1,0)?"Alice":"Bob");
    return 0;
}
```

