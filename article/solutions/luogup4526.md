[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Simpson)

# 【LuoguP4526】自适应辛普森法2 题解

正无穷？！惊了！我们的辛普森积分只能做定积分啊……

没有思路，在几何画板上随便画几个图像看看这是什么鬼函数

![](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fshq7jlyr1j30we0guglx.jpg)

好像发现了什么大新闻！！！

当a>0时，函数曲线在x>10的时候就近乎与x轴重合了，也就是说后面的我们都可以不用去管它。于是我们定积分，在正半轴积到10就可以了！但由于本题精度要求非常高，为了进一步减小误差，我们积到20（实测积到10只能拿90分，积到20可以AC）。注意绝对不能从0开始积，而应该从一个极小量开始，因为x在幂中做了分母，积0就呵呵了。

当a<0时，发现它不是收敛的，它在靠近y轴时趋于无穷大（图中黄线）。于是我们在a<0时输出orz即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const double eps=1e-12;
double a;

double F(double x){return pow(x,a/x-x);}
double simpson(double a,double b)
{
	double mid=(a+b)/2;
	return (F(a)+4*F(mid)+F(b))*(b-a)/6;
}
double asr(double a,double b,double A)
{
	double mid=(a+b)/2;
	double L=simpson(a,mid),R=simpson(mid,b);
	if(fabs(L+R-A)<=15*eps) return L+R+(L+R-A)/15.0;
	return asr(a,mid,L)+asr(mid,b,R);
}
double asr(double a,double b){return asr(a,b,simpson(a,b));}

int main()
{
	cin>>a;
	if(a<0) puts("orz");
	else printf("%.5lf\n",asr(eps,20.0));
	return 0;
}
```
