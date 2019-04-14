# A. 三角形

### 题目大意：

给定n个长度，从中选三个长度，请判断有多少种选法能构成直角三角形

n≤1e6，长度不超过1e5

### 题解

勾股数分布公式：

![](http://latex.codecogs.com/svg.latex?x=a^2-b^2,\quad&space;y=2ab,\quad&space;z=a^2+b^2)

于是我们只需要枚举a和b就行了。不难观察到a和b都是根号级别的，因此直接枚举复杂度是对的

因为勾股数翻倍后仍然成立，所以枚举到一对a、b后，把勾股数的倍数都统计一下。每组勾股数对答案的贡献显然就是三个数的数量之积。

注意因为我们对于每组勾股数，都统计了它的倍数，因此若后面算到了被之前某组勾股数更新过的，就不能重复计算。换句话说，只有当一组勾股数互质时，我们才去统计它

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
const int ha=998244353;
const int N=100010;
LL cnt[N],n,A,ans=0;

int main()
{
    n=read();A=read();
    for(int i=1;i<=n;i++) cnt[read()]++;
    for(LL i=1;i*i<=A;i++)
        for(LL j=1;j<=i;j++)
        {
            LL x=i*i-j*j,z=i*i+j*j,y=2ll*i*j;
            if(z>A) break;
            if(__gcd(__gcd(x,y),z)>1) continue;
            for(int a=x,b=y,c=z;c<=A;a+=x,b+=y,c+=z)
                ans=(ans+cnt[a]*cnt[b]*cnt[c])%ha;
        }
    printf("%lld\n",ans);
    return 0;
}
```

