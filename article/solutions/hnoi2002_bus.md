[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Matrix)

# 【HNOI2002】公交车路线 题解

阅读了一下题面，忽然就笑了出来，这不是就邻接矩阵的幂的模板题吗！

至于邻接矩阵的幂是什么，以后有空再说。过段时间在我们学校的交流活动中我会讲这个玩意儿。

我的代码毫无压力，甚至还想让出题人把n的规模扩大到2^60

嗯，都说了是邻接矩阵的幂了。所以，把题目上那幅图用邻接矩阵建出来，注意点E不要连出边，因为题目上说了主人公没那么蠢，不会到了E还继续走。

然后，计算这个邻接矩阵的n次幂就好了，最终答案在g[A][E]中

时间复杂度：非常非常轻松的O(log n)

```cpp
#include<bits/stdc++.h>
#define Mod 1000
using namespace std;

struct Matrix
{
	int m,n;
	int a[15][15];
	void init(int x,int y){m=x;n=y;memset(a,0,sizeof(a));}
};

Matrix operator * (const Matrix &a,const Matrix &b)
{
	Matrix ans;
	ans.m=a.m;
	ans.n=b.n;
	for(int i=0;i<a.m;i++)
		for(int j=0;j<b.n;j++)
		{
			ans.a[i][j]=0;
			for(int k=0;k<b.m;k++) ans.a[i][j]=(ans.a[i][j]+a.a[i][k]*b.a[k][j]%Mod)%Mod;
		}
	return ans;
}

Matrix QuickPow(Matrix &a,int b)
{
	Matrix ans;
	ans.init(a.m,a.n);
	for(int i=0;i<ans.m;i++) ans.a[i][i]=1;
	while(b)
	{
		if(b&1) ans=ans*a;
		a=a*a;
		b>>=1;
	}
	return ans;
}

int main()
{
	Matrix G;
	G.init(8,8);
	for(int i=0;i<7;i++)
		G.a[i][i+1]=G.a[i+1][i]=1;
	G.a[0][7]=G.a[7][0]=1;
	G.a[4][3]=G.a[4][5]=0;
	int n;
	cin>>n;
	G=QuickPow(G,n);
	cout<<G.a[0][4]<<endl;
	return 0;
}
```
