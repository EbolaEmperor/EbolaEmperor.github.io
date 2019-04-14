# 【AGC017E】Jigsaw

题目链接：[AGC017 E. Jigsaw  -  AtCoder](https://agc017.contest.atcoder.jp/tasks/agc017_e)

题面不太简洁，不是很好解释，翻译怕出锅，还是自己看原文吧

我们可以发现，拼图的过程类似插头匹配什么的，所以其实拼图的左右都可以看作接口。对于左接口，若C为零，则接口值为A，否则接口值为-C；对于右接口，若D为零，则接口值为-B，否则接口值为D

这样建模之后，我们发现，两个接口能连接，当且仅当接口值相等

这样我们就可以将不同的接口值视作点，拼图视作有向边 ，连接左右接口对应的点

现在我们的任务是，将图上的所有边划分成若干条互不相交的路径，要求路径起点为正权，终点为负权，这对应了一连串的拼图左边接地、右边接地、中间互接的这种情况

那可以直接根据点的度数来判断。对于一个正权点，出度必须要大于等于入度；对于一个负权点，入读必须要大于等于出度。此外，为了防止 “我 插 我 自 己” 这种情况的发生，需要特判环，即对于一个弱联通分量，至少要存在一个出入度不相等的点，直接用并查集搞一搞即可

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

const int N=410;
int n,h,in[N],out[N];
int fa[N];
bool ok[N];

int find(const int &x){return fa[x]==x?x:fa[x]=find(fa[x]);}

int main()
{
    n=read();h=read();
    for(int i=1;i<=(h<<1);i++) fa[i]=i;
    for(int i=1;i<=n;i++)
    {
        int A=read(),B=read(),C=read(),D=read();
        int x=C?C+h:A,y=D?D:B+h;
        out[x]++;in[y]++;
        x=find(x);y=find(y);
        if(x!=y) fa[x]=y;
    }
    for(int i=1;i<=h;i++) if(in[i]>out[i]) return !puts("NO");
    for(int i=h+1;i<=(h<<1);i++) if(in[i]<out[i]) return !puts("NO");
    for(int i=1;i<=(h<<1);i++) if(in[i]!=out[i]||!in[i]&&!out[i]) ok[find(i)]=1;
    for(int i=1;i<=(h<<1);i++) if(fa[i]==i&&!ok[i]) return !puts("NO");
    return !puts("YES");
}
```

