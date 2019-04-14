[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Matrix)

# 【LuoguP1707】刷题比赛 题解

这道题真的是非常非常非常地令人痛苦！

手写了一个11*11的递推矩阵，仿佛身体被掏空……

首先看到这个数据范围，就知道这题应该要写一个复杂度大概O(log N)的算法才能A

看到这种递推数列，而且时间复杂度要求那么苛刻的，肯定就是要用矩阵加速

那么我们来思考怎么构造矩阵

下面这个是我构造的初始状态矩阵（当然肯定有更加优雅的构造方法，毕竟我太菜了）

![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%871-2.png)

接下来就是构造递推矩阵

这题的恶心之处就在于后面那一大堆次方数的处理

可以说这样的递推数列真是丑陋不堪

直接上我构造的递推矩阵吧，可以自己手算矩阵乘法验证一下我的递推矩阵

思路当然是将上面的初始状态矩阵里面的数据看做已知量，通过这些已知量推出未知量

![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%872-1.png)

为了防止出现负数，构造完这个递推矩阵后要 (每个数%Mod+Mod)%Mod

由于模数太大了，边乘边模会爆掉，所以还要写一个快速乘，把乘法转换成加法……

所以最终代码的时间复杂度为O(d*log N*log Mod)

其中d是常数，由于递推矩阵过于庞大，算法的常数高达1331（就是11的3次方），因此不能忽略这个常数

然而如此糟糕的时间复杂度还是在100ms内轻松A掉了

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;

typedef long long LL;
LL Mod;

struct Matrix
{
	int m,n;
	LL a[20][20];
	Matrix(int _m=0,int _n=0):m(_m),n(_n){}
};

LL QuickMul(LL a,LL b)
{
	LL ans=0;
	while(b)
	{
		if(b&1) ans=(ans+a)%Mod;
		a=(a+a)%Mod;
		b>>=1;
	}
	return ans;
}

Matrix operator * (Matrix a,Matrix b)
{
	if(a.n!=b.m) return Matrix(0,0);
	Matrix ans;
	ans.m=a.m;
	ans.n=b.n;
	for(int i=1;i<=a.m;i++)
		for(int j=1;j<=b.n;j++)
		{
			ans.a[i][j]=0;
			for(int k=1;k<=a.n;k++) ans.a[i][j]=(ans.a[i][j]+QuickMul(a.a[i][k],b.a[k][j]))%Mod;
		}
	return ans;
}

Matrix QuickPow(Matrix a,LL b)
{
	if(b==0) return Matrix(0,0);
	Matrix ans=a;
	b--;
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
	LL n;
	LL p,q,r,t,u,v,w,x,y,z;
	cin>>n>>Mod>>p>>q>>r>>t>>u>>v>>w>>x>>y>>z;
	Matrix A;
	A.m=A.n=11;
	memset(A.a,0,sizeof(A.a));
	A.a[1][1]=p;A.a[1][2]=1;A.a[1][3]=1;A.a[1][4]=q;A.a[1][7]=r;A.a[1][8]=t-2*r;A.a[1][11]=r-t+1;
	A.a[2][1]=1;A.a[2][2]=u;A.a[2][3]=1;A.a[2][5]=v;A.a[2][9]=1;
	A.a[3][1]=1;A.a[3][2]=1;A.a[3][3]=x;A.a[3][6]=y;A.a[3][8]=1;A.a[3][10]=1;A.a[3][11]=1;
	A.a[4][1]=1;A.a[5][2]=1;A.a[6][3]=1;
	A.a[7][7]=1;A.a[7][8]=2;A.a[7][11]=1;
	A.a[8][8]=1;A.a[8][11]=1;
	A.a[9][9]=w;A.a[10][10]=z;A.a[11][11]=1;
	for(int i=1;i<=11;i++)
		for(int j=1;j<=11;j++) A.a[i][j]=(A.a[i][j]%Mod+Mod)%Mod;
	A=QuickPow(A,n-2);
	Matrix ans;
	ans.m=11;
	ans.n=1;
	ans.a[1][1]=3;ans.a[2][1]=3;ans.a[3][1]=3;
	ans.a[4][1]=1;ans.a[5][1]=1;ans.a[6][1]=1;
	ans.a[7][1]=4;
	ans.a[8][1]=2;
	ans.a[9][1]=w%Mod;
	ans.a[10][1]=z%Mod;
	ans.a[11][1]=1;
	ans=A*ans;
	cout<<"nodgd "<<ans.a[1][1]<<endl;
	cout<<"Ciocio "<<ans.a[2][1]<<endl;
	cout<<"Nicole "<<ans.a[3][1]<<endl;
	return 0;
}
```
