# B. ni

题目链接：[「2017 山东二轮集训 Day2」第二题 - LibreOJ](https://loj.ac/problem/6104)

对于任意的第i列，我们只关心最上面那个好格子的高度，记做h\[i\]

设f\[i\]\[j\]表示考虑到第i列，剩余j个格子时的最优步数。则枚举k表示第i列和第i-1列进行了几次**反L型**操作。则i-1的剩余格子数应该是正L型操作次数乘2加上反L型操作次数。可以得到转移方程：

![](http://latex.codecogs.com/svg.latex?f_{i,j}=\min_{k=1}^{h_i/2}(f_{i-1,2(h_i-j-2k)+k}+k+(h_i-j-2k)))

直接按上式暴力去做可以得到50分

然后有一个神仙结论就是这个dp是具有决策单调性的。如果在k处取得了f\[i\]\[j\]的最小值，则f\[i\]\[j+1\]的最小值一定在小于等于k处取得。**证明的天坑待填**

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=2010;
int f[N][N],h[N],n,m;

int main()
{
    scanf("%d%d",&n,&m);
    static char s[N];
    for(int i=n;i>=1;i--)
    {
        scanf("%s",s+1);
        for(int j=1;j<=m;j++)
            if(s[j]=='*'&&!h[j]) h[j]=i;
    }
    memset(f,0x3f,sizeof(f));
    memset(f[0],0,sizeof(f[0]));
    for(int i=1;i<=m;i++)
        for(int j=0,k=h[i]/2;j<=h[i];j++)
            for(k=min((h[i]-j)/2,k+1);k>=0;k--)
            {
                int now=f[i-1][min(h[i-1],2*(h[i]-j-2*k)+k)]+k+(h[i]-j-2*k);
                if(now<=f[i][j]) f[i][j]=now;
                else break;
            }
    printf("%d\n",f[m][0]);
    return 0;
}
```