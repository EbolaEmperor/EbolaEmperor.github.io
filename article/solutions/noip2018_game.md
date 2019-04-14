# 【NOIP2018】填数游戏

Latex繁多，请移步：https://ebola-emperor.blog.luogu.org/solution-p5023

先摆两个结论，原因自行思考

1. $V_{i+1,j}\geq V_{i,j+1}$
2. 若 $V_{i+1,j}=V_{i,j+1}$，则矩形区域$(i+1,j+1)$到$(n,m)$的每条左下-右上对角线中的数字都要相等

那么我们可以设$f_{i,k,s}$表示从右下往左上数的第$i$条对角线，放了$k$个$1$，相等状态为$s$时的方案数。显然这$k$个$1$必然是放在从左下开始的连续$k$个位置，因此$j$可以唯一确定这条对角线的状态。相等状态是指：若以对角线的第$t$个位置为左上角、以$(n,m)$为右下角的矩形，满足左下-右上对角线中数字相等，则$s$的二进制第$t$位为$1$

于是我们可以枚举$i,k,s$，然后考虑将$f_{i,k,s}$往下一条对角线转移。那么我们枚举下一条对角线的状态$x$（即它包含$x$个$1$），然后我们检查枚举到的这个状态是否合法。如果合法，那么对于下一条对角线的第$p$个位置（从左下到右上编号），除非$p$位置为$1$且$p+1$位置为$0$（即$p=x$），否则以$p$位置右边那个格子为左上角、以$(n,m)$为右下角的矩形，必然要满足左下-右上对角线中数字相等，利用对角线$i$的$s$状态判定即可。
检查了合法性，还要知道我们从$f_{i,k,s}$转移到哪里去。$x$已经确定了，所以我们要得到下一条对角线的相等状态，利用$s$状态稍作推导即可，具体见代码

这样得到的复杂度是$O(m2^nn^3)$，把无效状态break掉，拿80分应该没有问题

但是我们可以找规律啊。打表，发现，当$n\geq 2$且$m>n+1$时，有$ans_{n,m}=3\times ans_{n,m-1}$。所以我们只要求得$ans_{n,n+1}$，然后等比数列求一下就行了，复杂度$O(n^42^n)$

注意我们规律的前提条件是$n\geq 2$，因此要注意特判$n=1$的点。$n=1$时答案显然为$2^m$

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int ha=1e9+7;
int f[2][10][260],n,m,nm;
int len[1000020],tot;
 
void add(int &x,const int &y){x=x+y>=ha?x+y-ha:x+y;}
int dwn(int t,int x){return t>tot-n+1?x:x-1;}
int rght(int t,int x){return t>tot-n+1?x+1:x;}
bool in(int x,int s){return s&(1<<x-1);}
 
int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}
 
int main()
{
    cin>>n>>m;
    if(n>m) swap(n,m);
    if(n==1)
    {
        printf("%d\n",Pow(2,m));
        return 0;
    }
    if(m>n+1) nm=m,m=n+1;
    tot=n+m-1;
    for(int i=1;i<=n;i++) len[i]=i;
    for(int i=n+1;i<=m;i++) len[i]=n;
    for(int i=m+1;i<=tot;i++) len[i]=tot-i+1;
    f[1][0][1]=f[1][1][1]=1;
    for(int i=2;i<=tot;i++)
    {
        int k=i&1;
        memset(f[k],0,sizeof(f[k]));
        for(int s=0;s<(1<<len[i-1]);s++)
            for(int j=0;j<=len[i-1];j++)
            {
                if(!f[k^1][j][s]) continue;
                for(int y=0;y<=len[i];y++)
                {
                    bool flag=1;
                    for(int x=1;x<len[i];x++)
                        if(x!=y&&!in(rght(i,x),s)){flag=0;break;}
                    if(!flag) continue;
                    int t=0;
                    for(int k=1;k<=len[i];k++)
                    {
                        int d=dwn(i,k),r=rght(i,k);
                        if(d<1&&in(r,s)) t|=1<<k-1;
                        else if(r>len[i-1]&&in(d,s)) t|=1<<k-1;
                        else if(d!=j&&in(d,s)&&in(r,s)) t|=1<<k-1;
                    }
                    add(f[k][y][t],f[k^1][j][s]);
                }
            }
    }
    int ans=0;
    for(int i=0;i<2;i++)
        for(int j=0;j<2;j++)
            add(ans,f[tot&1][i][j]);
    if(nm) ans=1ll*ans*Pow(3,nm-m)%ha;
    printf("%d\n",ans);
    return 0;
}
```