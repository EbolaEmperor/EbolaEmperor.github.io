# 【ARC073F】Many Moves

题目链接：[ACR073 F. Many Moves](https://www.luogu.org/jump/atcoder/2558)

考虑dp。设f\[i\]表示一个棋子位于i，另一个棋子位于i-1时的最小代价，然后不难列出n²的dp转移方程：

![](http://latex.codecogs.com/svg.latex?f_{i}=\min\limits_{j=1}^{i-1}\left(f_j+\sum\limits_{k=j+1}^{i-1}\Delta_k+|x_i-x_{j-1}|\right))

其中![](http://latex.codecogs.com/svg.latex?\Delta_k=|x_k-x_{k-1}|)

于是我们可以求出$\Delta_k$的前缀和$sum$，然后式子可以化为：

![](http://latex.codecogs.com/svg.latex?f_{i}=\min\limits_{j=1}^{i-1}\left(f_j-sum_j+|x_i-x_{j-1}|\right)+sum_{i-1})

区间最小值这种东西，用线段树搞搞就好了。但有个绝对值，所以我们要维护两颗线段树，一棵存![](http://latex.codecogs.com/svg.latex?f_j-sum_j-x_{j-1})，另一棵存![](http://latex.codecogs.com/svg.latex?f_j-sum_j+x_{j-1})，然后以x的值为下标，这样对于每个i，我们只要在线段树一的\[1, xi\]中查找最小值，找到最小值加上xi+sum\[i-1\]，在线段树二中的查找也是类似的

然后初值有点麻烦，我的做法是设x0=a，然后f\[1\]=|b-x1|，再按上面说的去做。但这样涉及到a和b选哪个的问题，所以要跑完一遍后交换a和b再跑一遍取最小值才行

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

typedef long long LL;
const int N=200010;
int n,q,a,b,X[N];
int delta[N];
LL sumd[N],f[N];

struct SegmentTree
{
    LL mnv[N<<2];int leaf;
    void build()
    {
        for(leaf=1;leaf<=(n+2);leaf<<=1);
        for(int i=1;i<=leaf+n;i++) mnv[i]=INT64_MAX/3;
    }
    void modify(int k,LL x)
    {
        mnv[leaf+k]=min(mnv[leaf+k],x);
        for(int i=(leaf+k)>>1;i;i>>=1)
            mnv[i]=min(mnv[i<<1],mnv[i<<1|1]);
    }
    LL qmin(int l,int r)
    {
        LL res=INT64_MAX/3;
        for(l=leaf+l-1,r=leaf+r+1;l^r^1;l>>=1,r>>=1)
        {
            if(~l&1) res=min(res,mnv[l^1]);
            if(r&1) res=min(res,mnv[r^1]);
        }
        return res;
    }
}t1,t2;

LL solve(int a,int b)
{
    X[0]=a;
    for(int i=1;i<=q;i++)
    {
        delta[i]=abs(X[i]-X[i-1]);
        sumd[i]=sumd[i-1]+delta[i];
    }
    f[1]=abs(b-X[1]);
    t1.build();t2.build();
    t1.modify(X[0],f[1]-sumd[1]-X[0]);
    t2.modify(X[0],f[1]-sumd[1]+X[0]);
    for(int i=2;i<=q;i++)
    {
        f[i]=t1.qmin(1,X[i])+X[i]+sumd[i-1];
        f[i]=min(f[i],t2.qmin(X[i],n)-X[i]+sumd[i-1]);
        t1.modify(X[i-1],f[i]-sumd[i]-X[i-1]);
        t2.modify(X[i-1],f[i]-sumd[i]+X[i-1]);
    }
    LL ans=INT64_MAX/3;
    for(int i=0;i<=q;i++) ans=min(ans,f[i]+sumd[q]-sumd[i]);
    return ans;
}

int main()
{
    n=read();q=read();a=read();b=read();
    for(int i=1;i<=q;i++) X[i]=read();
    LL ans=min(solve(a,b),solve(b,a));
    printf("%lld\n",ans);
    return 0;
}
```

