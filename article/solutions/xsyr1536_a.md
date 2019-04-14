# A. 一道带权带花树好题

### 题意简述

给定一个带点权的无向图，定义一个匹配的权值为所有参与匹配的点权之和，求最大权匹配。点数范围500

### 题解

如果把边权视作它所连两点的点权之和，就变成了一般图最大权匹配裸题，直接上UOJ去找模板就行了。然而我既不会带花树也不会苏凯树qwq

于是乎，二分图特判一下，其它情况模拟退火，得到70分好成绩qwq

好的不扯皮了，我们来说正解

有一个叫做“tutte矩阵”的东西，可以去[这里](http://www.cnblogs.com/zhujiangning/p/7157465.html)看一下

然后最大权匹配，就将每条边视作一个未知数，并给它们赋上随机值，反边的值是正边的相反数。然后将得到矩阵的第x行行视作点x对应的n维向量，然后维护一个线性基。按点权从大到小将对应的n维向量插入线性基，能插入就加上它的点权

至于为什么是对的……我不知道qwq

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef pair<int,int> pii;
const int ha=19260817;
const int N=510;
int a[N][N],b[N],n,m;
pii p[N];

void gao()
{
    for(int i=1;i<=n;i++)
        for(int j=1;j<i;j++)
        {
            if(a[i][j]>=0) continue;
            a[i][j]=rand()%ha;
            a[j][i]=ha-a[i][j];
        }
}

int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}

bool insert(int x)
{
    for(int i=1;i<=n;i++)
    {
        if(!a[x][i]) continue;
        if(!b[i]){b[i]=x;return 1;}
        int y=b[i],k=1ll*a[x][i]*Pow(a[y][i],ha-2)%ha;
        for(int j=1;j<=n;j++)
            a[x][j]=(a[x][j]-1ll*k*a[y][j]%ha+ha)%ha;
    }
    return 0;
}

int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++)
        scanf("%d",&p[i].first),p[i].second=i;
    sort(p+1,p+1+n);
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        a[u][v]=a[v][u]=-1;
    }
    gao();
    long long ans=0;
    for(int i=n;i>=1;i--)
        if(insert(p[i].second))
            ans+=p[i].first;
    printf("%lld\n",ans);
    return 0;
}
```