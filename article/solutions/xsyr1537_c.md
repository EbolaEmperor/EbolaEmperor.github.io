# C. 青春野狼不做恶魔小学妹的梦

### 题意简述

设$G$是$n$个点的连通图，求$\sum\limits_{G=(V,E)}|E|^k$，答案对$998244353$取模

数据范围：$n \leq 50000,\quad k\leq 15$

***

### 题解

首先斯特林展开，得：$|E|^k=\sum\limits_{i=0}^k{k\brace i}\binom{|E|}{i}i!$

考虑换一个组合意义。原来我们是枚举边集$E$，然后从$E$中选$i$个边去计算贡献。不难发现，对于任意一组固定的$i$条边，它对答案的贡献次数就是包含它的边集$E$数量，每次贡献${k\brace i}i!$。于是我们可以对每组固定的$i$条边，去统计有多少边集$E$包含它

设$f_{i,j}$表示在$i$个点的**连通图**中钦点了任意$j$条边必选，有多少边集$E$包含了这$j$条边。$g_{i,j}$则表示任意图中的上述意义

那么，设$i$个点的图总边数为$e$，显然有$g_{i,j}=\binom{e}{j}2^{e-j}$

考虑标号为$1$的点在哪个联通块中，不难得到$g_{n,m}=\sum\limits_{i=0}^n\sum\limits_{j=0}^m\binom{n-1}{i-1}f_{i,j}g_{n-i,m-j}$

稍作变化，得到：$\frac{g_{n,m}}{(n-1)!}=\sum\limits_{i=0}^n\sum\limits_{j=0}^m\frac{f_{i,j}}{(i-1)!}\times\frac{g_{n-i,m-j}}{(n-i)!}$

设$x=i,y=n-i$，得到：$\frac{g_{n,m}}{(n-1)!}=\sum\limits_{j=0}^m\sum\limits_{x+y=n}\frac{f_{x,j}}{(x-1)!}\times\frac{g_{y,m-j}}{y!}$

这是一个卷积的形式，我们设$F_{j,k}=\frac{f_{k,j}}{(k-1)!},\quad G_{j,k}=\frac{g_{k,j}}{k!},\quad H_{j,k}=\frac{g_{k,j}}{(k-1)!}$

则根据原式，有：$H_m(x)=\sum\limits_{j=0}^mF_j(x)*G_{m-j}(x)$

稍作变化，得到：$F_m(x)=\frac{H_m(x)-\sum\limits_{j=0}^{m-1}F_j(x)*G_{m-j}(x)}{G_0(x)}$

NTT+多项式求逆即可

最后的答案就是$\sum\limits_{i=0}^kF_{i,n}(n-1)!{k\brace i}i!$

***

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int ha=998244353;
const int N=200010;
int r[N],tinv[N];
int omg[N],iomg[N],inv[N];
int strl[20][20],fac[N],ifac[N];
int f[20][N],g[20][N],h[20][N],ig0[N],sum[N];
int n,k;

inline int add(const int &x,const int &y){return (x+y>=ha)?(x+y-ha):(x+y);}
inline int dev(const int &x,const int &y){return (x-y<0)?(x-y+ha):(x-y);}

int Pow(int a,LL b)
{
    int ans=1;
    if(b<0) b+=ha-1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}

void math_init(int n=200000)
{
    fac[0]=1;
    for(int i=1;i<=n;i++)
        fac[i]=1ll*fac[i-1]*i%ha;
    ifac[n]=Pow(fac[n],ha-2);
    for(int i=n-1;i>=0;i--)
        ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
    strl[0][0]=1;
    for(int i=1;i<=15;i++)
        for(int j=1;j<=i;j++)
            strl[i][j]=(strl[i-1][j-1]+1ll*j*strl[i-1][j])%ha;
}

void NTT_init()
{
    for(int i=1;i<N;i<<=1)
    {
        int p=i<<1;
        omg[i]=Pow(3,(ha-1)/p);
        iomg[i]=Pow(omg[i],ha-2);
        inv[i]=Pow(i,ha-2);
    }
}

void NTT(int *A,int n,int d)
{
    for(int i=0;i<n;i++) if(i<r[i]) swap(A[i],A[r[i]]);
    for(int i=1;i<n;i<<=1)
    {
        int wn=(d>0)?omg[i]:iomg[i];
        for(int j=0;j<n;j+=i<<1)
            for(int k=0,w=1;k<i;k++)
            {
                int x=A[j+k],y=1ll*w*A[i+j+k]%ha;
                A[j+k]=add(x,y);
                A[i+j+k]=dev(x,y);
                w=1ll*w*wn%ha;
            }
    }
    if(d>0) return;
    for(int i=0;i<n;i++) A[i]=1ll*A[i]*inv[n]%ha;
}

void get_inv(int *a,int *b,int n)
{
    if(n==1){b[0]=Pow(a[0],ha-2);return;}
    get_inv(a,b,(n+1)>>1);
    int l=0,len=1;while(len<(n<<1)) len<<=1,l++;
    for(int i=0;i<len;i++) r[i]=(r[i>>1]>>1)|((i&1)<<l-1);
    for(int i=0;i<len;i++) tinv[i]=(i<n)?a[i]:0;
    NTT(tinv,len,1);NTT(b,len,1);
    for(int i=0;i<len;i++)
        b[i]=1ll*(2ll-1ll*tinv[i]*b[i]%ha+ha)*b[i]%ha;
    NTT(b,len,-1);
    for(int i=n;i<len;i++) b[i]=0;
}

int C(LL n,int m)
{
    int res=1;
    for(LL i=n;i>n-m;i--) res=i*res%ha;
    res=1ll*res*ifac[m]%ha;
    return res;
}

void falun_dafa_is_good()
{
    int len=1;while(len<=n) len<<=1;
    for(int i=0;i<=k;i++)
    {
        for(int j=0;j<len;j++)
        {
            LL e=1ll*j*(j-1)/2;
            int s=1ll*C(e,i)*Pow(2,e-i)%ha;
            g[i][j]=1ll*s*ifac[j]%ha;
            if(j) h[i][j]=1ll*s*ifac[j-1]%ha;
        }
        if(!i) get_inv(g[i],ig0,len);
        NTT(g[i],len<<1,1);
    }
    NTT(ig0,len<<1,1);
    int ans=0;
    for(int i=0;i<=k;i++)
    {
        memset(sum,0,sizeof(sum));
        for(int l=0;l<(len<<1);l++)
            for(int j=0;j<i;j++)
                sum[l]=(sum[l]+1ll*f[j][l]*g[i-j][l])%ha;
        NTT(sum,len<<1,-1);
        for(int j=0;j<len;j++) f[i][j]=(h[i][j]-sum[j]+ha)%ha;
        NTT(f[i],len<<1,1);
        for(int j=0;j<(len<<1);j++) f[i][j]=1ll*f[i][j]*ig0[j]%ha;
        NTT(f[i],len<<1,-1);
        for(int j=len;j<(len<<1);j++) f[i][j]=0;
        ans=(ans+1ll*fac[n-1]*f[i][n]%ha*strl[k][i]%ha*fac[i])%ha;
        NTT(f[i],len<<1,1);
    }
    printf("%d\n",ans);
}

int main()
{
    NTT_init();math_init();
    scanf("%d%d",&n,&k);
    falun_dafa_is_good();
    return 0;
}
```