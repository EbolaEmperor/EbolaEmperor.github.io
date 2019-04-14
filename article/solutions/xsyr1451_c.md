# C. LCIS

### 题意简述

给定一个序列，你可以进行多次区间翻转操作，要求使得最终序列的LIS最大。

翻转操作的区间总长不得超过400万，序列长度与值域范围均为32000

### 题解

要让LIS最大，其实就是要你把这个序列排个序

所以n方的算法就很显然了，直接冒泡排序。因为冒泡排序只会交换相邻的两个数，我们可以将它视作翻转了一个长度为2的区间，符合题目要求

继续考虑更高效的算法

如果整个序列只有0和1两种数字，那可以直接归并排序。归并排序时，两个子区间是已经排序好了的，所以左区间的0都在左边，右区间的1都在右边，于是我们直接把中间那段给翻转一下就好了

序列的值域大于1时，直接对值域再分治一下。对于每个分治区间，将mid作为分界点，小于等于mid的值视为0，大于mid的值视为1，然后按01排序的方法做一遍。然后继续分治下去即可

复杂度两个log，大概720万的样子，但根本不可能卡满

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int N=32010,M=4000010;
struct ANS{int l,r;} ans[M];
int a[N],n,tot=0;
int val[N];
 
int solve(int l,int r)
{
    if(l==r) return val[l];
    int mid=(l+r)/2;
    int L=solve(l,mid);
    int R=solve(mid+1,r);
    if(R&&L<mid-l+1)
    {
        ans[++tot]={l+L,mid+R};
        reverse(a+l+L,a+mid+R+1);
    }
    return L+R;
}
 
void Solve(int l,int r,int L,int R)
{
    if(l==r||L>=R) return;
    int mid=(l+r)/2;
    for(int i=L;i<=R;i++)
        val[i]=bool(a[i]<=mid);
    int cnt=solve(L,R);
    Solve(l,mid,L,L+cnt-1);
    Solve(mid+1,r,L+cnt,R);
}
 
int main()
{
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
        scanf("%d",a+i);
    Solve(0,32000,1,n);
    printf("%d\n",tot);
    for(int i=1;i<=tot;i++)
        printf("%d %d\n",ans[i].l,ans[i].r);
    return 0;
}
```