# 【JXOI2018】排序问题

题目链接：[2543. 排序问题  -  LibreOJ](https://loj.ac/problem/2543)

### 吐槽

样例解释真的给了我很大的提示。我的期望学的不好，结果样例解释直接就告诉了我轮数期望就是一次成功概率的倒数，这一点吉老师真的很良心了

然后本题的两个关键结论，我在考场上真的全都想到了，但当时代码能力实在是差，套路又学的不够多，导致想到了结论都不知道怎么做

### 题解

从简单的开始想起。若数列中有cnt个最小值，那么这些最小值可以放在1-cnt之间的任意位置，于是它们有cnt!种可行排列方式。再用一样的方式去考虑其它的值。那根据乘法原理，一次成功的概率显然就是![](http://latex.codecogs.com/svg.latex?\frac{\prod&space;cnt_x&space;!}{n!})

然后感谢吉老师的样例解释，让我知道了期望就是它的倒数：![](http://latex.codecogs.com/svg.latex?\frac{n!}{\prod&space;cnt_x&space;!})

于是现在我们的任务就是让cnt\[x\]!的乘积尽可能小，这样期望才会尽可能大

于是不难想到贪心。每次选cnt最小的数x，把它加入数列，并更新答案

选最小数？难不成用堆？不行！插入的次数达到了1e7的级别，再加之有1e5组询问，我们的复杂度甚至不能和m扯上半毛钱关系，否则就只能拿到50分的暴力分了！

考虑把原数列离散化。然后对于区间\[l, r\]内的数x，将它的cnt保存下来，然后将保存下的所有cnt排个序。这样我们就可以从左往右扫cnt数组，对于第i个cnt，我们将cnt\[i\]及其前面的所有cnt填平到cnt\[i+1\]的高度，答案也就需要乘上![](http://latex.codecogs.com/svg.latex?(cnt_{i+1}!/cnt_i!)^i)，预处理阶乘及其逆元，再写一个快速幂即可

当然，并不是要一直填平。若继续填平会导致操作次数大于m，显然需要停止。设c为此时剩余操作次数，此时只要填平到c/i的高度就可以了，然后再处理一下剩余的c%i次操作即可

考虑到l和r的跨度可能比较大，中间可能会有很多元素是原数列里没有的，所以我们计算出\[l, r\]中在原数列里没有出现的元素数量s，在排序后的cnt数组头部加一个0，并且给它附上一个系数s即可

记住不要每次询问都把cnt数组整个memset一遍，否则复杂度会退化成O(Tn)。只需清空cnt数组的前n个元素即可。

```cpp
#include<bits/stdc++.h>
using namespace std;

namespace IO
{
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
    char obuf[S],*oS=obuf,*oT=oS+S-1,c,qu[55];int qr;
    inline void flush(){fwrite(obuf,1,oS-obuf,stdout);oS=obuf;}
    inline void putc(char x){*oS++ =x;if(oS==oT) flush();}
    template <class I>inline void print(I x)
    {
        if(!x) putc('0');
        if(x<0) putc('-'),x=-x;
        while(x) qu[++qr]=x%10+'0',x/=10;
        while(qr) putc(qu[qr--]);
    }
}

using namespace IO;
const int N=200010;
const int M=10200010;
const int ha=998244353;
int n,m,l,r,cnt[N],sum[N];
int a[N],Hash[N],hs;
int fac[M],ifac[M];

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

int main()
{
    Init(10200000);
    for(int T_T=read();T_T;T_T--)
    {
        n=read();m=read();l=read();r=read();
        memset(cnt,0,sizeof(int)*(n+5));
        memset(sum,0,sizeof(int)*(n+5));
        int all=fac[n+m],rest=r-l+1,ans=1;
        for(int i=1;i<=n;i++) Hash[i]=a[i]=read();
        sort(Hash+1,Hash+1+n);
        hs=unique(Hash+1,Hash+1+n)-(Hash+1);
        for(int i=1;i<=hs;i++) rest-=(Hash[i]>=l&&Hash[i]<=r);
        for(int i=1;i<=n;i++)
        {
            a[i]=lower_bound(Hash+1,Hash+1+hs,a[i])-Hash;
            if(Hash[a[i]]>=l&&Hash[a[i]]<=r) cnt[a[i]]++;
            sum[a[i]]++;ans=1ll*ans*sum[a[i]]%ha;
        }
        sort(cnt+1,cnt+1+hs);int zero=0;
        for(int i=1;i<=hs;i++) if(cnt[i]==0) zero=i;
        for(int i=1;i<=hs-zero;i++) cnt[i]=cnt[i+zero]%ha;
        n=hs-zero;
        for(int i=0;i<n;i++)
        {
            if(cnt[i+1]==cnt[i]) continue;
            if(1ll*(cnt[i+1]-cnt[i])*(i+rest)<=m)
            {
                m-=(cnt[i+1]-cnt[i])*(i+rest);
                int x=1ll*fac[cnt[i+1]]*ifac[cnt[i]]%ha;
                ans=1ll*ans*Pow(x,i+rest)%ha;
            }
            else
            {
                int x=m/(i+rest);
                int t=1ll*fac[cnt[i]+x]*ifac[cnt[i]]%ha;
                ans=1ll*ans*Pow(t,i+rest)%ha;
                ans=1ll*ans*Pow(cnt[i]+x+1,m%(i+rest))%ha;
                m=0;break;
            }
        }
        if(m>0)
        {
            int x=m/(r-l+1);
            int t=1ll*fac[cnt[n]+x]*ifac[cnt[n]]%ha;
            ans=1ll*ans*Pow(t,r-l+1)%ha;
            ans=1ll*ans*Pow(cnt[n]+x+1,m%(r-l+1))%ha;
        }
        ans=1ll*all*Pow(ans,ha-2)%ha;
        print(ans);putc('\n');
    }
    flush();
    return 0;
}
```

