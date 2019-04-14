# 【AGC020C】Median Sum

### 题意简述

给定一个序列（长度和值域范围2000），求该序列所有非空子集权值和的中位数

### 题解

结论：将序列划分成两个权值和尽可能相近的集合，较大的那个权值和就是答案。证明显然

于是可以开一个值域bitset，方便查找哪些值可以通过序列中的若干个数相加得到

然后贪心地令两个集合权值和尽可能靠近，于是从 (所有数之和 + 1) / 2 这个值开始向上枚举，得到的第一个能通过序列中若干个数相加得到的值即为答案

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=2010;
int x,n,sum=0,ans;
bitset<N*N> S;

int main()
{
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
        scanf("%d",&x),S|=S<<x,S[x]=1,sum+=x;
    for(ans=(sum+1)/2;ans<=sum;ans++) if(S[ans]) break;
    printf("%d\n",ans);
    return 0;
}
```

