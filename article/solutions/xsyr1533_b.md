# B. 粒子

### 题面

![](http://www.ebola.pro/images/xsyr1533_b_1.png)

n≤1e5，k≤100

### 题解

其实挺简单的一道题，比赛时候脑子卡屎了qwq

先考虑求第一个碰撞。显然可以二分时间，求出最早的可能碰撞时间是什么，显然给定时间内能走的总路程超过L的一对粒子x、y会碰撞。找到求得时间内能走的路程最长的x粒子和y粒子，答案就是它们了

那剩下的K个碰撞一样解决。相当于把上一次求得的粒子删除，然后重复一遍同样的问题，因为k很小，所以能过。注意卡常，实测二分次数控制在30次可以通过

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=50010;
struct INFO{int v,t,id;} px[N],py[N];
bool donex[N],doney[N];
int n,L,k;

void getans(double t)
{
    int ansx=0,ansy=0;
    double maxx=0,maxy=0;
    for(int i=1;i<=n;i++)
    {
        if(!donex[i]&&px[i].t<=t)
            if(px[i].v*(t-px[i].t)>maxx)
                maxx=px[i].v*(t-px[i].t),ansx=i;
        if(!doney[i]&&py[i].t<=t)
            if(py[i].v*(t-py[i].t)>maxy)
                maxy=py[i].v*(t-py[i].t),ansy=i;
    }
    donex[ansx]=doney[ansy]=1;
    printf("%d %d\n",px[ansx].id,py[ansy].id);
}

bool check(double t)
{
    double maxx=0,maxy=0;
    for(int i=1;i<=n;i++)
    {
        if(!donex[i]&&px[i].t<=t) maxx=max(maxx,px[i].v*(t-px[i].t));
        if(!doney[i]&&py[i].t<=t) maxy=max(maxy,py[i].v*(t-py[i].t));
        if(maxx+maxy>=L) return 1;
    }
    return 0;
}

double gao()
{
    double l=0,r=0,mid;
    for(int i=1;i<=n;i++)
    {
        if(!donex[i]) r=max(r,px[i].t+1.0*L/px[i].v);
        if(!doney[i]) r=max(r,py[i].t+1.0*L/py[i].v);
    }
    for(int tms=0;tms<30;tms++)
    {
        mid=(l+r)/2;
        if(check(mid)) r=mid;
        else l=mid;
    }
    return r;
}

int main()
{
    scanf("%d%d%d",&n,&L,&k);
    for(int i=1;i<=n;i++) scanf("%d%d",&px[i].t,&px[i].v),px[i].id=i;
    for(int i=1;i<=n;i++) scanf("%d%d",&py[i].t,&py[i].v),py[i].id=i;
    sort(px+1,px+1+n,[](INFO a,INFO b){return a.t+1.0*L/a.v<b.t+1.0*L/b.v;});
    sort(py+1,py+1+n,[](INFO a,INFO b){return a.t+1.0*L/a.v<b.t+1.0*L/b.v;});
    for(int i=1;i<=k;i++) getans(gao());
    return 0;
}
```

