# 小Y增员操直播群

### 题意简述

对于两个节点个数分别为n和m的图（n<=m），我们可以把它拼起来，假设第一个图中的点为a0,…,an-1，第二个图中的点为b0,…,bm-1，则对于0<=i<m，连一条a(i mod n)到b(i)的边，然后把b中的每个点的标号都加上n。

每一次拼接需要花1单位时间，可以多线程进行。一开始你有n个节点数为1的图，求最少需要多少时间才能拼出给定的图（或判断不合法）。

### 题解

显然一个图的点编号在最终的图中是连续的。现在假设要拼出图\[0,h\]。不难发现h号节点所连的最小编号节点一定是在最近一次操作连出去的，所以h所连的编号最小点（设为p）一定在较小的图中，h-p-1号节点所连的最小编号节点一定是较小图的最大编号节点

于是我们就把较小图和较大图分离出来了，对较大图中的每个点判断一下最近连的那个点是不是本次操作需要连的点，然后再对两个子图递归进行上述过程即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010;
vector<int> lft[N];
int cur[N];
int T,n,m;

int divide(int l,int r)
{
    if(l==r) return cur[l]==lft[l].size()?0:-1;
    int mid=(l+r)/2;
    if(cur[r]==lft[r].size()) return -1;
    int p=lft[r][cur[r]++],len=p-l+1;
    if(p<l||p>mid) return -1;
    if((r-l+1)%2==0&&p==mid)
    {
        for(int i=1;i<len;i++)
        {
            int q=r-i;
            if(cur[q]==lft[q].size()) return -1;
            if(q-lft[q][cur[q]++]!=r-p) return -1;
        }
        int L=divide(l,p);
        if(L==-1) return -1;
        int R=divide(p+1,r);
        if(R==-1) return -1;
        return max(L,R)+1;
    }
    if(cur[r-len]==lft[r-len].size()) return -1;
    int u=lft[r-len][cur[r-len]],sz=u-l+1;
    if(u<l||u>mid) return -1;
    for(int i=u+1;i<r;i++)
    {
        if(cur[i]==lft[i].size()) return -1;
        if(lft[i][cur[i]++]!=(i-l)%sz+l) return -1;
    }
    int L=divide(l,u);
    if(L==-1) return -1;
    int R=divide(u+1,r);
    if(R==-1) return -1;
    return max(L,R)+1;
}

int gao()
{
    scanf("%d%d",&n,&m);
    for(int i=0;i<n;i++) lft[i].clear(),cur[i]=0;
    bool falun=0;
    for(int i=0,x,y;i<m;i++)
    {
        scanf("%d%d",&x,&y);
        if(x>y) swap(x,y);
        if(x==y) falun=1;
        lft[y].push_back(x);
    }
    if(falun) return -1;
    for(int i=0;i<n;i++)
        sort(lft[i].begin(),lft[i].end());
    return divide(0,n-1);
}

int main()
{
    scanf("%d",&T);
    while(T--) printf("%d\n",gao());
    return 0;
}
```
