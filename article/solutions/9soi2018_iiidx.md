# 【九省联考2018】IIIDX

**题目链接：[IIIDX - 洛谷](https://www.luogu.org/problemnew/show/P4364)**

不难发现题中所述关系构成了一棵树。在di互不相同的时候可以考虑直接贪心，贪心方法比较naive，这里就不赘述了

将所有di从大到小排序，对于每个节点，为它的子树预留一段空间。具体地，记该节点子树大小为s，在di数组中找到第一个前面还有s个未使用值的位置，找到与这个位置值相同的最后一个位置p，将p分配给当前节点，然后在p前面预留一段s的位置

现在说说预留方法。设f\[i\]表示位置i及其前面**至多**还有fi个未使用位置。则把p分配给当前节点时，f\[p\]到f\[n\]都需要减去s，前面的不确定，我们不进行任何操作

找第一个前面还有s个未使用值的位置，可以在线段树上二分。对于当前线段树节点，看它的右儿子中f的最小值是否小于s。如果是，显然左半区间中不存在我们需要的位置，就去找右半区间；如果否，则可以考虑到左半区间中试着找一找

因为f数组有“至多”这个定义，导致思维模糊，理解起来非常困难，可以自己举个例子模拟一遍，然后自己YY一下，这个YY过程我也帮不了大家了qwq

具体实现只要一棵维护区间最小f值的线段树就可以了，需要支持加法操作

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=500010;
int mn[N<<2],tag[N<<2];
int n,a[N],cnt[N],used[N];
int fa[N],sz[N],ans[N];
bool fuck[N];

void pushdown(int o)
{
    if(!tag[o]) return;
    mn[o<<1]+=tag[o];
    mn[o<<1|1]+=tag[o];
    tag[o<<1]+=tag[o];
    tag[o<<1|1]+=tag[o];
    tag[o]=0;
}

void build(int o,int l,int r)
{
    if(l==r){mn[o]=l;return;}
    int mid=(l+r)/2;
    build(o<<1,l,mid);
    build(o<<1|1,mid+1,r);
    mn[o]=min(mn[o<<1],mn[o<<1|1]);
}

void add(int o,int l,int r,int nl,int nr,int x)
{
    if(l>=nl&&r<=nr)
    {
        mn[o]+=x;tag[o]+=x;
        return;
    }
    int mid=(l+r)/2;
    pushdown(o);
    if(nl<=mid) add(o<<1,l,mid,nl,nr,x);
    if(nr>mid) add(o<<1|1,mid+1,r,nl,nr,x);
    mn[o]=min(mn[o<<1],mn[o<<1|1]);
}

int query(int o,int l,int r,int k)
{
    if(l==r) return mn[o]>=k?l:l+1;
    int mid=(l+r)/2;
    pushdown(o);
    if(mn[o<<1|1]>=k) return query(o<<1,l,mid,k);
    else return query(o<<1|1,mid+1,r,k);
}

int main()
{
    double k;
    scanf("%d%lf",&n,&k);
    for(int i=1;i<=n;i++)
    {
        scanf("%d",a+i);
        fa[i]=(int)floor(i/k);
        sz[i]=1;
    }
    for(int i=n;i>=1;i--) sz[fa[i]]+=sz[i];
    sort(a+1,a+1+n,[](int x,int y){return x>y;});
    for(int i=n;i>=1;i--)
        cnt[i]=(a[i+1]==a[i])?(cnt[i+1]+1):1;
    build(1,1,n);
    for(int i=1;i<=n;i++)
    {
        if(fa[i]&&!fuck[fa[i]])
        {
            add(1,1,n,ans[fa[i]],n,sz[fa[i]]-1);
            fuck[fa[i]]=1;
        }
        int x=query(1,1,n,sz[i]);
        x+=cnt[x]-1;used[x]++;
        ans[i]=(x-=used[x]-1);
        add(1,1,n,x,n,-sz[i]);
    }
    for(int i=1;i<=n;i++)
        printf("%d ",a[ans[i]]);
    return 0;
}
```