# 【JXOI2017】颜色

题目链接：[2275. 颜色  -  LibreOJ](https://loj.ac/problem/2275)

每种删色方案与合法剩余区间是一一对应的，因此我们转而求合法剩余区间

设min\[x\]表示颜色x最左边出现的位置，max\[x\]表示颜色x最右边出现的位置

对于一个确定的右端点r，它与左端点l构成的区间不合法，可能是如下两种情况

1. 若max\[color\[x\]\]>r，则位于\[1, x\]的左端点均不合法
2. 若max\[c\]≤r，则位于(min\[c\], max\[c\]]的左端点均不合法

不难 ~~感性~~ 证明上述两个条件的充分性与必要性

于是我们可以维护一个单调栈，对于每个位置p，我们都将p入栈，然后在栈里面找到第一个满足条件1的x，把x上面的所有元素弹掉，然后在区间\[x, r\]中考虑条件2

我们可以实现一个支持区间覆盖的线段树，然后\[x, r\]中被覆盖的位置数量，就是满足条件2的不合法左端点，减去即可

~~怎么每次做吉老师的JXOI题，TLE时的瓶颈都出在memset~~

```cpp
#include<bits/stdc++.h>
#define FR first
#define SE second
using namespace std;

typedef pair<int,int> pii;
const int N=300010;
int col[N],mn[N],mx[N],n;
int tag[N<<2],val[N<<2];
pii stk[N];

void pushdown(int o,int l,int r)
{
    if(!tag[o]) return;
    int mid=(l+r)/2;
    val[o<<1]=mid-l+1;
    val[o<<1|1]=r-mid;
    tag[o<<1]=tag[o<<1|1]=1;
    tag[o]=0;
}

void modify(int o,int l,int r,int nl,int nr)
{
    if(l>=nl&&r<=nr){val[o]=r-l+1;tag[o]=1;return;}
    int mid=(l+r)/2;pushdown(o,l,r);
    if(nl<=mid) modify(o<<1,l,mid,nl,nr);
    if(nr>mid) modify(o<<1|1,mid+1,r,nl,nr);
    val[o]=val[o<<1]+val[o<<1|1];
}

int query(int o,int l,int r,int nl,int nr)
{
    if(l>=nl&&r<=nr) return val[o];
    int mid=(l+r)/2,res=0;pushdown(o,l,r);
    if(nl<=mid) res+=query(o<<1,l,mid,nl,nr);
    if(nr>mid) res+=query(o<<1|1,mid+1,r,nl,nr);
    return res;
}

int main()
{
    int T_T;scanf("%d",&T_T);
    while(T_T--)
    {
        scanf("%d",&n);
        memset(mn,0x7f,sizeof(int)*(n+3));
        memset(mx,0,sizeof(int)*(n+3));
        memset(tag,0,sizeof(int)*(n<<2));
        memset(val,0,sizeof(int)*(n<<2));
        for(int i=1;i<=n;i++)
        {
            scanf("%d",col+i);
            mn[col[i]]=min(mn[col[i]],i);
            mx[col[i]]=max(mx[col[i]],i);
        }
        long long ans=0;
        for(int i=1,top=0;i<=n;i++)
        {
            if(i==mx[col[i]]&&mn[col[i]]<mx[col[i]])
                modify(1,1,n,mn[col[i]]+1,mx[col[i]]);
            else stk[++top]=pii(col[i],i);
            while(top&&mx[stk[top].FR]<=i) top--;
            int l=top?stk[top].SE:0;
            if(l<i) ans+=i-l-query(1,1,n,l+1,i);
        }
        printf("%lld\n",ans);
    }
    return 0;
}
```

