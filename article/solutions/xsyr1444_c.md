# C. 耐心

### 题面

![](http://www.ebola.pro/images/xsyr1444_c_1.png)

### 题解

如果可以的话，我们当然希望能⌈rc/d⌉天就解决这个问题，事实证明是可以的

可以只留一个\(r%d+d,c%d+d\)的矩形，因为其它的区域都可以用连续的d格完美解决。然后我们对这个矩形来进行操作。为方便表述，设这个新矩形的大小为n\*m

假如我们每行都从最左端起填上连续的d格，那么剩下还需要的天数就是⌈n(m-d)/d⌉，但这些格子似乎并不能用这么多步解决，因为要求每天搞的区域必须是同一行或同一列。下面画一个n=m=7,d=5的情况

![](http://www.ebola.pro/images/xsyr1444_c_2.png)

但我们会发现，由于我们行数与列数的特殊性，一个数字最多只会出现在相邻的两列，那么我们不妨把这些数字往右推一推。在这个例子中，我们可以把3往右推一列，空出新的位置就把第一列的2推到第二列去

![](http://www.ebola.pro/images/xsyr1444_c_3.png)

因为我们只是平行推动，所以并没有影响每行的剩余空位数量，每行仍然有d个空位，这样就可以每行一个数字把剩余空位填满了

按上述方法做，可以满足第一部分的答案要求，实测55分

那要满足第二部分答案要求，我们就要让剩余的空位尽可能连续。不难发现每一列的数字位置在模n意义下是连续的，所以我们可以记录每一列数字的起始位置，然后将有数字的列按起始位置排序，这样就保证了剩余空位是尽可能连续的。仔细思考甚至能发现，剩余空位最多被分成连续的两段，所以保证了每天的连续段数至多两段。在每天都连续取不到天数下界时，这个做法显然是最好的选择

代码里的第一个solve是用来填连续d格的，比较丑见谅

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=410;
int ans[N][N],pos[N];
int T,r,c,d,cnt;

void solve(int xl,int xr,int yl,int yr)
{
    int lenx=xr-xl+1,leny=yr-yl+1;
    if(lenx<=0||leny<=0) return;
    if(lenx>=d)
    {
        for(int i=yl;i<=yr;i++)
        {
            cnt++;
            for(int j=xl;j<xl+d;j++)
                ans[j][i]=cnt;
        }
        solve(xl+d,xr,yl,yr);
    }
    else if(leny>=d)
    {
        for(int i=xl;i<=xr;i++)
        {
            cnt++;
            for(int j=yl;j<yl+d;j++)
                ans[i][j]=cnt;
        }
        solve(xl,xr,yl+d,yr);
    }
    else
    {
        if(lenx<leny)
        {
            for(int i=xl;i<=xr;i++)
            {
                cnt++;
                for(int j=yl;j<=yr;j++)
                    ans[i][j]=cnt;
            }
        }
        else
        {
            for(int j=yl;j<=yr;j++)
            {
                cnt++;
                for(int i=xl;i<=xr;i++)
                    ans[i][j]=cnt;
            }
        }
    }
}

void solve(int n,int m)
{
    int k=(n*(m-d)-1)/d+1;
    pos[1]=1;
    for(int i=2;i<=k;i++)
        pos[i]=(pos[i-1]+d-1)%n+1;
    sort(pos+1,pos+1+k);
    for(int i=1;i<=k;i++)
    {
        cnt++;
        for(int j=0;j<d;j++)
            ans[(pos[i]+j+n-1)%n+1][i]=cnt;
    }
    for(int i=1;i<=n;i++)
    {
        cnt++;
        for(int j=1;j<=m;j++)
            if(!ans[i][j]) ans[i][j]=cnt;
    }
}

void gao()
{
    cnt=0;solve(1,r,1,c);
    if(r>=d&&c>=d&&cnt>(r*c-1)/d+1)
    {
        int n=r%d+d,m=c%d+d;cnt=0;
        memset(ans,0,sizeof(ans));
        solve(n+1,r,1,c);
        solve(1,n,m+1,c);
        solve(n,m);
    }
}

int main()
{
    for(scanf("%d",&T);T;T--)
    {
        scanf("%d%d%d",&r,&c,&d);gao();
        for(int i=1;i<=r;i++,puts(""))
            for(int j=1;j<=c;j++)
                printf("%d ",ans[i][j]);
    }
    return 0;
}
```