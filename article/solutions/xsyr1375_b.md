# B. 遇见

### 题意简述

给定一个序列，求有多少个区间，满足区间内所有数的出现次数都为奇数。序列长度不超过30000，值域范围10^6

### 题解

如果值域范围在60以内，不妨用一个二进制数来表示奇偶状态。具体地，枚举左端点l，设sum\[p\]表示l到p的奇偶状态，如果某个数在\[l,p\]区间中出现偶数次，则对应的二进制位为1，否则为0。于是我们直接看有多少个右端点r满足sum\[r\]=0即可。枚举到一个新的左端点，需要更新sum数组，设l右边第一个值与l相等的位置为x，直接将sum\[x到n\]异或上l的值对应的二进制位即可

当值域范围大于60时，似乎可以使用bitset来解决？空间复杂度没错，更新sum数组的复杂度也是正确的。可是查询某个sum值是否为0，需要远大于O(1)的复杂度。本来我们的O(n²)就是卡着过的，再乘个查询复杂度岂不是爆炸？

那如果我们不对应什么二进制位，直接异或值本身呢？那冲突的概率太大了

于是就有奇技淫巧了：我们给每个值赋一个随机值，随机范围越大越好。然后更新sum数组时，我们异或某个值对应的随机值，这样冲突的概率就小了很多，可以说几乎不会冲突了！

然后就做完了，真有意思

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=1000010;
int n,a[N],nxt[N],pos[N];
LL rnd[N],sum[N];

int main()
{
    scanf("%d",&n);
    for(int i=1;i<=n;i++) scanf("%d",a+i),pos[a[i]]=n+1;
    for(int i=n;i>=1;i--) nxt[i]=pos[a[i]],pos[a[i]]=i;
    for(int i=1;i<=1000000;i++) rnd[i]=(LL)rand()<<30|rand();
    int ans=0;
    for(int i=n;i>=1;i--)
    {
        for(int p=nxt[i];p<=n;p++) sum[p]^=rnd[a[i]];
        for(int p=i;p<=n;p++) if(!sum[p]) ans++;
    }
    printf("%d\n",ans);
    return 0;
}
```