# 【JXOI2017】加法

题目链接：[2274. 加法  -  LibreOJ](https://loj.ac/problem/2274)

emmm……我参加了JXOI2016和JXOI2018，就是没参加2017（滚去中考了），真奇妙

考虑二分答案。二分答案之后，问题就变成了：选择不超过k个区间来操作，使得所有数都不小于mid

于是我们可以将所有区间按左端点排序，然后从左到右扫描，扫到一个点，就将左端点位于该点左侧的线段全部加入一个容器中。若该点小于mid，我们就需要在容器中拿一个区间来操作，这个点前面的所有点已经大于等于mid了，所以我们的目的是影响后面尽可能多的点，因此右端点更靠右的区间是更优的。

这样其实就确定了我们要使用的容器是堆，是一个以右端点为关键字的大根堆

若某一个数小于mid，然而堆顶元素的右端点都在mid左边了，显然check就失败了

现在我们还需要一个能支持区间加法、单点查询的数据结构，那就是树状数组。再统计一下区间加法的次数，超过k次就返回false

```cpp
#include<bits/stdc++.h>
#define FR first
#define SE second
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

typedef pair<int,int> pii;
typedef long long LL;
const int N=200010;
pii seg[N];
int n,m,k,a;
int A[N];
LL bit[N];
priority_queue<pii> pq;

inline int lowbit(const int &x){return x&-x;}
void add(int p,int x){for(;p<=n+1;p+=lowbit(p))bit[p]+=x;}
LL qry(int p){LL res=0;for(;p;p-=lowbit(p))res+=bit[p];return res;}

bool check(int x)
{
    memset(bit,0,sizeof(LL)*(n+3));
    for(int i=1;i<=n;i++)
        add(i,A[i]),add(i+1,-A[i]);
    int p=1,ans=0;
    while(!pq.empty()) pq.pop();
    for(int i=1;i<=n;i++)
    {
        while(p<=m&&seg[p].FR<=i) pq.push(pii(seg[p].SE,seg[p].FR)),p++;
        while(qry(i)<x)
        {
            if(pq.empty()) return 0;
            pii t=pq.top();pq.pop();
            if(t.FR<i) return 0;
            add(t.SE,a);add(t.FR+1,-a);
            if(++ans>k) return 0;
        }
    }
    return 1;
}

int main()
{
    for(int T_T=read();T_T;T_T--)
    {
        int l=INT_MAX;
        n=read();m=read();k=read();a=read();
        for(int i=1;i<=n;i++) l=min(l,A[i]=read());
        for(int i=1;i<=m;i++) seg[i].FR=read(),seg[i].SE=read();
        sort(seg+1,seg+1+m);
        int r=l+k*a,mid;
        while(l<=r)
        {
            mid=(l+r)/2;
            if(check(mid)) l=mid+1;
            else r=mid-1;
        }
        printf("%d\n",check(l)?l:l-1);
    }
    return 0;
}
```

