# A. 跑步

### 题意简述

有一个n\*n(n≤2000)的网格，每个格子上有一个数字。每次可以往左或往上走一步，对于每个出发点，求出按上述规则走到左上角，途经数字的和最大值。输出所有出发点的答案之和。此外，会有n次修改操作，每次令一个格子加一或减一，每次操作后都要输出所有出发点的答案之和

### 题解

好不容易在场上想出一次正解，结果被错解覆盖了qwq，然后还被卡常了qwq

n²的dp显然。问题在于n次修改操作

考虑修改一个位置，左下方的某些位置会受到影响，而每条左下-右上对角线上，受影响的位置必然连续（显然。画图即可理解）。于是我们对每条左下-右上对角线求出受影响的区间，从一条对角线推到下一条时，显然只要判断左右端点是否扩展或缩减一位就行了。判断的时候有一些细节不太好写，需要自己推敲一下

现在我们需要快速修改对角线上一段区间的dp值。我们需要一个支持区间加法、单点查询的数据结构，树状数组显然是最好的选择。于是对每条左下-右上对角线维护一个dp值的树状数组即可

现在问题是，判断一个端点时，需要询问两个dp值，那一次判断就是四次询问，还要附带一次区间修改（实质上是两次单点修改）。然后对角线有2n条，所以我们的时间是O(12 \* n²log n)，非常容易被卡（我超了230ms，已经是极致常数了）

于是我们画出受影响的位置图，我们发现，在横排上，受影响的位置也是一段连续的区间，而且往下推时，区间左右端点均单调右移！于是我们可以对横排维护树状数组，然后维护受影响的左右端点，利用单调的性质可以证明询问次数不超过4n次。此时我们时间降到了O(6 \* n² log n)。然而实践证明，这么写比维护对角线快了远远不止2倍

```cpp
#include<bits/stdc++.h>
using namespace std;

const int S=(1<<20)+5;
char buf[S],*H,*T;
inline char Get()
{
    if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
    if(H==T) return -1;return *H++;
}
inline int read()
{
    int x=0;char c=Get();
    while(!isdigit(c)) c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x;
}
inline char readc()
{
    char c=Get();
    while(c!='U'&&c!='D') c=Get();
    return c;
}

typedef long long LL;
const int N=2010;
int num[N][N],n;
int f[N][N],bit[N][N];
LL ans=0;

inline int lowbit(int x){return x&-x;}
void add(int d,int p,int x){for(;p<=n+1;p+=lowbit(p)) bit[d][p]+=x;}
int query(int d,int p){int res=d?0:-n;for(;p;p-=lowbit(p)) res+=bit[d][p];return res;}

int change1(int x,int y)
{
    int l=y,r=y,cnt=0;
    while(r<n&&query(x,r)+1>query(x-1,r+1)) r++;
    add(x,l,1);add(x,r+1,-1);cnt+=r-l+1;
    for(int i=x+1;i<=n;i++)
    {
        while(r<n&&query(i,r)+1>query(i-1,r+1)) r++;
        while(query(i,l-1)>=query(i-1,l)) l++;
        if(l>r) break;
        add(i,l,1);add(i,r+1,-1);
        cnt+=r-l+1;
    }
    return cnt;
}

int change2(int x,int y)
{
    int l=y,r=y,cnt=0;
    while(r<n&&query(x,r)-1>=query(x-1,r+1)) r++;
    add(x,l,-1);add(x,r+1,1);cnt+=r-l+1;
    for(int i=x+1;i<=n;i++)
    {
        while(r<n&&query(i,r)-1>=query(i-1,r+1)) r++;
        while(query(i,l-1)>query(i-1,l)) l++;
        if(l>r) break;
        add(i,l,-1);add(i,r+1,1);
        cnt+=r-l+1;
    }
    return cnt;
}

int main()
{
    n=read();
    for(int i=1;i<=n;i++)
        for(int j=1;j<=n;j++)
        {
            num[i][j]=read();
            f[i][j]=max(f[i-1][j],f[i][j-1])+num[i][j];
            add(i,j,f[i][j]);add(i,j+1,-f[i][j]);
            ans+=f[i][j];
        }
    printf("%lld\n",ans);
    char opt;int x,y;
    for(int T=1;T<=n;T++)
    {
        opt=readc();x=read();y=read();
        if(opt=='U') ans+=change1(x,y);
        else ans-=change2(x,y);
        printf("%lld\n",ans);
    }
    return 0;
}
```