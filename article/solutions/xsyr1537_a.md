# A. 青春野狼不做理性小魔女的梦

### 题意简述

你记得你有一个长度为k的序列，每个位置都填了一个$[0,m)$中的整数

可是你现在不记得$m$了，也不记得这个序列了，不过你手中有一个序列$A$，有一些位置没有值，你知道若序列$A$的第$i$个位置有值，则原序列的第$i$个数是$A_i \mod m$

你还记得方程$\sum\limits_{i=1}^kA_ix_i\equiv1(mod\;m)$有解

求$m$在$[1,n]$之间时，$m$与原序列的可能方案数，模$10^9+7$

数据范围：$k\leq 50,\quad n\leq 10^9$

***

### 题解

首先根据裴蜀定理，方程有解的充要条件是$\gcd(A_1,A_2,...,A_k,m)=1$

设$g=\gcd(A_1,A_2,...,A_k)$，$c$表示序列$A$中没有值的位置数量，考虑容斥，则答案就是：$\sum\limits_{m=1}^n\sum\limits_{d|g,\;d|m}\mu(d)\left(\frac{m}{d}\right)^c$

交换枚举顺序，得到：$\sum\limits_{d=1}^n[d|g]\mu(d)\sum\limits_{d|m,\;m\leq n}\left(\frac{m}{d}\right)^c$

稍作变化，得到：$\sum\limits_{d=1}^n[d|g]\mu(d)\sum\limits_{i=1}^{\left\lfloor
\frac{n}{d}\right\rfloor}i^c$

若序列$A$所有位置要么没有值，要么为$0$，则答案就是$\sum\limits_{d=1}^n\mu(d)\sum\limits_{i=1}^{\left\lfloor
\frac{n}{d}\right\rfloor}i^c$，直接就杜教筛+自然数幂求和

否则，可以枚举$g$中$\mu$值非零的约数$d$，然后计算出$\mu(d)$，再自然数幂求和即可

考虑到求和的范围比较大，$c$的范围比较小，我们使用拉格朗日插值法来实现自然数幂求和

***

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=1e9+7;
const int N=55;
int k,n,c,g;

int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}

namespace MuSum
{
    const int M=1000010;
    int mu[M],prm[M],mark[M],tot=0;
    unordered_map<int,int> smu;

    void init(int n=1000000)
    {
        mu[1]=1;
        for(int i=2;i<=n;i++)
        {
            if(!mark[i]) prm[++tot]=i,mu[i]=-1;
            for(int j=1;j<=tot&&i*prm[j]<=n;j++)
            {
                mark[i*prm[j]]=1;
                if(i%prm[j]) mu[i*prm[j]]=-mu[i];
                else{mu[i*prm[j]]=0;break;}
            }
        }
        for(int i=2;i<=n;i++) mu[i]+=mu[i-1];
    }

    int gao(int x)
    {
        if(x<=1000000) return mu[x];
        if(smu.count(x)) return smu[x];
        int d,res=1;
        for(int i=2;i<=x;i=d+1)
            d=x/(x/i),res=(res-1ll*(d-i+1)*gao(x/i)%ha+ha)%ha;
        return smu[x]=res;
    }
}

namespace PowSum
{
    int y[N],pre[N],suf[N],ifac[N];

    void init()
    {
        y[1]=1;
        for(int i=2;i<=c+2;i++)
            y[i]=(y[i-1]+Pow(i,c))%ha;
        int facn=1;
        for(int i=1;i<=c+2;i++)
            facn=1ll*facn*i%ha;
        ifac[c+2]=Pow(facn,ha-2);
        for(int i=c+1;i>=0;i--)
            ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
        pre[0]=suf[c+3]=1;
    }

    int gao(int x)
    {
        if(x<=c+2) return y[x];
        for(int i=1;i<=c+2;i++) pre[i]=1ll*pre[i-1]*(x-i)%ha;
        for(int i=c+2;i>=1;i--) suf[i]=1ll*suf[i+1]*(x-i)%ha;
        int res=0;
        for(int i=1;i<=c+2;i++)
        {
            int t=1ll*y[i]*pre[i-1]%ha*suf[i+1]%ha;
            t=1ll*t*ifac[i-1]%ha*ifac[c+2-i]%ha;
            if((c+2-i)&1) t=ha-t;
            res=(res+t)%ha;
        }
        return res;
    }
}

int gao1()
{
    int ans=0,d;
    for(int i=1;i<=n;i=d+1)
    {
        d=n/(n/i);
        int u=(MuSum::gao(d)-MuSum::gao(i-1)+ha)%ha;
        ans=(ans+1ll*u*PowSum::gao(n/i))%ha;
    }
    return ans;
}

int gao2()
{
    static int p[N],tot=0;
    for(int i=2;i*i<=g;i++)
    {
        if(g%i) continue;
        p[++tot]=i;
        while(g%i==0) g/=i;
    }
    if(g>1) p[++tot]=g;
    int ans=0;
    for(int s=0;s<(1<<tot);s++)
    {
        int fg=1,d=1;
        for(int i=1;i<=tot;i++) if(s&(1<<i-1)) fg*=-1,d*=p[i];
        ans=(ans+fg*PowSum::gao(n/d))%ha;
    }
    return (ans+ha)%ha;
}

int main()
{
    scanf("%d%d",&k,&n);
    int cnt=0;
    for(int i=1,x;i<=k;i++)
    {
        scanf("%d",&x);
        if(x==-1) c++;
        if(x>0) g=g?__gcd(g,x):x;
        if(x) cnt++;
    }
    PowSum::init();MuSum::init();
    printf("%d\n",(c==cnt)?gao1():gao2());
    return 0;
}
```