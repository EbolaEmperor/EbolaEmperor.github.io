# 【AGC019F】Yes or No

题目链接：[AGC019 F. Yes or No  -  AtCoder](https://agc019.contest.atcoder.jp/tasks/agc019_f)

如果当前yes的数量多于no，那你肯定会去猜yes

然后就画一个坐标系，横坐标表示未猜yes的数量，纵坐标表示未猜no的数量。你从(n,m)出发，若你现在位于直线y=x的下方，那你肯定会去往左走；若你现在位于直线y=x的上方，那你肯定会去往下走

max(n,m)你是肯定能猜到的，问题就在于你到直线y=x时能蒙对几个。那位于y=x时再走一步，蒙对的概率是1/2，所以就要统计经过直线y=x的路径数，那显然就是![](http://latex.codecogs.com/svg.latex?\sum\limits_{i=1}^{\min(n,m)}\binom{2i}{i}\times\binom{n-i+m-i}{n-i})

算出之后，除以总的路径数得到概率，再乘上1/2算出期望就行了，然后再把你稳了的那max(n,m)加上，你就win了

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=998244353;
const int N=1000000;
int fac[N+5],ifac[N+5];

int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}

void Init()
{
    fac[0]=1;
    for(int i=1;i<=N;i++)
        fac[i]=1ll*fac[i-1]*i%ha;
    ifac[N]=Pow(fac[N],ha-2);
    for(int i=N-1;i>=0;i--)
        ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
}

inline int C(const int &n,const int &m){return 1ll*fac[n]*ifac[m]%ha*ifac[n-m]%ha;}
inline int invC(const int &n,const int &m){return 1ll*ifac[n]*fac[m]%ha*fac[n-m]%ha;}

int main()
{
    int n,m;Init();
    cin>>n>>m;
    int t=min(n,m),ans=0;
    for(int i=1;i<=t;i++)
        ans=(ans+1ll*C(2*i,i)*C(n+m-2*i,n-i))%ha;
    ans=1ll*ans*invC(n+m,n)%ha*ifac[2]%ha;
    printf("%d\n",(ans+max(n,m))%ha);
    return 0;
}
```

