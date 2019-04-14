# Lucas定理   学习笔记

模板题链接：[戳我](https://www.lydsy.com/JudgeOnline/problem.php?id=2982)

### 前置芝士

如果要你求C(n,k) mod p的值，你会怎么求？

你也许知道组合数和杨辉三角的关系，于是可以递推预处理出**一些**C。对了，问题就出在“一些”二字上。“一些”是多少？杨辉三角的递推显然是O(n²)的，因此，顶多预处理3000*3000的组合数值。当n,m比较大时，这并不是一种优秀的方法

你也许知道组合数的公式。你甚至会逆元。于是你可以O(n)预处理出阶乘，以及阶乘的逆元，然后直接套公式O(1)得到C(n,m)。这个方法看上去并没有什么问题

Lucas定理求组合数也是需要线性预处理的，而且求C(n,m)还比上述法二复杂度更高。那么既然有了上述法二，为什么还要Lucas定理呢？

### 推翻法二

我们考虑用法二来求C(50,25) mod 23的值

![](http://wx2.sinaimg.cn/mw690/0060lm7Tly1ft67hn2o2nj311y0kg0vz.jpg)

这是怎么回事？怎么输出了0？难道答案就是0？还是我的代码写错了？

代码写错显然是不可能的。~~我的代码怎么会写错~~

至于答案，我们用自带高精度的Python来验算一下

![](http://wx3.sinaimg.cn/mw690/0060lm7Tly1ft67oght60j30o60820to.jpg)

显然不是0。~~我也相信我的Python代码不可能写错~~

那么问题出在哪里呢？

我们看一看是不是fac数组出了锅，打出来看一下

![](http://wx1.sinaimg.cn/mw690/0060lm7Tly1ft67wcimfdj30it0crwew.jpg)

果然出锅了！从fac[23]开始，后面就全是0了

原因？很简单！fac[23]=fac[22]*23%23，乘个23又模个23，显然就变成0了啊！然后有了这一个0，后面推过去就全都是0了！但后面的答案显然不全是0啊！

所以说，法二只适用于n<p的情况

### 新的基本法要上台了

Lucas定理：![](http://latex.codecogs.com/svg.latex?C(n,m)=C(n/p,m/p)*C(n\;mod\;p,m\;mod\;p))

注意：这里的除法运算是整除

为什么它是对的？

那么好用又好记的定理还不赶紧记下来！在这里问为什么！比赛又不会考你证明！

那么我们就可以先线性预处理出fac[0~min(n,p)]，然后写一个函数递归求解了！

```cpp
#include<bits/stdc++.h>
#define ha 10007
using namespace std;

int fac[ha+5],ifac[ha+5];

int Pow(int a,int b)
{
	int ans=1;
	for(;b;b>>=1,a=1ll*a*a%ha)
		if(b&1) ans=1ll*ans*a%ha;
	return ans;
}

void Init()
{
	fac[0]=1;
	for(int i=1;i<ha;i++)
		fac[i]=1ll*fac[i-1]*i%ha;
	ifac[ha-1]=Pow(fac[ha-1],ha-2);
	for(int i=ha-2;i>=0;i--)
		ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
}

int C(int n,int m){return (n<m)?0:1ll*fac[n]*ifac[m]%ha*ifac[n-m]%ha;}
int Lucas(int n,int m)
{
	if(n<ha&&m<ha) return C(n,m);
	return 1ll*Lucas(n/ha,m/ha)*C(n%ha,m%ha)%ha;
}

int main()
{
	Init();
	int T,n,m;
	for(scanf("%d",&T);T;T--)
	{
		scanf("%d%d",&n,&m);
		printf("%d\n",Lucas(n,m));
	}
	return 0;
}
```

时间复杂度？

每次递归，问题的规模缩减一个p，因此复杂度是![](http://latex.codecogs.com/svg.latex?log_p\;n)的，非常优秀