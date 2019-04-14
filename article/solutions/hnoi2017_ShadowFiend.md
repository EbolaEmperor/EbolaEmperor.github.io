# 【HNOI2017】影魔

题目链接：[4826. 影魔  -  BZOJ](https://www.lydsy.com/JudgeOnline/problem.php?id=4826)

对每个位置p求出它后面第一个比它大的数，记作bigger\[p\]，单调栈实现即可

这样，对于每个点，我们就求出了以它为左端点的区间中，有哪些区间的最大值是它，我们假定这样的区间产生的贡献都是p2，当然实际上某些区间的贡献应该是p1，先不管它

发现我们这样做，没法分清哪些区间的贡献是p1。但我们可以知道，每个位置为区间左端点且为区间次大值时，区间右端点是哪个。我们还需要求出每个位置为区间右端点且为区间次大值，区间左端点是哪个，这样就可以完全统计出贡献为p1的区间。发现只要把序列和询问一并反转一下再做一样的事，就可以得到我们要的东西

贡献为p1的区间，之前被我们误算为p2了，而且正反各误算了一遍。但我们总共只发现了这个区间一遍，因此我们可以用某种方式，使得这个区间的贡献最后加上p1-2*p2

于是就得出了算法框架：扫描线，维护一个数据结构，支持区间加、区间求和。对于每个位置p，让区间\[p+1, bigger\[p\]\]加上p2，然后将位置bigger\[p\]加上p1-2*p2。从后往前扫，每扫到一个位置p，就求出所有以p为左端点的询问的答案，答案就是对询问区间求和

```cpp
#include<bits/stdc++.h>
using namespace std;

namespace IO
{
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
    char obuf[S],*oS=obuf,*oT=oS+S-1,c,qu[55];int qr;
    inline void flush(){fwrite(obuf,1,oS-obuf,stdout);oS=obuf;}
    inline void putc(char x){*oS++ =x;if(oS==oT) flush();}
    template <class I>inline void print(I x)
    {
        if(!x) putc('0');
        while(x) qu[++qr]=x%10+'0',x/=10;
        while(qr) putc(qu[qr--]);
        putc('\n');
    }
}

using namespace IO;
typedef long long LL;
const int N=200010;
LL sum[N<<2],tag[N<<2];
int n,m,p1,p2,val[N],bgr[N];
struct QRY{int l,r,id;} q[N];
vector<QRY> L[N];
stack<int> s;
LL ans[N];

void Add(int o,int l,int r,int nl,int nr,LL x)
{
    if(nl>nr) return;
    if(l>=nl&&r<=nr)
    {
        sum[o]+=x*(r-l+1);
        tag[o]+=x;
        return;
    }
    int mid=(l+r)/2;
    if(nl<=mid) Add(o<<1,l,mid,nl,nr,x);
    if(nr>mid) Add(o<<1|1,mid+1,r,nl,nr,x);
    sum[o]=sum[o<<1]+sum[o<<1|1]+tag[o]*(r-l+1);
}

LL Qsum(int o,int l,int r,int nl,int nr)
{
    if(l==nl&&r==nr) return sum[o];
    int mid=(l+r)/2;LL res=tag[o]*(nr-nl+1);
    if(nr<=mid) res+=Qsum(o<<1,l,mid,nl,nr);
    else if(nl>mid) res+=Qsum(o<<1|1,mid+1,r,nl,nr);
    else res+=Qsum(o<<1,l,mid,nl,mid)+Qsum(o<<1|1,mid+1,r,mid+1,nr);
    return res;
}

void solve()
{
    memset(sum,0,sizeof(sum));
    memset(tag,0,sizeof(tag));
    for(int i=1;i<=n;i++) L[i].clear();
    for(int i=1;i<=m;i++) L[q[i].l].push_back(q[i]);
    while(!s.empty()) s.pop();
    val[n+1]=n+1;s.push(n+1);
    for(int i=n;i>=1;i--)
    {
        while(val[s.top()]<val[i]) s.pop();
        bgr[i]=s.top();s.push(i);
    }
    for(int i=n;i>=1;i--)
    {
        Add(1,1,n,i+1,min(bgr[i],n),p2);
        if(bgr[i]<=n) Add(1,1,n,bgr[i],bgr[i],p1-2*p2);
        for(auto x : L[i]) ans[x.id]+=Qsum(1,1,n,x.l,x.r);
    }
}

int main()
{
    n=read();m=read();p1=read();p2=read();
    for(int i=1;i<=n;i++) val[i]=read();
    for(int i=1;i<=m;i++)
    {
        q[i].l=read();
        q[i].r=read();
        q[i].id=i;
    }
    solve();
    for(int i=1;i<=m;i++)
    {
        q[i].l=n-q[i].l+1;
        q[i].r=n-q[i].r+1;
        swap(q[i].l,q[i].r);
    }
    reverse(val+1,val+1+n);solve();
    for(int i=1;i<=m;i++) print(ans[i]);
    flush();
    return 0;
}
```

