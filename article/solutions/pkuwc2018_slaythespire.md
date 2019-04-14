# 【PKUWC2018】Slay The Spire

题目链接：[「PKUWC2018」Slay the Spire - LibreOJ](https://loj.ac/problem/2538)

不难发现，我们的最优策略是先把强化牌打完，然后从大到小出攻击牌。如果强化牌多于k张，那就用前k-1大的，然后再出最大的的攻击牌。证明非常显然，这里省略不提

下面是一些dp数组的定义：

![](http://www.ebola.pro/images/pkuwc2018_sts_1.png)

下面是dp数组之间一些显然的关系

![](http://www.ebola.pro/images/pkuwc2018_sts_2.png)

根据我们的结论，答案就应该是：

![](http://www.ebola.pro/images/pkuwc2018_sts_3.png)

现在考虑如何计算f与g。对于f，转移时由于每个方案都会乘上一个数，所以转移时f可以直接乘上一个数。对于g，转移时每个方案都会加上一个数，所以转移时g应该加上要加的数乘方案数。于是有转移方程：

![](http://www.ebola.pro/images/pkuwc2018_sts_4.png)

注意到后面那个求和是一个前缀和的形式，所以可以预处理前缀和，从而做到O(nm)的复杂度

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=998244353;
const int N=3010;
int C[N][N];
int mul[N],add[N];
int f[N][N],g[N][N];
int n,m,k;

int F(int x,int y)
{
    int res=0;
    if(y==0) return C[n][x];
    for(int i=1;i<=n;i++)
        res=(res+1ll*f[y][i]*C[n-i][x-y])%ha;
    return res;
}

int G(int x,int y)
{
    int res=0;
    for(int i=1;i<=n;i++)
        res=(res+1ll*g[y][i]*C[n-i][x-y])%ha;
    return res;
}

int gao()
{
    f[0][0]=1;
    static int sumf[N],sumg[N];
    for(int i=1;i<=m;i++)
    {
        sumf[0]=f[i-1][0];
        sumg[0]=g[i-1][0];
        for(int j=1;j<=n;j++)
        {
            sumf[j]=(sumf[j-1]+f[i-1][j])%ha;
            sumg[j]=(sumg[j-1]+g[i-1][j])%ha;
        }
        for(int j=1;j<i;j++) f[i][j]=g[i][j]=0;
        for(int j=i;j<=n;j++)
        {
            f[i][j]=1ll*mul[j]*sumf[j-1]%ha;
            g[i][j]=(1ll*add[j]*C[j-1][i-1]+sumg[j-1])%ha;
        }
    }
    int ans=0;
    for(int i=0;i<k;i++)
        ans=(ans+1ll*F(i,i)*G(m-i,k-i))%ha;
    for(int i=k;i<=m;i++)
        ans=(ans+1ll*F(i,k-1)*G(m-i,1))%ha;
    return ans;
}

int main()
{
    for(int i=0;i<=3000;i++)
    {
        C[i][0]=1;
        for(int j=1;j<=i;j++)
            C[i][j]=(C[i-1][j-1]+C[i-1][j])%ha;
    }
    int T;
    for(scanf("%d",&T);T;T--)
    {
        scanf("%d%d%d",&n,&m,&k);
        for(int i=1;i<=n;i++) scanf("%d",mul+i);
        for(int i=1;i<=n;i++) scanf("%d",add+i);
        sort(mul+1,mul+1+n);reverse(mul+1,mul+1+n);
        sort(add+1,add+1+n);reverse(add+1,add+1+n);
        printf("%d\n",gao());
    }
    return 0;
}
```