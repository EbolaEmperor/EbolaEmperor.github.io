[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Matrix)

# 【NOI2013】矩阵游戏 题解

#### 为什么会有这篇题解？

网上其实有很多这道题的题解。但我为什么要再写一遍呢？

因为网上的题解重点全都在怎么处理n和m，以及为什么a=1和c=1要特判，没一篇题解说到怎么构造矩阵！他们都说构造很简单，但我认为对于我这种新手还是有难度的。所以我把我构造的矩阵拿出来供参考。

#### 矩阵递推公式推导

我们先解决每一行内的递推，容易推出：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%871-3-300x101.png)

所以对于每一行，有：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%872-2-300x99.png)

接下来解决每一行第一个数字的递推，容易推出：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%873-1-300x104.png)

结合行内递推公式，得到：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%874-1-350x85.png)

所以对于每一行的第一个数，有：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%875-1-480x166.png)

将这个式子代入第二个式子，并且将F(1,1)=1代入，得到最终的递推式：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%871-4-768x221.png)

#### n和m怎么处理？

发现n和m都是10的十万次方级的。所以你想怎么办？写高精度？

要是写了高精度，时间复杂度就糟糕的不能再糟糕了！还是洗洗睡，拿暴力分吧，还在这里推什么矩阵！

还记得费马小定理吗？

如果p是一个质数，则对于任意正整数a，都满足：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%871-e1522910871562-230x55.png)

你以为真的只是对于任意正整数满足？我告诉你，如果a是一个矩阵，费马小定理依然成立！而我们要模的数1000000007，不就是一个质数吗？

如果指数大于(p-1)，不就循环了？所以我们读n的时候模上(p-1)不就行了？

所以边读边模！

#### 为什么我只能拿90分？

很简单，当a或c等于0时要特判！至于怎么特判，请参照下面的代码。至于为什么要特判，网上一大把的证明，自己搜去。

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
#define Mod 1000000007
using namespace std;

typedef long long LL;

struct Matrix
{
	int m,n;
	LL a[5][5];
	Matrix(int _m=0,int _n=0):m(_m),n(_n){}
};

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
			for(int k=1;k<=a.n;k++) ans.a[i][j]=(ans.a[i][j]+a.a[i][k]*b.a[k][j]%Mod)%Mod;
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

LL read(string s,LL p)
{
	int pos=0;
	char c=s[pos++];
	LL ans=0;
	while(c<'0'&&c>'9') c=s[pos++];
	while(c>='0'&&c<='9') ans=(ans*10+c-'0')%p,c=s[pos++];
	return ans;
}

int main()
{
	LL n,m,a,b,c,d;
	string s1,s2;
	cin>>s1>>s2>>a>>b>>c>>d;
	if(a==1) n=read(s1,Mod);else n=read(s1,Mod-1);
	if(c==1) m=read(s2,Mod);else m=read(s2,Mod-1);
	Matrix A;
	A.m=A.n=2;
	A.a[1][1]=a;A.a[1][2]=b;
	A.a[2][1]=0;A.a[2][2]=1;
	A=QuickPow(A,m-1);
	Matrix B;
	B.m=B.n=2;
	B.a[1][1]=c;B.a[1][2]=d;
	B.a[2][1]=0;B.a[2][2]=1;
	B=B*A;
	B=QuickPow(B,n-1);
	A=A*B;
	Matrix ans;
	ans.m=2;ans.n=1;
	ans.a[1][1]=ans.a[2][1]=1;
	ans=A*ans;
	cout<<ans.a[1][1]<<endl;
	return 0;
}
```
