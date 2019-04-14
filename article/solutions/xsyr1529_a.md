# A. 摸鱼

### 题意简述

给定一个排列，将其划分成两个子序列，使得一个上升，一个下降，求任意一个划分方案。多组询问，排列长度不超过1e5

### 题解

设f\[i\]\[0\]表示将第i个数划分给下降序列时，上升序列最后一个数的值；f\[i\]\[1\]表示将第i个数划分给上升序列时，下降序列最后一个数的值

然后需要求出pre\[i\]\[0/1\]表示当i划分给下降/上升序列时，i-1是否划分给上升序列，这样就能直接输出方案了

dp的转移还是比较显然的。具体请看代码及注释，建议画图理解

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010;
int a[N],n,T;
int f[N][2],pre[N][2];
stack<int> ans1,ans2;

int main()
{
    scanf("%d",&T);
    while(T--)
    {
        scanf("%d",&n);
        for(int i=1;i<=n;i++) scanf("%d",a+i);
        f[1][0]=0;f[1][1]=n+1;
        for(int i=2;i<=n;i++)
        {
            f[i][0]=n+1;f[i][1]=0;
            if(a[i]<a[i-1]&&f[i][0]>f[i-1][0]) f[i][0]=f[i-1][0],pre[i][0]=0;  //考虑将i-1和i一并加入下降序列
            if(a[i-1]<f[i][0]&&a[i]<f[i-1][1]) f[i][0]=a[i-1],pre[i][0]=1;  //考虑将i加入下降序列，将i-1加入上升序列
            if(a[i]>a[i-1]&&f[i][1]<f[i-1][1]) f[i][1]=f[i-1][1],pre[i][1]=1;  //考虑将i-1和i一并加入上升序列
            if(a[i-1]>f[i][1]&&a[i]>f[i-1][0]) f[i][1]=a[i-1],pre[i][1]=0;  //考虑将i加入上升序列，将i-1加入下降序列
        }
        int d;
        if(f[n][0]<=n) d=0;
        else if(f[n][1]>=1) d=1;
        else{puts("NO");continue;}
        for(int i=n;i>=1;d=pre[i][d],i--)
            if(d) ans1.push(a[i]);else ans2.push(a[i]);
        printf("YES\n%d ",ans1.size());
        while(!ans1.empty()) printf("%d ",ans1.top()),ans1.pop();
        printf("\n%d ",ans2.size());
        while(!ans2.empty()) printf("%d ",ans2.top()),ans2.pop();
        puts("");
    }
    return 0;
}
```
