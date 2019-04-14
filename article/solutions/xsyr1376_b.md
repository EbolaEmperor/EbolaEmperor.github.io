# B. 景中人

### 题意简述

二维平面上有n个点。你需要用底边位于x轴且面积不超过S的矩形，把所有点覆盖起来。求最少需要多少个矩形。本题中，我们认为与y轴平行的直线面积为0

### 题解

一定存在最优策略满足：两个矩形的横坐标要么相离，要么包含

证明：如果两个矩形横坐标相交且无包含关系，那更矮的那个矩形完全没必要伸进更高的那个矩形里面去。

考虑dp。设f\[l\]\[r\]\[h\]表示：将横坐标范围在\[l,r\]中，且纵坐标不小于h的所有点覆盖，所需的最少矩形数量

转移可以直接记忆化搜索。具体地，先把l往右移动到横坐标范围\[l,r\]中最左边的纵坐标不小于h的点，对r也做类似的操作。然后我们有两种选择：要么以\[l,r\]为底画一个尽可能大的矩形，然后去处理矩形上面还没覆盖的点；要么把区间拆成两段然后把答案加起来

注意横纵坐标范围较大，要先把坐标离散化一下

具体细节见代码注释。复杂度O(n^4)

```cpp
#include<bits/stdc++.h>
using namespace std;

const int INF=0x3f3f3f3f;
const int N=110;
struct Point
{
    int x,y;
    bool operator < (const Point &p) const{return x<p.x||x==p.x&&y<p.y;}
    bool operator == (const Point &p) const{return x==p.x&&y==p.y;}
} pt[N];
int f[N][N][N];
int Hash[N],tot=0;
int n,S,T;

int geth(int l,int r){return pt[l].x==pt[r].x?INF-1:S/(pt[r].x-pt[l].x);}
int dp(int l,int r,int h)
{
    int &res=f[l][r][h];
    if(~res) return res;res=INF;
    while(pt[l].y<Hash[h]&&l<=r) l++;  //去除两端纵坐标小于待处理高度的点
    while(pt[r].y<Hash[h]&&l<=r) r--;
    if(l>r) return res=0;
    for(int i=l;i<r;i++) res=min(res,dp(l,i,h)+dp(i+1,r,h));  //枚举拆分点，将待处理区间分成两份，答案相加
    int t=upper_bound(Hash+1,Hash+1+tot,geth(l,r))-Hash;  //求得以[l,r]为底的矩形最高能到哪里
    if(t<=h) return res;
    res=min(res,1+dp(l,r,t));  //画一个最大的矩形之后去处理上面还没覆盖到的
    return res;
}

int main()
{
    for(scanf("%d",&T);T;T--)
    {
        memset(f,-1,sizeof(f));
        scanf("%d%d",&n,&S);
        for(int i=1;i<=n;i++)
            scanf("%d%d",&pt[i].x,&pt[i].y);
        sort(pt+1,pt+1+n);  //先将点排好序
        for(int i=1;i<=n;i++)
            Hash[i]=pt[i].y;
        sort(Hash+1,Hash+1+n);
        tot=unique(Hash+1,Hash+1+n)-(Hash+1);  //纵坐标离散化
        Hash[++tot]=INF;  //加入一个极大值
        int ans=dp(1,n,1);
        printf("%d\n",ans);
    }
    return 0;
}
```