# A. 作业

### 题意简述

已知![](http://latex.codecogs.com/svg.latex?f_0=1-\frac{1}{e},\;\;\;f_n=1-nf_{n-1})，给定n求fn

### 题解

乍一看似乎非常简单，用来当普及组签到题的？但这是NOI模拟赛啊qwq

好的恭喜获得10分的好成绩。。。别忘了有一种东西叫”精度“

试想，如果我们这么去算，精度误差会指数级放大，终有一刻会由于精度误差而爆负，然后就会突破平衡，决堤似地迅速增长，然后就gg了

其实不难发现，随着n的不断增大，fn会越来越小，最后趋近于0。于是我们可以认为当n很大时，fn=0，然后倒着推回去，精度可以保证

fn趋0的证明可以把递推式写成通项，然后再泰勒展开一下就完了

```cpp
#include<bits/stdc++.h>
using namespace std;
 
int main()
{
    int n;cin>>n;
    double res=0,tmp=1;
    for(int i=100000;i>n;i--)
        res=(1-res)/i;
    printf("%.4lf\n",res);
    return 0;
}
```
