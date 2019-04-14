# 特征多项式及其常规求法

### 一些定义

我们定义矩阵的特征多项式满足：![img](http://latex.codecogs.com/svg.latex?p_A(k)=det(kI_n-A))

其中，det(M)表示M的行列式的值，![img](http://latex.codecogs.com/svg.latex?I_n)表示n阶单位矩阵（即n*n的，对角线上元素均为1，其余位置均为0的矩阵）

性质：n阶矩阵的特征多项式次数为n，且最高次项系数为1

我们称![img](http://latex.codecogs.com/svg.latex?p_A(k))为k对应的特征值

### 常规求法

首先，我们可以用高斯消元求矩阵的行列式，复杂度O(n³)

我们还知道，一个n次的多项式，只要知道了位于它函数图像上的n+1个点，这个多项式就能被唯一确定

于是我们可以枚举1到n+1，算出对应的特征值，这就是特征多项式的点值表达法了

有了点值表达法，就可以用高斯消元或拉格朗日插值的方式得到多项式的系数表达法

高斯消元似乎代码量更小一些……然而我傻逼忘记了高消，写完拉格朗日插值和多项式乘法，才想起这么一回事，然而代码都已经打完了，对拍都已经过了，也就懒得改了……

复杂度瓶颈在于求特征值，总复杂度![img](http://latex.codecogs.com/svg.latex?O(n^4))

```cpp
#include<bits/stdc++.h>
#define ha 1000000007
using namespace std;

int Pow(int a,int b)
{
	int ans=1;
	for(;b;b>>=1,a=1ll*a*a%ha)
		if(b&1) ans=1ll*ans*a%ha;
	return ans;
}

struct Matrix
{
	int n,a[75][75];
	Matrix(){memset(a,0,sizeof(a));}
	int* operator [] (const int &x){return a[x];}
	
	friend Matrix operator * (Matrix &A,Matrix &B)
	{
		Matrix C;C.n=A.n;
		for(int i=1;i<=A.n;i++)
			for(int j=1;j<=A.n;j++)
				for(int k=1;k<=A.n;k++)
					C[i][j]=(C[i][j]+1ll*A[i][k]*B[k][j])%ha;
		return C;
	}
};

struct Poly
{
	int n,a[155];
	Poly(){memset(a,0,sizeof(a));n=0;}
	int& operator [] (const int &x){return a[x];}
	
	friend Poly operator * (Poly A,Poly B)
	{
		Poly C;C.n=A.n+B.n;
		for(int i=0;i<=A.n;i++)
			for(int j=0;j<=B.n;j++)
				C[i+j]=(C[i+j]+1ll*A[i]*B[j])%ha;
		return C;
	}
	
	friend Poly operator * (Poly A,int x)
	{
		Poly C=A;
		for(int i=0;i<=C.n;i++)
			C[i]=1ll*C[i]*x%ha;
		return C;
	}
	
	friend Poly operator + (Poly A,Poly B)
	{
		Poly C;C.n=max(A.n,B.n);
		for(int i=0;i<=C.n;i++)
			C[i]=(A[i]+B[i])%ha;
		return C;
	}
};

int Det(Matrix &A)
{
	int ans=1;
	for(int i=1;i<=A.n;i++)
	{
		int pos=i;
		while(pos<=A.n&&A[pos][i]==0) pos++;
		if(!A[pos][i]) return 0;
		if(pos!=i)
		{
			ans=1ll*ans*(ha-1)%ha;
			for(int j=1;j<=A.n;j++)
				swap(A[pos][j],A[i][j]);
		}
		ans=1ll*ans*A[i][i]%ha;
		int inv=Pow(A[i][i],ha-2);
		for(int j=1;j<=A.n;j++)
		{
			if(i==j||A[j][i]==0) continue;
			int t=1ll*A[j][i]*inv%ha;
			for(int k=1;k<=A.n;k++)
				A[j][k]=(A[j][k]-1ll*A[i][k]*t%ha+ha)%ha;
		}
	}
	return ans;
}

int n,k;
Matrix A,B;
int ay[75];

Poly Lagrange(int n)
{
	Poly res;
	for(int i=0;i<=n;i++)
	{
		Poly s1;
		s1[0]=1;
		for(int j=0;j<=n;j++)
		{
			if(i==j) continue;
			Poly tmp;tmp.n=1;
			tmp[0]=(ha-j)%ha;tmp[1]=1;
			s1=s1*tmp*Pow((i-j+ha)%ha,ha-2);
		}
		res=res+s1*ay[i];
	}
	return res;
}

int main()
{
	scanf("%d",&k);A.n=k;
	for(int i=1;i<=k;i++)
		for(int j=1;j<=k;j++)
			scanf("%d",&A[i][j]);
    for(int x=0;x<=k;x++)
    {
    	B=A;
    	for(int i=1;i<=k;i++)
    		B[i][i]=(B[i][i]-x+ha)%ha;
    	ay[x]=Det(B);
    }
    Poly cp=Lagrange(k);
    for(int i=0;i<=cp.n;i++) printf("%d ",cp[i]);
    return 0;
}
```

### 更优秀的求法

yww大佬的博客里介绍了一种O(n³)的求法，我并没怎么看懂，这里还是放一个链接吧：[yww的博客](https://www.cnblogs.com/ywwyww/p/8522541.html)