# A. ichi

### 题面

![](http://www.ebola.pro/images/xsyr1441_a_1.png)

### 题解

考虑从上往下填，将图分成若干层，每层是一个矩形，从最顶层的矩形开始填一下，然后把剩余的砖填在下面那几层的矩形中

对于区域的分层，可以用单调栈来解决。维护一个高度单调递增的栈，每个节点是一个矩形，记录矩形的左边界以及顶边高。对于某一列，如果前面有顶边比它高的矩形，则将那个矩形填满，矩形的高度可以计算得出，填完之后剩下的砖可以送给稍矮一点的那个矩形。然后将该列最顶端的矩形入栈

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=100010;
struct node{int h,l,rest;};
stack<node> stk;
int n,k,h[N];

int main()
{
    scanf("%d%d",&n,&k);
    for(int i=1;i<=n;i++)
        scanf("%d",h+i);
    stk.push({0,0,0});
    LL ans=0;
    for(int i=1;i<=n+1;i++)
    {
        int rest=0,left=i;
        while(stk.size()>1&&stk.top().h>=h[i])
        {
            node top=stk.top();stk.pop();
            top.rest+=rest;left=top.l;
            int d=max(stk.top().h,h[i]);
            LL S=1ll*(top.h-d)*(i-top.l)-top.rest;
            if(S<=0) rest=-S;
            else
            {
                LL cnt=(S-1)/k+1;
                ans+=cnt;
                rest=k*cnt-S;
            }
        }
        stk.push({h[i],left,rest});
    }
    printf("%lld\n",ans);
    return 0;
}
```