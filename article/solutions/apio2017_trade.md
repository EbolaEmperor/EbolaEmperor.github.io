[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/FracPro)

# 【APIO2017】商旅 题解

发现点数很少，这就允许了我们随便乱搞

首先，对于任意点对(u,v)，我们可以预处理出u点买入v点卖出的最大盈利，以及u到v的最短距离，这个过程可以用Floyd来完成

处理出了这个就非常好办了，建一张新的图，对于所有可以获得盈利的点对(u,v)，我们从u向v连一条边，保存u到v的最大盈利以及最短距离

题目要求我们就是要最大化 Σ(盈利)/Σ(距离)，我们使用分数规划

为了表述方便，下面记盈利为w，距离为d

考虑二分答案，假设有一个答案ans，它满足：

![](http://latex.codecogs.com/svg.latex?\frac{\sum&space;w_i}{\sum&space;d_i}\leq&space;ans)

对柿子稍作变换，得到：

![](http://latex.codecogs.com/svg.latex?\sum(w_i-ans*d_i)\leq0)

若左式取得最大值时不等式成立，则不等式恒成立，说明存在更优解，需要调整二分下界

那么我们可以调整图中的边权，将边权设为w-ans*d，然后判定正权环即可

同样的套路，边权取相反数变成判负环，然后用DFS-SPFA来做

```cpp
#include<bits/stdc++.h>
#define double long double
#define INF 1e11
using namespace std;

const double eps=5*1e-8;
struct Edge{int to,next;double capa;} e[20010];
int h[110],sum=0,n,m,K;
double dis[110][110];
double buy[110][1010],sell[110][1010];
double cap[20010],val[20010],earn[110][110];
double d[110];bool vis[110],flag;

void add_edge(int u,int v)
{
	e[++sum].to=v;
	cap[sum]=dis[u][v];
	val[sum]=earn[u][v];
	e[sum].next=h[u];
	h[u]=sum;
}

void dfs(int u)
{
	if(flag) return;
	vis[u]=1;
	for(int tmp=h[u];tmp;tmp=e[tmp].next)
		if(d[u]+e[tmp].capa<d[e[tmp].to])
		{
			if(vis[e[tmp].to]||flag){flag=1;return;}
			d[e[tmp].to]=d[u]+e[tmp].capa;
			dfs(e[tmp].to);
		}
	vis[u]=0;
}

bool chk(double x)
{
	for(int i=1;i<=sum;i++)
		e[i].capa=x*cap[i]-val[i];
	memset(vis,0,sizeof(vis));
	memset(d,0,sizeof(d));
	flag=0;
	for(int i=1;i<=n;i++)
		if(!flag) dfs(i);
	return flag;
}

int main()
{
	int u,v;double w;
	scanf("%d%d%d",&n,&m,&K);
	for(int i=1;i<=n;i++)
		for(int j=1;j<=K;j++)
			scanf("%Lf%Lf",&buy[i][j],&sell[i][j]);
	for(int i=1;i<=n;i++)
		for(int j=1;j<=n;j++)
			dis[i][j]=(i==j)?0:INF;
	for(int i=1;i<=m;i++)
	{
		scanf("%d%d%Lf",&u,&v,&w);
		dis[u][v]=min(dis[u][v],w);
	}
	for(int k=1;k<=n;k++)
		for(int i=1;i<=n;i++)
			for(int j=1;j<=n;j++)
				dis[i][j]=min(dis[i][j],dis[i][k]+dis[k][j]);
	for(int i=1;i<=n;i++)
		for(int j=1;j<=n;j++)
			if(i!=j&&dis[i][j]<INF-eps)
			{
				for(int k=1;k<=K;k++)
					if(fabs(buy[i][k]+1)>eps&&fabs(sell[j][k]+1)>eps)
						earn[i][j]=max(earn[i][j],sell[j][k]-buy[i][k]);
				add_edge(i,j);
			}
	double l=0,r=1e9,mid;
	while(r-l>eps)
	{
		mid=(l+r)/2;
		if(chk(mid)) l=mid;
		else r=mid;
	}
	printf("%d\n",(int)floor(r));
	return 0;
}
```
