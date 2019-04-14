# C. recog

### 题意简述

提答题。给了你一万张28\*28的灰度图，每个图都是幼儿园小朋友写的一个数字，你需要识别出这一万个数字，要求识别率95%以上。此外，还提供了五万组样例（含灰度图及答案）

下面给大家近距离感受一下小朋友们的字有多丑：

![](http://www.ebola.pro/images/xsyr1534_c_1.png)

### 题解

~~人工识别两个半小时，可以AC~~

如果你会炼丹，就可以秒切了，然而我不会，于是考虑暴力

定义两个灰度图的差异值为每个像素点的差异值之和。对于一个待识别的图，在给定的五万组样例中找到差异值最小的图，输出对应答案即可。运行时间大约10分钟，识别率高达95.4%

```cpp
#include<bits/stdc++.h>
using namespace std;

double s[50000][28][28];
double deal[28][28];
int ans[50000];

int solve()
{
    double mx=1e9;int mxid;
    for(int T=0;T<50000;T++)
    {
        double cnt=0;
        for(int i=0;i<28;i++)
            for(int j=0;j<28;j++)
                cnt+=fabs(s[T][i][j]-deal[i][j]);
        if(cnt<mx) mx=cnt,mxid=T;
    }
    return ans[mxid];
}

int main()
{
    freopen("training.in","r",stdin);
    for(int T=0;T<50000;T++)
        for(int i=0;i<28;i++)
            for(int j=0;j<28;j++)
                scanf("%lf",s[T][i]+j);
    freopen("training.out","r",stdin);
    for(int i=0;i<50000;i++) scanf("%d",ans+i);
    freopen("data.in","r",stdin);
    freopen("ans.out","w",stdout);
    for(int T=0;T<1000;T++)
    {
        for(int i=0;i<28;i++)
            for(int j=0;j<28;j++)
                scanf("%lf",deal[i]+j);
        cerr<<T<<endl;
        printf("%d\n",solve());
    }
    return 0;
}
```

