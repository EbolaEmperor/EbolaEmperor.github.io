# B. 专家系统

### 题面

> 现在有n(n≤1e5)个坐标(xi, yi)，你要从中选出k个。
> 
> 假设你选出的全部k个坐标中，x坐标最小值为xmin，x坐标最大值为xmax，y坐标最小值为ymin，y坐标最大值为ymax。
> 
> 那么你得出的不确定度c=max(xmax-xmin, ymax-ymin)
> 你的目的是让c尽可能小。
> 
> 由于可能有很多种选法能够达成这一目的，你只需要输出这个最小的c即可。

### 题解

二分答案显然。二分之后，问题变成了能否找到一个边长为mid的矩形，使得矩形内部至少包含k个给定点

考虑扫描线。从左往右扫描矩形左侧边界x，然后利用双指针维护，使得当前只处理横坐标在范围[x,x+mid]中的点。然后我们要求一个长为mid的纵坐标区间，使得在当前的处理范围内，它包含最多的点。

对于一个纵坐标为y的点，他会对长为mid、左端点为y-mid到y的纵坐标区间造成1的贡献。所以可以用线段树来维护，线段树下标为k的叶子节点含义是纵坐标区间[k,k+mid]。支持区间加法、查询整体最大值即可

然后就做完了，复杂度两个log

```cpp
#include<bits/stdc++.h>
using namespace std;

const int S=(1<<23)+5;
char buf[S],*H,*T;
inline char Get()
{
    if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
    if(H==T) return -1;return *H++;
}
inline int read()
{
    int x=0,fg=1;char c=Get();
    while(!isdigit(c)&&c!='-') c=Get();
    if(c=='-') fg=-1,c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x*fg;
}

typedef long long LL;
const int N=100010;
struct Point{LL x,y;} p[N];
int mx[N*3],tag[N*3];
int n,k,my,lft[N];
LL hy[N];

bool operator < (const Point &a,const Point &b){return a.x<b.x;}

void add(int o,int l,int r,int nl,int nr,int x)
{
    if(l>=nl&&r<=nr)
    {
        mx[o]+=x;
        tag[o]+=x;
        return;
    }
    int mid=(l+r)/2;
    if(nl<=mid) add(o<<1,l,mid,nl,nr,x);
    if(nr>mid) add(o<<1|1,mid+1,r,nl,nr,x);
    mx[o]=max(mx[o<<1],mx[o<<1|1])+tag[o];
}

bool check(LL d)
{
    memset(mx,0,sizeof(mx));
    memset(tag,0,sizeof(tag));
    for(int i=1,l=1;i<=my;i++)
    {
        while(hy[i]-hy[l]>d) l++;
        lft[i]=l;
    }
    for(int i=1,l=1,r=0;i<=n;i++)
    {
        while(r<n&&p[r+1].x-p[i].x<=d) r++,add(1,1,my,lft[p[r].y],p[r].y,1);
        while(l<r&&p[l].x<p[i].x) add(1,1,my,lft[p[l].y],p[l].y,-1),l++;
        if(mx[1]>=k) return 1;
        if(r==n) return 0;
    }
    return 0;
}

int main()
{
    n=read();k=read();
    for(int i=1;i<=n;i++)
    {
        p[i].x=read();
        p[i].y=read();
        hy[i]=p[i].y;
    }
    sort(hy+1,hy+1+n);
    my=unique(hy+1,hy+1+n)-(hy+1);
    for(int i=1;i<=n;i++) p[i].y=lower_bound(hy+1,hy+1+my,p[i].y)-hy;
    sort(p+1,p+1+n);
    LL l=0,r=4e9,mid;
    while(l<=r)
    {
        mid=(l+r)/2;
        if(check(mid)) r=mid-1;
        else l=mid+1;
    }
    printf("%lld\n",l);
    return 0;
}
```