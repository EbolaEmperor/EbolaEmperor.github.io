# 【JXOI2018】守卫

题目链接：[2545. 守卫  -  LibreOJ](https://loj.ac/problem/2545)

本题的误导性极强，看到题大部分人都会去想凸包、单调栈一类的东西

显然的有这么一个事实：若游戏的区间是\[l, r\]，那么r号亭子必然需要放一个保镖

于是我们确定右端点，然后可以从右往左扫，记p表示r号亭子能看到的最左边的亭子。然后扫的过程中，用一个sum记录p及其右边部分的答案。因为r最左端能看到的点是p，所以p-1的纵坐标必然低于p，所以对于\[l, p-1\]这一段，我们必然要在p或p-1中选一个放置保镖，故有：

![](http://latex.codecogs.com/svg.latex?f_{l,r}=sum+\min(f_{l,p-1},f_{l,p}))

在扫的过程中，p和sum也是需要更新的，若当前点是r号亭子可见的，那么必然要在之前的p和p-1中选一个放置保镖，所以sum要加上![](http://latex.codecogs.com/svg.latex?\min(f_{l+1,p-1},f_{l+1,p}))，然后将p更新至当前的l

考虑可见性判定，显然只要l与r的连线，比p与r的连线斜率更大，l就是可见的

就这样，做完了……如果想的方向对了真的非常简单

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=5010;
int f[N][N],h[N],n;

double slope(int l,int r){return (double)(h[r]-h[l])/(r-l);}
bool cansee(int l,int x,int r){return slope(x,r)>slope(l,r);}

int main()
{
    int ans=0;
    scanf("%d",&n);
    for(int i=1;i<=n;i++) scanf("%d",h+i);
    for(int r=1;r<=n;r++)
    {
        ans^=(f[r][r]=1);
        int sum=1,p=0;
        for(int l=r-1;l>=1;l--)
        {
            if(!p||cansee(l,p,r)) sum+=min(f[l+1][p-1],f[l+1][p]),p=l;
            ans^=(f[l][r]=sum+min(f[l][p-1],f[l][p]));
        }
    }
    cout<<ans<<endl;
    return 0;
}
```

