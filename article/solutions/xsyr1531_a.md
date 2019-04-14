# A. 地中海气候

### 题意简述

给定一个长度为n(n≤1e5)的数组A，两人在玩游戏。游戏一开始有p个数（数组的前p个元素），每人轮流取一个数，取得的数计入得分。第i次取数后，A\[p+i\]会加入可选范围，直到取空整个数组，游戏结束。两人都会最大化自己得分，求先手得分与后手之差

一共进行了k(k≤2000)轮游戏，每轮的p都是不一样的

### 题解

最近脑子大概被挖了个坑，又成功地场上想出正解，然后因为访问了两倍的数组空间WA掉了……立个flag，以后打比赛的时候，只要空间够，数组一定开两倍

考虑维护当前的可选范围，用一个单调右移的指针q来维护。从大到小考虑数组A的元素。如果某个数x的位置在当前可选范围的右边，则肯定是那时候轮到谁就谁拿走，可以根据数x的位置与p的差值奇偶性得出那时候轮到谁。如果x在当前可选范围左边，则此时进行到谁的回合，就归谁，此时进行到谁的回合可以根据q与p的差值奇偶性来得到

需要注意的是，x在当前可选范围右边时，我们相当于预定了q移动到x的位置时的操作。所以我们需要将q移动到右边第一个未被预定的位置，这样得出的x到底轮到谁取才是正确的

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=200010;
struct NUM{int x,p;} a[N];
int n,m;
LL ans[N];
int p[N],pp[N];
bool done[N];

inline int fg(int x){return (x&1)?-1:1;}

int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++)
    {
        scanf("%d",&a[i].x);
        a[i].p=i;
    }
    for(int i=1;i<=m;i++) scanf("%d",p+i),pp[i]=p[i];
    sort(a+1,a+1+n,[](NUM a,NUM b){return a.x<b.x;});
    for(int i=n;i>=1;i--)
    {
        int pos=a[i].p;
        for(int j=1;j<=m;j++)
        {
            while(done[pp[j]]) pp[j]++;  //注意pp[j]可能会超过n，所以done数组要开两倍！
            if(pos<=pp[j]) ans[j]+=fg(pp[j]-p[j])*a[i].x,pp[j]++;
            else ans[j]+=fg(pos-p[j])*a[i].x;
        }
        done[pos]=1;
    }
    for(int i=1;i<=m;i++)
        printf("%lld\n",ans[i]);
    return 0;
}
```