# A. Dandelion

### 题意简述

有n个任务，完成第i个任务要求当前能力值至少为si，完成任务需要消耗ti天，完成后你的能力值可以增加pi，一个任务只能做一次。你的初始能力值为R，总共有T天的时间，你可以自己选择一些任务完成，求最后能力值最大是多少，输出方案

### 题解

显然要想最后的能力值最大，每天的能力值都要尽可能大

记录每一天的最大能力值，并记录这个能力值是完成哪项任务后变成的，以及这项任务开始时是哪一天。把那天的任务完成标记给继承过来，再把这项任务打上完成标记即可

最后输出方案只要沿着我们记录的信息回到第0天，顺路存一下完成了哪些任务就行了。最后把存的任务反过来输出就行了

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=1010;
struct Dan{int s,p,t;} dan[N];
int choice[N],pre[N];
int n,T,magic[N];
bool used[N][N];

int main()
{
    scanf("%d%d%d",&n,&T,magic);
    for(int i=1;i<=n;i++)
        scanf("%d%d%d",&dan[i].s,&dan[i].p,&dan[i].t);
    int ans=magic[0],ansd=0;
    for(int i=0;i<=T;i++)
    {
        if(magic[i]>ans) ans=magic[i],ansd=i;
        memcpy(used[i],used[pre[i]],sizeof(used[i]));
        if(i>0) used[i][choice[i]]=1;
        for(int j=1;j<=n;j++)
        {
            if(used[i][j]||dan[j].s>magic[i]) continue;
            int x=i+dan[j].t,y=magic[i]+dan[j].p;
            if(x>T) continue;
            if(y>magic[x]) magic[x]=y,pre[x]=i,choice[x]=j;
        }
    }
    printf("%d\n",ans);
    stack<int> stk;
    for(int u=ansd;u;u=pre[u])
        stk.push(choice[u]);
    while(!stk.empty())
        printf("%d ",stk.top()),stk.pop();
    return 0;
}
```
