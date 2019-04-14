# B. 恢复

### 题意简述

给定平面上n个点两两之间的距离。任意构造出n个点使得两两之间的距离恰为给定值

### 题解

前两个点显然任意取，只要距离符合要求即可。然后以两个点为圆心，分别以它们到第3个点的距离为半径画圆，显然会有两个交点，任取一个即可

现在前3个点确定了。对于第i个点，我们以前两个点为圆心，分别以它们到第i个点的距离为半径画圆，会有两个交点，根据第3个点到第i个点的距离判断选那个点

然后只要写一些基础的计算几何函数就行了，完全没必要写官解的2-SAT

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long double LD;
const int N=110;
LD dist[N][N];
int n;

struct Point
{
    LD x,y;
    Point(LD _x=0,LD _y=0):x(_x),y(_y){}
    Point operator - (const Point &a){return Point(x-a.x,y-a.y);}
    LD angle(){return atan2(y,x);}  //向量极角
    LD dis(Point &a){return sqrt((x-a.x)*(x-a.x)+(y-a.y)*(y-a.y));}
} p[N];
typedef pair<Point,Point> ppp;

struct Circle
{
    LD x,y,r;
    Circle(LD _x=0,LD _y=0,LD _r=0):x(_x),y(_y),r(_r){}
    Point getp(LD alpha){return Point(x+r*cos(alpha),y+r*sin(alpha));}  //已知圆上点的极角求坐标
};

LD cos_theo(LD a,LD b,LD c){return acos((b*b+c*c-a*a)/(2*b*c));}  //余弦定理
LD dis(LD x1,LD x2,LD y1,LD y2){return sqrt((x2-x1)*(x2-x1)+(y2-y1)*(y2-y1));}  //两点距离
bool dcmp(LD a,LD b){return fabs(a-b)<1e-5;}

ppp enjo_kosai(Circle &a,Circle &b)  // enjo kosai 意为”援交“，此处谐音做”圆交“
{
    LD beta=cos_theo(b.r,a.r,dis(a.x,b.x,a.y,b.y));
    Point vec(b.x-a.x,b.y-a.y);
    LD alpha=vec.angle()-beta;
    return ppp(a.getp(alpha),a.getp(alpha+beta*2));
}

int main()
{
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
        for(int j=1;j<=n;j++)
            scanf("%Lf",dist[i]+j);
    p[1]=Point(0.0,0.0);
    p[2]=Point(dist[1][2],0.0);
    Circle a(p[1].x,p[1].y,dist[1][3]);
    Circle b(p[2].x,p[2].y,dist[2][3]);
    p[3]=enjo_kosai(a,b).first;
    for(int i=4;i<=n;i++)
    {
        a=Circle(p[1].x,p[1].y,dist[1][i]);
        b=Circle(p[2].x,p[2].y,dist[2][i]);
        ppp c=enjo_kosai(a,b);
        Point p1=c.first,p2=c.second;
        p[i]=dcmp(p1.dis(p[3]),dist[3][i])?p1:p2; 
    }
    for(int i=1;i<=n;i++)
        printf("%.5Lf %.5Lf\n",p[i].x,p[i].y);
    return 0;
}
```