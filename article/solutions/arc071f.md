# 【ARC071F】Infinite Sequence

题目链接：[ARC071 F. Infinite Sequence  -  AtCoder](https://arc071.contest.atcoder.jp/tasks/arc071_d)

哇居然只读入一个数！打个表找找规律！斐波那契？卡特兰？等差？差分多项式定理？

（30分钟后）我可去你大爷！emmm……（打开OEIS），搜索失败……

（10分钟后）咦？如果某个位置大于1，它的下一个位置又大于1，那后面的数就没法再变了啊！好，记忆化搜索！对于每个位置p，假设它填1，则进入dfs(p+1)；枚举它填2到n之间的数，假设下一个位置填1，直接进入dfs(p+a\[p\]+1)，假设下一个位置填大于1的数，则后面无需再枚举，直接加n-1（即下一个位置的可填数字）即可

具体的记忆化搜索如下：

```cpp
int dfs(int p)
{
    int &res=f[p];
    if(~res) return res;
    if(p>n) return res=1;
    res=0;add(res,dfs(p+1));
    for(int i=2;i<=n;i++)
    {
        if(p<n) add(res,n-1);
        add(res,dfs(p+i+1));
    }
    return res;
}
```

（10分钟后）冷静分析，似乎复杂度n方……尝试根据记忆化搜索的代码写出递推式：

![](http://latex.codecogs.com/svg.latex?f_i=f_{i+1}+(n-1)^2+\sum_{i=2}^nf_{i+j+1})

初始条件：![](http://latex.codecogs.com/svg.latex?f_n=n,\quad&space;f_k=1(k>n))

哎舒服！后面这个求和是一段连续区间和，区间每次往左移动一格，于是dp的时候顺便维护一下就好了！O(n)搞定！

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=1e9+7;
int f[2000010],n;

void add(int &x,const int &y){x=(x+y>=ha)?(x+y-ha):(x+y);}
void dev(int &x,const int &y){x=(x-y<0)?(x-y+ha):(x-y);}

int main()
{
    scanf("%d",&n);
    int sum=n-1;f[n]=n;
    for(int i=n+1;i<=2*n;i++) f[i]=1;
    for(int i=n-1;i>=1;i--)
    {
        add(f[i],f[i+1]);
        f[i]=(f[i]+1ll*(n-1)*(n-1))%ha;
        add(f[i],sum);
        dev(sum,f[i+n+1]);
        add(sum,f[i+2]);
    }
    printf("%d\n",f[1]);
    return 0;
}
```

