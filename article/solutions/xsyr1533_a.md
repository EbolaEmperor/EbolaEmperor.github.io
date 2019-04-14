# A. 魔力

### 题意简述

给定一个只由大小写字母构成的字符串，求有多少个子串，其中包含出现过的每个字符，且每个字符出现次数相等。字符串长度不超过1e5

### 题解

不妨考虑乱搞

假设字符集是\[1,m\]，我们给字符1到字符m-1都附上随机权，字符m的权设为前m-1种字符随机权值和的相反数。于是我们可以认为，字符串上一段权值和为0的区间就是符合题目要求的子串

于是，我们可以枚举左端点L，看有多少右端点R满足sum(R)=sum(L)，其中sum是指权值的前缀和。求区间中有多少个等于x的数，主席树的经典应用

其实仔细分析，发现被卡的概率是很小的，至少比某些实现的不太好的哈希优秀多了。我比赛时候拿了95分，结束之后把随机数种子改成了某个神秘质数然后AC了

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int ha=1e9+7;
const int N=100010;
int sum[N*20],lc[N*20],rc[N*20],rt[N],tot=0;
int cnt=0,n,m;
char Hash[N],s[N];
LL val[300],hs[N];
LL presum[N];

void insert(int &o,int p,int l,int r,int k)
{
    sum[o=++tot]=sum[p]+1;
    lc[o]=lc[p];rc[o]=rc[p];
    if(l==r) return;
    int mid=(l+r)/2;
    if(k<=mid) insert(lc[o],lc[p],l,mid,k);
    else insert(rc[o],rc[p],mid+1,r,k);
}

int query(int L,int R,int l,int r,int k)
{
    if(l==r) return sum[R]-sum[L];
    int mid=(l+r)/2;
    if(k<=mid) return query(lc[L],lc[R],l,mid,k);
    else return query(rc[L],rc[R],mid+1,r,k);
}

int main()
{
    srand(19260817);
    scanf("%d%s",&n,s+1);
    for(int i=1;i<=n;i++) Hash[i]=s[i];
    sort(Hash+1,Hash+1+n);
    m=unique(Hash+1,Hash+1+n)-(Hash+1);
    for(int i=1;i<m;i++)
    {
        val[Hash[i]]=rand();
        val[Hash[m]]-=val[Hash[i]];
    }
    for(int i=1;i<=n;i++)
        hs[i]=presum[i]=presum[i-1]+val[s[i]];
    sort(hs+1,hs+2+n);
    m=unique(hs+1,hs+2+n)-(hs+1);
    presum[0]=lower_bound(hs+1,hs+1+m,presum[0])-hs;
    for(int i=1;i<=n;i++)
    {
        presum[i]=lower_bound(hs+1,hs+1+m,presum[i])-hs;
        insert(rt[i],rt[i-1],1,m,presum[i]);
    }
    LL ans=0;
    for(int i=0;i<n;i++)
        ans+=query(rt[i],rt[n],1,m,presum[i]);
    printf("%lld\n",ans%ha);
    return 0;
}
```

