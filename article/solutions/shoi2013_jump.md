[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Matrix)

# 【SHOI2013】超级跳马 题解

设F[i,j]表示第i行第j列的方案数，可以得到F[i,j]=Σ{F[i,j-2k+1]+F[i-1,j-2k+1]+F[i+1,j-2k+1]}(k=1~j/2)

如果枚举i，j，并按上式更新答案，那么时间复杂度O(nm²)，可以拿10分

发现可以利用前缀和优化这个式子。

设S[i,j]表示Σ{F[i,j-2k+1]}{k=1~j/2}，那么可以推出S[i,j]=S[i,j-2]+S[i,j-1]+S[i-1,j-1]+S[i+1,j-1]

枚举i，j，按上式更新答案，时间复杂度O(nm)，可以拿50分

发现数据范围非常诡异，n很小，m很大

我们将S数组画出图示，看一下对S[i,j]产生贡献的点有什么分布规律

![](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fsdvaln3ndj30sb0n5mx4.jpg)

发现对S[i,j]有影响的位置都在[i,j]的左边两列，假如我们把每两列设计为一个状态，不就可以用矩阵乘法来往右递推了吗

以n=4为例，S的矩阵递推表达式如下：

![](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fsdvl0eh74j31240ofq4f.jpg)

这样就可以矩阵快速幂往右推，时间复杂度O(n³log m)

但是当n不同的时候，我们总不可能手动打表转移矩阵吧，我们应该发现这个矩阵的普遍规律

作两条辅助线，将矩阵划分成4块：

![](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fsdvo2vexbj31290v93zj.jpg)

发现左上块全是0，右上、左下块的左上-右下对角线为1，右下块则是左上-右下对角线及其左右两格均为1

利用这个规律就可以构造出更大的矩阵了

```cpp
#include<bits/stdc++.h>
#define Mod 30011
using namespace std;

struct Matrix
{
	int m,n;
	int a[110][110];
} a,b,c;

Matrix operator * (const Matrix &a,const Matrix &b)
{
	Matrix c;
	c.m=a.m;c.n=b.n;
	for(int i=1;i<=a.m;i++)
		for(int j=1;j<=b.n;j++)
		{
			c.a[i][j]=0;
			for(int k=1;k<=a.n;k++)
				c.a[i][j]=(c.a[i][j]+a.a[i][k]*b.a[k][j])%Mod;
		}
	return c;
}

Matrix operator ^ (Matrix a,int b)
{
	Matrix ans=a;b--;
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
	int n,m;
	cin>>n>>m;
	b.m=2*n;b.n=1;
	b.a[1][1]=b.a[n+1][1]=b.a[n+2][1]=1;
	a.m=a.n=2*n;
	for(int i=1;i<=n;i++) a.a[i][n+i]=1;
	for(int i=n+1;i<=2*n;i++) a.a[i][i-n]=1;
	a.a[n+1][n+1]=a.a[n+1][n+2]=1;
	for(int i=n+2;i<2*n;i++)
		a.a[i][i]=a.a[i][i-1]=a.a[i][i+1]=1;
	a.a[2*n][2*n]=a.a[2*n][2*n-1]=1;
	c=a^(m-3);
	b=c*b;
	int ans1=b.a[n][1];
	b=a*b;
	cout<<(b.a[2*n][1]-ans1+Mod)%Mod<<endl;
	return 0;
}
```
