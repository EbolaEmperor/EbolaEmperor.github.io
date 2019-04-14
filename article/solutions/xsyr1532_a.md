# A. coin

### 题意简述

你有n个朋友和m种物品，你现在知道了第i个朋友喜欢第j种物品的概率。你可以自己选择带n个物品回去送给你的朋友，你的朋友会在你的物品中选择他喜欢的那个，如果没有他喜欢的那个他就不选。要求最大化获得物品的朋友数量期望，求这个期望值

### 题解

首先我们可以预处理出f\[i\]\[j\]表示i号物品带j个时，拿到这个物品的朋友数量期望。然后根据期望的线性可加性，可以直接将第i个物品选j个的期望加起来，我们要让每个物品的选择数量加起来恰好为n，所以直接跑一个背包即可

f\[i\]\[j\]的处理也非常简单。设a\[i\]\[k\]表示**恰好**有k个朋友喜欢i的概率，则有

![](http://latex.codecogs.com/svg.latex?f_{i,j}=\sum\limits_{k=0}^na_{i,k}\times\min(k,j))

然后考虑a数组的处理。设w\[i\]\[j\]表示第i个朋友喜欢j的概率，则构造生成函数

![](http://latex.codecogs.com/svg.latex?\prod\limits_{k=0}^n(w_{k,j}x+1-w_{k,j}))

这个多项式的k次项系数就是a\[j\]\[k\]

发现第一步（f数组的处理）和第二步（背包）的复杂度都是O(n²m)的，显然不能接受。第一步的处理不难想到用分治FFT优化，可是第二步我却没想到什么更好的方法

考虑暴力撵标算。不难发现，上面提到的那个生成函数，随着x次数的升高，系数会越来越小，最后趋近于0。而题目要求保留9位小数，所以当系数远小于1e-9的时候，我们就没必要继续计算下去了。这样一来，多项式乘法的复杂度从O(n²)降到了O(kn)，其中k是一个常数，表示多项式系数大于1e-13部分的次数，均摊不超过40。跑背包的时候，显然不需要考虑该物品选择超过k个的情况，于是总复杂度降到了O(knm)

然而事实证明数据极水，甚至根本不需要使用long double，而且只要考虑系数大于1e-11的部分就够了

```cpp
#include<bits/stdc++.h>
using namespace std;

const int S=(1<<20)+5;
char buf[S],*H,*T;
inline char Get()
{
    if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
    if(H==T) return -1;return *H++;
}
inline int read()
{
    int x=0;char c=Get();
    while(!isdigit(c)) c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x;
}

const int N=6010,M=610;
int p[N][M],n,m;
double a[N],b[N],f[M][N],sum[N];

void mul(double *A,int n,int x)
{
    double p=(double)x/1000.0,q=1-p;
    for(int i=n+1;i>0;i--)
        A[i]=A[i-1]*p+A[i]*q;
    A[0]=A[0]*q;
}

int main()
{
    n=read();m=read();
    for(int i=1;i<=n;i++)
        for(int j=1;j<=m;j++)
            p[i][j]=read();
    for(int i=1;i<=m;i++)
    {
        memset(a,0,sizeof(a));
        int len=0;a[0]=1;
        for(int j=1;j<=n;j++)
        {
            mul(a,len,p[j][i]);
            if(a[len+1]>1e-11) len++;
        }
        memset(b,0,sizeof(b));
        memset(sum,0,sizeof(sum));
        for(int j=len;j>=1;j--) b[j]=b[j+1]+a[j];
        for(int j=1;j<=len;j++) sum[j]=sum[j-1]+a[j]*j;
        for(int j=1;j<=len;j++) b[j]=b[j]*j+sum[j-1];
        for(int j=0;j<=n;j++)
            for(int k=0;k<=len;k++)
                f[i][j+k]=max(f[i][j+k],f[i-1][j]+b[k]);
    }
    printf("%.10lf\n",f[m][n]);
    return 0;
}
```

