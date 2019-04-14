# B. 我的画风不一样

题面出锅，重造数据，难度降低，舒服！

设f\[r\]\[b\].first/second表示剩余r个红球，b个蓝球时，先手/后手的胜率，然后记忆化搜索

对于每一步，枚举选几个球，计算出选这么多球时的胜率，first应该由下一层搜索的second转移过来，second类似，然后取一个最优值（first尽可能大，second尽可能小）

枚举出选几个球之后，要枚举取了多少个红球，然后算出这种情况的概率。若当前有R个红球、B个蓝球，要取r个红球、b个蓝球，则概率为：C(R,r) \* C(B,b) / C(R+B,r+b)

然后对于一开始的m，我们就枚举一开始取了多少红球，然后按上述方法算出概率，再乘上记忆化搜索的结果，贡献相加即可

一开始挺担心C(200,99)会存不下的，然后写了手动约分。赛后被告知浮点数可以保存前几位有效数字，然后再除一下，差不多精度误差就控制在1e-6以内了，非常奇妙

```cpp
#include<bits/stdc++.h>
#define FR first
#define SE second
using namespace std;

typedef pair<double,double> pdd;
pdd f[110][110];
bool vis[110][110];
double C[210][210];
int n,m,R,B;

void Init()
{
    for(int i=0;i<=200;i++)
    {
        C[i][0]=1;
        for(int j=1;j<=i;j++)
            C[i][j]=C[i-1][j]+C[i-1][j-1];
    }
}

pdd dp(int r,int b)
{
    pdd &res=f[r][b];
    if(vis[r][b]) return res;
    vis[r][b]=1;res=pdd(0.0,1.0);
    if(r==0) return res=pdd(1.0,0.0);
    for(int num=1;num<=min(n,r+b);num++)
    {
        pdd tans(0,0);
        for(int i=max(0,num-b);i<=min(num,r);i++)
        {
            pdd tmp=dp(r-i,b-(num-i));
            double prb=C[r][i]/C[r+b][num]*C[b][num-i];
            tans.FR+=prb*tmp.SE;
            tans.SE+=prb*tmp.FR;
        }
        if(tans.FR<res.FR) continue;
        if(tans.FR>res.FR) res=tans;
        else if(tans.SE<res.SE) res=tans;
    }
    return res;
}

int main()
{
    double ans=0;Init();
    cin>>R>>B>>n>>m;
    for(int i=0;i<=m;i++)
    {
        pdd tmp=dp(R-i,B-(m-i));
        double prb=C[R][i]/C[R+B][m]*C[B][m-i];
        ans+=prb*tmp.FR;
    }
    printf("%.7lf\n",ans);
    return 0;
}
```

