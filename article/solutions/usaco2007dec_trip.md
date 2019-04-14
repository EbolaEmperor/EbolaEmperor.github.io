[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/FracPro)

# 【USACO2007DEC】Sightseeing Cows 题解

分数规划模板题

考虑二分答案，假定有一个答案满足：

![](http://latex.codecogs.com/svg.latex?\frac{\sum&space;F(i)}{\sum&space;T(i)}\leq&space;ans)

其中，T(i)与F(i)是已经选定的集合。那么说明还有解比当前选定的集合更优秀

对上式做简单变换，得到

![](http://latex.codecogs.com/svg.latex?\sum(F(i)-ans*T(i))\leq&space;0)

假如这个式子恒成立，则原式成立，说明有更优解，那么二分下界就可以往上调

判定需要修改原图边权，将点权加到边权里面

具体地，设边e从u连向v，那么判定时，设这条边的边权为F[v]-ans*capa[e]，其中capa[e]是一开始输入的边权

我们知道，上式恒成立的条件是，当左边取得最大值时不等式成立。那么只需判断修改后的图是否存在正权环即可

判正权环真别扭，那就把边权取相反数判负权环吧，用DFS-SPFA即可

```cpp
#include<bits/stdc++.h>
using namespace std;

struct Edge{int to,next;double capa;} e[5010];
int h[1010],sum=0,n,m;
double cap[5010],d[1010],w[1010];
bool flag,vis[1010];

void add_edge(int u,int v,double w)
{
	e[++sum].to=v;
	cap[sum]=w;
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
		e[i].capa=x*cap[i]-w[e[i].to];
	memset(vis,0,sizeof(vis));
	memset(d,0,sizeof(d));
	flag=0;
	for(int i=1;i<=n;i++)
		if(!flag) dfs(i);
	return flag;
}

int main()
{
	int u,v;double c;
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;i++) scanf("%lf",w+i);
	for(int i=1;i<=m;i++)
	{
		scanf("%d%d%lf",&u,&v,&c);
		add_edge(u,v,c);
	}
	double l=0,r=1e3,mid;
	while(r-l>1e-3)
	{
		//printf("%lf %lf\n",l,r);
		mid=(l+r)/2;
		if(chk(mid)) l=mid;
		else r=mid;
	}
	printf("%.2lf\n",l);
	return 0;
}
```
