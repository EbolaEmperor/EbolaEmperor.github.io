# 【PKUWC2018】随机算法

设f\[x\]\[s\]表示当前选了x个点，s是已选集合，用二进制表示

对于一个点y，如果它与x没有连边，则可以在剩下的第一个空位放y，而选了y之后新增的不能选集合可以任意排列在序列后面的所有空位中，方案数A(n-cnt\[s\]-1, cnt\[新增集合\])。其中cnt表示集合元素数量（即二进制中1的数量）

不管之前怎么放，对后面都是没有影响的，因为之前怎么排列，并不会改变剩余空位数量，所以方案用乘法原理合并。而每种选择状态只能由一种状态转移过来，所以转移过去之后用加法原理合并

枚举顺序先是选了几个点，再是状态，理由还是避免后效性

然后找到最多能选几个点，设它为m，则方案数就是f\[m\]\[全集\]。一共有n!种排列，所以乘上n的阶乘逆元即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=998244353;
const int N=21;
int cant[N],n,m;
int f[N][1<<N];
int cnt[1<<N];
int fac[N],ifac[N];

int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}

void Init(int lim)
{
    fac[0]=1;
    for(int i=1;i<=lim;i++)
        fac[i]=1ll*fac[i-1]*i%ha;
    ifac[lim]=Pow(fac[lim],ha-2);
    for(int i=lim-1;i>=0;i--)
        ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
}

int A(int n,int m){return 1ll*fac[n]*ifac[n-m]%ha;}

int main()
{
    scanf("%d%d",&n,&m);Init(n);
    for(int i=1;i<1<<n;i++)
        cnt[i]=cnt[i^(i&-i)]+1;
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        cant[u]|=1<<v-1;
        cant[v]|=1<<u-1;
    }
    f[0][0]=1;
    for(int i=0;i<=n;i++)
        for(int s=0;s<1<<n;s++)
        {
            if(!f[i][s]) continue;
            for(int j=1;j<=n;j++)
            {
                if(s&(1<<j-1)) continue;
                (f[i+1][s|cant[j]|(1<<j-1)]+=1ll*f[i][s]*A(n-cnt[s]-1,cnt[cant[j]^(cant[j]&s)])%ha)%=ha;
            }
        }
    for(int i=n;i>=0;i--)
    {
        if(!f[i][(1<<n)-1]) continue;
        int ans=1ll*f[i][(1<<n)-1]*ifac[n]%ha;
        printf("%d\n",ans);break;
    }
    return 0;
}
```

