# 几类最短路算法的介绍与比较

[返回首页](http://www.ebola.pro)
[返回专题](http://www.ebola.pro/special/ShortestPath)

### Floyd算法

通常认为这是最暴力的最短路算法，但它也有它的优势。它可以在O(n^3)的复杂度内求出任意两点间的最短路径，并且算法常数非常小。

先贴一下算法的基本框架(领接矩阵存图)：

```cpp
for(int k=1;k<=n;k++)
	for(int i=1;i<=n;i++)
		for(int j=1;j<=n;j++)
			upmin(dis[i][j],dis[i][k]+dis[k][j]);
```

代码中的dis数组初始值设为正无穷，如果两点间存在边，则设为两点间最短边的长度

基本思想就是，假如i到k再到j的路径长度比当前i到j的最短路径更短，那么就有了一条i到j的更短路径，此时更新i到j的最短路径

算法很好理解，也很容易证明这样求出的路径就是最短路径

### SPFA算法

[模板题链接](https://www.luogu.org/problemnew/show/P3371)

SPFA算法的全程是Shortest Path Faster Algorithm，这个名字听起来就很快的样子。其实这个算法是Bellman-Ford算法的队列优化，国外早就已经出现了，被段凡丁教授引入了国内，并且起了这个土鳖的名字。这位教授通过实验证明，SPFA的复杂度是O(kn)，其中k是个常数，不会超过2。事实真的如此吗？

如果数据随机，该教授的推论是正确的。但是假如使用特殊数据，SPFA的复杂度会退化到和Bellman-Ford一样，也就是O(nm)。因此SPFA是一种非常不稳定的算法，在近几年的比赛中经常被出题人卡。在8102 ION 比赛中，我的暴力使用了SPFA，然后就被卡掉了5分。需要记住的一句话就是：“关于SPFA？它死了。”

先贴出SPFA的算法框架(链式前向星存图)：

```cpp
queue<int> q;
q.push(s);
memset(vis,0,sizeof(vis));
for(int i=1;i<=n;i++) dis[i]=0x7fffffff;
vis[s]=1;dis[s]=0;
while(!q.empty())
{
	int o=q.front();
	for(int tmp=h[o];tmp;tmp=e[tmp].next)
		if(dis[o]+e[tmp].capa<dis[e[tmp].to])
		{
			dis[e[tmp].to]=dis[o]+e[tmp].capa;
			if(!vis[e[tmp].to]) q.push(e[tmp].to),vis[e[tmp].to]=1;
		}
	q.pop();
	vis[o]=0;
}
```

SPFA算法基于“松弛”这一基本操作

其基本思想就是，遍历当前点的所有出边，假如当前点的距离加上这条边的边权比出边指向点的距离小，就对指向点进行松弛操作，即更新指向点的距离，并使其进入待处理队列。用一个布尔数组来记录所有点的入队情况，避免重复的入队操作。

它与Floyd算法不同，它是一类“单源最短路径”算法，只能处理出某个点到其它所有点的最短路，并不能处理出任意两点的最短路径。尽管该算法复杂度玄学，使用该算法时仍应谨慎，应该把时间复杂度计算为O(nm)

### Dijkstra算法

[模板题链接](https://www.luogu.org/problemnew/show/P4779)

这是由Dijkstra发明的一个算法。它也是一类“单源最短路径”算法，但是与SPFA不同，这个算法并不能解决有负权边的情况。因此适用面比SPFA小。

这个算法一开始是O(m^2)的，它非常的不优秀。但是它有优化空间，后人用“堆”这一数据结构对它进行优化，使它的时间复杂度降到了O(m log m)。值得一提的是，它的时间复杂度的稳定的，并不会因为图的特殊性而退化，这一点会在后面证明

先将算法框架贴出(领接表存图)：

```cpp
memset(vis,0,sizeof(vis));
for(int i=1;i<=n;i++) dis[i]=INF;
priority_queue<Node> q;
q.push(Node(s,0));dis[s]=0;
while(!q.empty())
{
	Node tmp=q.top();q.pop();
	int u=tmp.v,d=tmp.w;
	if(vis[u]) continue;
	vis[u]=1;
	for(int i=0;i<g[u].size();i++)
		if(d+g[u][i].w<dis[g[u][i].v])
		{
			dis[g[u][i].v]=d+g[u][i].w;
			q.push(Node(g[u][i].v,dis[g[u][i].v]));
		}
}
```

代码中的Node是自己定义的一个二元组，并且定义了比较的优先级。定义优先级就是定义小于号，但是由于STL的优先队列是使用大于号比较，所以你定义小于号时要与你的内心意愿正好相反才行

下面是代码中Node的结构体定义：

```cpp
struct Node
{
	int v,w;
	Node(int a=0,int b=0):v(a),w(b){}
	friend bool operator < (const Node &a,const Node &b){return a.w>b.w||a.w==b.w&&a.v<b.v;}
};
```

这个Node可以两用，存边时可以表示一条边，其中v表示指向点，w表示边权；在堆中则表示一种情况，其中v表示点编号，w表示这个情况下v的距离

算法的主要思想是：每次选择距离最小的那个二元组，然后假如那个二元组的点没有处理过，就从那个二元组的距离出发，处理所有出边，并将必要的边入堆。解释的不是很清楚，建议自己结合代码理解一下算法的过程

复杂度很好证明。因为每个点只会被处理一次，那么每个点的所有出边也只会被遍历一次，因此每条边至多入一次堆。入堆的时间复杂度是O(log m)，如果每条边都入堆，那就是O(m log m)
