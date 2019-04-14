# A. Polynominal

### 题目大意

给定三个正整数a,b,c，求有多少个各项系数均为非负整数的多项式P，满足P(a)=b,P(b)=c

a,b,c在long long范围内。答案对998244353取模

### 题解

看到取模，是不是以为这是一道神仙计数题啊（掀桌

冷静……

我们先假设多项式至少有两项。因为多项式系数非负，所以必然有P(x)>=x。如果x是一个正整数，则P(x)一定比多项式P的任意一个系数都大。所以b一定比多项式P的任意一个系数都大。于是![](http://latex.codecogs.com/svg.latex?P(c)=k_0+k_1b+k_2b^2+k_3b^3+...)，因为![](http://latex.codecogs.com/svg.latex?k_i<b)，所以将![](http://latex.codecogs.com/svg.latex?k_i)其实就是将c转成b进制后第i位的数。因为将c转换成b进制的方法是唯一的，所以k数组的取值也是唯一的！

好的，现在你离正解就差一些特判了（滑稽

如果a=b=c=1，则多项式可以是任意的幂函数，故答案为无穷大

如果a=b=c≠1，则多项式可以是P(x)=a或P(x)=x，故答案为2

如果a=1,c是b的幂，则多项式可以是![](http://latex.codecogs.com/svg.latex?P(x)=b\times&space;x^{\log_bc-1})，答案为1

上面那三个特判看代码不太容易明白，所以提了一下。剩下的特判我就懒得说了，直接看代码自己YY一下吧

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
LL a,b,c,k[100];

bool ispow(LL a,LL b)
{
    while(a%b==0) a/=b;
    return a==1;
}

int Main()
{
    if(a==1&&b==1&&c==1) return -1;
    if(a==b) return b==c?2:0;
    if(a>b) return b==c?1:0;
    if(b>=c) return b==c?1:0;
    if(a==1&&ispow(c,b)) return 1;
    LL x=c;int n=0;
    while(x) k[++n]=x%b,x/=b;
    LL y=0;x=1;
    for(int i=1;i<=n;i++)
        y+=k[i]*x,x*=a;
    return y==b;
}

int main()
{
    while(cin>>a>>b>>c)
    {
        int ans=Main();
        if(ans==-1) puts("infinity");
        else printf("%d\n",ans);
    }
    return 0;
}
```