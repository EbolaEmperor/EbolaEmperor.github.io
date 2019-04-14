# 【AGC019C】Fountain Walk

题目链接：[AGC019 C. Fountain Walk  -  AtCoder](https://agc019.contest.atcoder.jp/tasks/agc019_c)

先通过镜像翻转使得目标点在出发点的右上方，这样能简化问题

每行每列都只会有一个转盘，这是一个重要条件

想象一下我们的路径，就是不断往右上方走，遇到转盘就绕1/4周转弯，可以把原来转弯的20优化成5π，我们希望在路径上遇到尽可能多的转盘。

于是这就可以建立LIS模型。取出(x1,y1)到(x2,y2)区域内的转盘，以横坐标为下标，纵坐标为值，跑一遍LIS，得到的答案就是我们路径上能经过的最多圆盘数量

但这样会有一些小问题。比如说当每一行都经过一个转盘时，最顶上那一行的转盘必然要绕1/2周。因此需要加一个特判，若每一行都经过一个圆盘，或每一列都经过一个圆盘，则答案加上1/4圆周的长度

```cpp
#include<bits/stdc++.h>
#define X first
#define Y second
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

typedef pair<int,int> pii;
const double pi=acos(-1);
const int N=200010;
const int MX=1e8+5;
int x1,yy1,x2,yy2,n,tot=0,lis;
int g[N],idx[N],a[N],f[N],gx[N],ax[N];
pii pnt[N];
bool deled[N];

int LIS()
{
    int res=0;
    memset(g,0x7f,sizeof(g));
    for(int i=1;i<=tot;i++)
    {
        auto it=lower_bound(g+1,g+1+n,a[i]);
        if(*it==0x7f7f7f7f) res++;
        *it=a[i];
    }
    return res;
}

int main()
{
    x1=read();yy1=read();x2=read();yy2=read();n=read();
    for(int i=1;i<=n;i++) pnt[i].X=read(),pnt[i].Y=read();
    if(x2<x1||yy2<yy1)
    {
        if(x1<=x2) goto swap2;
        x1=MX-x1;x2=MX-x2;
        for(int i=1;i<=n;i++) pnt[i].X=MX-pnt[i].X;
        if(yy1<=yy2) goto work;
        swap2: yy1=MX-yy1;yy2=MX-yy2;
        for(int i=1;i<=n;i++) pnt[i].Y=MX-pnt[i].Y;
    }
    work: sort(pnt+1,pnt+1+n);
    for(int i=1;i<=n;i++)
        if(pnt[i].X>=x1&&pnt[i].X<=x2&&pnt[i].Y>=yy1&&pnt[i].Y<=yy2) a[++tot]=pnt[i].Y;
    double ans=100.0*(x2-x1+yy2-yy1)+(pi*5-20)*(lis=LIS());
    if(lis==min(x2-x1,yy2-yy1)+1) ans+=pi*5;
    output: printf("%.12lf\n",ans);
    return 0;
}
```

