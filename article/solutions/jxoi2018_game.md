# 【JXOI2018】游戏

题目链接：[2544. 游戏  -  LibreOJ](https://loj.ac/problem/2544)

### 吐槽

时隔半年，我终于回来做这道题了，然后对半年前的我产生了深深的怀疑

半年前，我在场上，居然只打了20分暴力，甚至连l=1的部分分都没有推出来，然而现在看来l=1的部分分不是普及组难度吗qwq

题目本身也不太难，作为省选题，吉老师真的算是非常友好了

算是见证了自己水平的提升吧，希望下一个赛季能比半年前高一个大台阶

### 题解

首先计算出l到r中，不是任何数的倍数的有多少个，记这个值为sum。比如说l=4, r=8时，4,5,6都不是任何数的倍数，此时sum=3。我们称这样的数为“关键数”

不难发现，t(p)的定义，其实就是排列p中，最后一个关键数的位置

于是我们可以枚举最后一个关键数的位置i，在它后面不能存在关键数，所以它后面的元素有C(n-sum, n-i)种选取方法，对于每种选取方案，i后面的元素有(n-i)!种排列方式，i前面的元素有(i-1)!种排列方式，然后i这个位置可以是sum个关键数中的任意一个，并且这个位置对答案的贡献为i

于是答案可以表达成下式：

![](http://latex.codecogs.com/svg.latex?\sum\limits_{i=1}^ni\times&space;sum\times&space;\binom{n-sum}{n-i}\times(n-i)!\times(i-1)!)

直接预处理阶乘及其逆元即可，sum直接埃氏筛暴算即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=1000000007;
const int N=10000010;
bool vis[N];
int l,r,sum=0;
int fac[N],ifac[N];

int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}

void Init(int n)
{
    fac[0]=1;
    for(int i=1;i<=n;i++)
        fac[i]=1ll*fac[i-1]*i%ha;
    ifac[n]=Pow(fac[n],ha-2);
    for(int i=n-1;i>=0;i--)
        ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
}

inline int C(int n,int m){return n<m?0:1ll*fac[n]*ifac[m]%ha*ifac[n-m]%ha;}

int main()
{
    cin>>l>>r;Init(r);
    for(int i=l;i<=r;i++)
    {
        if(vis[i]) continue;
        sum++;
        for(int j=i;j<=r;j+=i)
            vis[j]=1;
    }
    int ans=0,n=r-l+1;
    for(int i=1;i<=n;i++)
    {
        int res=1ll*i*sum%ha;
        res=1ll*res*C(n-sum,n-i)%ha;
        res=1ll*res*fac[n-i]%ha;
        res=1ll*res*fac[i-1]%ha;
        ans=(ans+res)%ha;
    }
    cout<<ans<<endl;
    return 0;
}
```

