# A. 踢罐子

### 题面


> 平面上有n(n≤1000)个点，其中任意2点不重合，任意3点不共线。
> 
> 我们等概率地选取一个点A，再在剩下的n-1个点中等概率地选取一个点B，再在剩下的n-2个点中等概率地选取一个点C。
>
> 然后我们计算伤害倍率d。作ABC外接圆，每一个位于弧BC和线段BC之间的点计1倍，每一个位于弧BC上的点(包括B,C两点)计1/2倍，特别的，点A计1倍。将这些倍率全部加起来得到伤害倍率d。
>
> 注意：弧BC是指，ABC外接圆上B到C而不包含点A的部分，这个弧不一定是劣弧。
>
> 给定这n个点的坐标，你需要求出d的期望。为了简单起见，你只需要输出d\*n\*(n-1)\*(n-2)\*2的值，可以看出这是一个整数。
> 
> 数据保证不存在三点共线

### 题解

把A、B、C的贡献单独考虑，显然是A(n,3)。然后考虑其它点的贡献

考虑一个凸四边形，它对答案的贡献不论如何都是4（分四点共圆和其他情况随便讨论一下即可）。而一个凹四边形对答案的贡献不论如何都是0

于是问题变成了凸四边形计数

考虑在一个凸四边形中任取两个点连一条直线，有4种取法使得另外两个点在直线同侧，2种取法使得另外两个点在直线异侧；在一个凹四边形中任取两个点连一条直线，有3种取法使得另外两个点在直线同侧，3种取法使得另外两个点在直线异侧

设a表示凸四边形数量，b表示凹四边形数量，x表示任取两个点画直线再任取两个异侧点的方案数，x表示任取两个点画直线再任取两个同侧点的方案数

于是，有：2a+3b=x, 4a+3b=y。解得：a=(y-x)/2

枚举画线的两个点，然后统计x、y即可

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=1010;

struct Point
{
    LL x,y;double ang;
    Point(LL _x=0,LL _y=0):x(_x),y(_y){}
    void init(){ang=atan2(y,x);}
    Point operator - (const Point &a){return Point(x-a.x,y-a.y);}
    double operator * (const Point &a){return 1.0*x*a.y-1.0*y*a.x;}
    bool operator < (const Point &a) const{return ang<a.ang;}
} p[N],pp[N];
LL ans=0,x=0,y=0,n;

int nxt(int x){if(++x==n) x=1;return x;}
void gao()
{
    for(int i=1;i<n;i++) pp[i]=p[i]-p[n],pp[i].init();
    sort(pp+1,pp+n);
    for(int i=1,q=1,cnt=0;i<n;i++,cnt--)
    {
        if(cnt<0) cnt=0,q=i;
        while(nxt(q)!=i&&pp[i]*pp[nxt(q)]>=0) q=nxt(q),cnt++;
        x+=1ll*cnt*(n-cnt-2);
        y+=1ll*cnt*(cnt-1)/2+(n-cnt-2)*(n-cnt-3)/2;
    }
}

int main()
{
    scanf("%lld",&n);
    for(int i=1;i<=n;i++) scanf("%lld%lld",&p[i].x,&p[i].y);
    for(int i=1;i<=n;i++) swap(p[i],p[n]),gao(),swap(p[i],p[n]);
    ans=4ll*(y-x)/2+4ll*n*(n-1)*(n-2);
    printf("%lld\n",ans);
    return 0;
}
```