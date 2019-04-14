# A. 走楼梯

### 题意简述

给定一个长为n(n≤1e5)的序列，多次询问整个序列除去区间\[l,r\]后的LIS，强制在线

### 题解

发现没什么好做法，于是考虑分块（雾）

设f\[b\]\[r\]表示b号块及其之前以及r之后所有数的LIS，g\[b\]\[l\]表示只b号块及其之后以及l之前的所有数的LIS

考虑f的预处理。设d\[x\]表示以x号位置结束的LIS。我们将整个序列离散化，遇到相同数时认为位置靠前的数较大，这样就不会影响LIS。然后将d数组放到值域上，即：设dd\[x\]表示以离散化后值为x的数结束的LIS。于是，f\[b\]\[r\]首先可以从f\[b\]\[r+1\]转移过来，然后设r位置的数离散化后值为v，我们只需要求出由b号块及其之前的所有数构成的dd数组，然后直接在dd\[1 to v\]中找一个最大值并加上以位置r开头的LIS长度就可以转移f\[b\]\[r\]了。g数组的预处理也是类似的。直接前缀max优化一下就可以O(n sqrt(n))完成预处理

现在考虑询问。设l所在的块是bl，r所在的块是br，那么可以先用f\[bl-1\]\[r+1\]和g\[br+1\]\[l-1\]更新答案。l到块头和r到块尾有两段零散的区间，现在我们还没考虑的就是LIS同时经过这两个零散区间的情况。可以将这两个零散区间的所有数排序，然后从小到大扫描，遇到左区间的位置p就用以p结尾的LIS更新一个变量c，遇到右区间的位置p就用以p开头的LIS加上c来更新答案。为了保证复杂度正确，我们预处理每个块的排序，然后询问的时候就不必再排序，直接二路归并一下就行了。复杂度O(m sqrt(n))

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef pair<int,int> pii;
const int N=100010,B=330;
int dv[N],st[B],ed[B],tot=0;
int f[B][N],g[B][N];
int n,m,online,a[N];
int pre[N],suf[N];
int mxpre[N],mxsuf[N];
pii pred[N],sufd[N];
pii val[B][B+5];

void divide()
{
    for(int i=1,cnt=0;i<=n;i++)
    {
        if(!cnt) st[++tot]=i;
        dv[i]=tot;cnt++;
        if(cnt==B) ed[tot]=i,cnt=0;
    }
    ed[tot]=n;
}

void prework1()
{
    static int d[N];
    memset(d,0x3f,sizeof(d));
    for(int i=1;i<=n;i++)
    {
        int p=lower_bound(d+1,d+1+n,a[i])-d;
        pre[i]=p;d[p]=a[i];
        pred[a[i]]=pii(p,dv[i]);
    }
    memset(d,0x3f,sizeof(d));
    for(int i=n;i>=1;i--)
    {
        int p=lower_bound(d+1,d+1+n,-a[i])-d;
        suf[i]=p;d[p]=-a[i];
        sufd[a[i]]=pii(p,dv[i]);
    }
    for(int i=1;i<=n;i++) mxpre[i]=max(mxpre[i-1],pre[i]);
    for(int i=n;i>=1;i--) mxsuf[i]=max(mxsuf[i+1],suf[i]);
}

void prework2(int b)
{
    static int r[N];
    memset(r,0,sizeof(r));
    for(int i=1;i<=n;i++)
    {
        if(pred[i].second<=b) r[i]=pred[i].first;
        r[i]=max(r[i-1],r[i]);
    }
    f[b][n+1]=mxpre[ed[b]];
    for(int i=n;i>ed[b];i--)
        f[b][i]=max(f[b][i+1],r[a[i]]+suf[i]);
    memset(r,0,sizeof(r));
    for(int i=n;i>=1;i--)
    {
        if(sufd[i].second>=b) r[i]=sufd[i].first;
        r[i]=max(r[i+1],r[i]);
    }
    g[b][0]=mxsuf[st[b]];
    for(int i=1;i<st[b];i++)
        g[b][i]=max(g[b][i-1],r[a[i]]+pre[i]);
    for(int i=st[b];i<=ed[b];i++)
        val[b][i-st[b]+1]=pii(a[i],i);
    sort(val[b]+1,val[b]+(ed[b]-st[b]+1)+1);
}

int query(int l,int r)
{
    static pii v[N],t1[N],t2[N];
    int bl=dv[l],br=dv[r],c1=0,c2=0,cnt=0,p1=1,p2=1;
    int res=max(f[bl-1][r+1],g[br+1][l-1]);
    for(int i=1;i<=B;i++) if(val[bl][i].second&&val[bl][i].second<l) t1[++c1]=pii(val[bl][i].first,pre[val[bl][i].second]);
    for(int i=1;i<=B;i++) if(val[br][i].second&&val[br][i].second>r) t2[++c2]=pii(val[br][i].first,suf[val[br][i].second]);
    while(p1<=c1||p2<=c2)
    {
        if(p1>c1) v[++cnt]=pii(t2[p2++].second,2);
        else if(p2>c2) v[++cnt]=pii(t1[p1++].second,1);
        else if(t1[p1].first<t2[p2].first) v[++cnt]=pii(t1[p1++].second,1);
        else v[++cnt]=pii(t2[p2++].second,2);
    }
    for(int i=1,c=0;i<=cnt;i++)
    {
        if(v[i].second==1) res=max(res,c=max(c,v[i].first));
        if(v[i].second==2) res=max(res,c+v[i].first);
    }
    return res;
}

int main()
{
    static pii qwq[N];
    scanf("%d%d",&n,&online);
    for(int i=1,x;i<=n;i++)
    {
        scanf("%d",&x);
        qwq[i]=pii(x,-i);
    }
    sort(qwq+1,qwq+1+n);
    for(int i=1;i<=n;i++) a[-qwq[i].second]=i;
    divide();prework1();
    for(int i=1;i<=tot;i++) prework2(i);
    scanf("%d",&m);
    int l,r,ans=0;
    while(m--)
    {
        scanf("%d%d",&l,&r);
        if(online) l^=ans,r^=ans;
        printf("%d\n",ans=query(l,r));
    }
    return 0;
}
```

