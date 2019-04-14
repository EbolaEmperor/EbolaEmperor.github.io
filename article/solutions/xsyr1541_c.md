# C. 多线程

### 题意简述

给定序列A，你需要将它划分成一个上升子序列和一个下降子序列，求方案数

序列长度不超过5e5，每个测试点有10组数据

### 题解

考虑暴力dp。设![](http://latex.codecogs.com/svg.latex?f_{i,j})表示第i个**位置**属于上升序列，且下降序列的最后一个**值**为j时的方案数；![](http://latex.codecogs.com/svg.latex?g_{i,j})表示第i个**位置**属于下降序列，且上升序列的最后一个**值**为j时的方案数；

于是，有转移方程：

![](http://www.ebola.pro/images/xsyr1541_c_1.png)

发现可以直接用线段树优化转移。求两次区间和，然后整体清零一下，再做两次单点加法即可

但这题正解是线性的，所以线段树必须写得非常松才能卡过去。反正我是没卡过，于是考虑树状数组

树状数组无法支持区间覆盖。但我们发现，我们每次单点加法会修改log个点，然后每次转移会做两次单点加法，总共修改的点数是n log n级别的。所以可以直接把修改过的位置存下来，然后整体清零时，遍历修改过的位置将它清零即可

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
    int x=0,fg=1;char c=Get();
    while(!isdigit(c)&&c!='-') c=Get();
    if(c=='-') fg=-1,c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x*fg;
}

const int N=500010;
int ToT,n,m,a[N],h[N];

struct BIT
{
    int b[N];vector<int> used;
    inline int lowbit(const int &x){return x&-x;}
    void reset(){memset(b,0,sizeof(b));used.clear();}
    void add(int k,int x){for(k++;k<=m+1;k+=lowbit(k)) b[k]+=x,used.push_back(k);}
    int presum(int x){int res=0;for(x++;x;x-=lowbit(x)) res+=b[x];return res;}
    int sufsum(int x){return presum(m)-presum(x-1);}
    void clear(){for(int x : used) b[x]=0;used.clear();}
} f,g;

void gao(int x,int y)
{
    int s1=f.sufsum(x);
    int s2=g.presum(x);
    (x>y)?g.clear():f.clear();
    g.add(y,s1);f.add(y,s2);
}

int main()
{
    for(ToT=read();ToT;ToT--)
    {
        n=read();f.reset();g.reset();
        if(n==0){puts("1");continue;}
        for(int i=1;i<=n;i++) h[i]=a[i]=read();
        sort(h+1,h+1+n);
        m=unique(h+1,h+1+n)-h;
        f.add(m,1);g.add(0,1);
        for(int i=1;i<=n;i++) a[i]=lower_bound(h+1,h+m,a[i])-h;
        for(int i=2;i<=n;i++) gao(a[i],a[i-1]);
        printf("%d\n",f.presum(m)+g.presum(m));
    }
    return 0;
}
```