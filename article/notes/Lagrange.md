# 拉格朗日插值   学习笔记

[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Lagrange)

### 提出问题

给定一个n次多项式的点值表达式，请你将k代入多项式中求值，答案对998244353取模

范围：n≤2000

[提交地址](https://www.luogu.org/problemnew/show/P4781)

### 分析问题

如果我们把多项式的系数表达式看作一个n+1元方程，那么将所有点代入，就可以得到一个n+1元一次方程组，可以用高斯消元法求出多项式的各系数。这是初中老师常说的“待定系数法”

但高斯消元的时间复杂度高达O(n³)，显然不能解决这道题

其实题目并没有要求我们求出多项式的各项系数，只是要我们将k代入求值，那么直观上就感觉求出各项系数是绕了一个很大的弯路。有没有一种方法，可以免去这一步骤呢？

这就是我们即将要说到的拉格朗日插值

### 解决问题

伟大的数学家拉格朗日归纳前人的经验，将这个方法写成了论文

拉格朗日插值的核心就是下面这个公式：

![](http://latex.codecogs.com/svg.latex?f(x)=\sum_{i=1}^{n+1}y_i\prod_{j\neq&space;i}\frac{(x-x_j)}{(x_i-x_j)})

这个式子可能看着有点恐怖，我们举个具体的例子来直观感受一下

假如给定的点是(1,4)、(2,9)、(3,16)，将其代入拉格朗日插值公式并展开，得到：

![](http://latex.codecogs.com/svg.latex?f(x)=4\frac{(x-2)(x-3)}{(1-2)(1-3)}+9\frac{(x-1)(x-3)}{(2-1)(2-3)}+16\frac{(x-1)(x-2)}{(3-1)(3-2)})

然后~~用小学知识~~对它进行化简得到：![](http://latex.codecogs.com/svg.latex?f(x)=x^2+2x+1)

将给定点代入验算确定无误

这个式子为什么是正确的？

展开之后其实容易发现，如果我们将第m个给定点的横坐标代入，那么除第m个分式外，所有分式的分母中都会有一个因式等于0，这就相当于把除了第m个分式之外的分式全部废掉了。而第m个分式的分子分母，在代入横坐标后会完全相等，于是这个分式消掉了，留下的就只剩第m个给定点的纵坐标

其实我们上面的分析，还是把系数给求出来了，复杂度和高斯消元还是一样的

但这个式子非常适合我们提出的问题，我们可以把问题中给定的k直接代入拉格朗日插值公式，然后套公式算就可以了。公式里只要枚举i和j两个变量，于是时间复杂度降低到了O(n²)，问题迎刃而解

### 代码实现

其实代码实现起来并不难，因为拉格朗日插值公式本来就非常简洁。然后题目要求取模的话，我们就再写一个快速幂或扩欧来求一下逆元，因为复杂度瓶颈不在于求逆元，所以总的复杂度还是O(n²)的。下面贴出主要代码

```cpp
int lagrange(int *arrx,int *arry,int n,int x)
{
	int res=0;
	for(int i=0;i<n;i++)
	{
		int s1=1,s2=1;
		for(int j=0;j<n;j++)
		{
			if(i==j) continue;
			s1=1ll*s1*(x-arrx[j])%ha;
			s2=1ll*s2*(arrx[i]-arrx[j])%ha;
		}
		res=(res+1ll*s1*Pow(s2,ha-2)%ha*arry[i])%ha;
	}
	return res;
}
```

### 特殊情况

很多题目里面用拉格朗日插值时，都是求出1~n的函数值，然后插出一个比较大的k。其实拉格朗日插值在x取值连续的情况下，是可以做到O(n)的

可以预处理出![](http://latex.codecogs.com/svg.latex?(k-x_i))的前缀积与后缀积，形式化地表示就是：

![](http://latex.codecogs.com/svg.latex?pre(m)=\prod_{i=1}^m(k-x_i))

![](http://latex.codecogs.com/svg.latex?suf(m)=\prod_{i=m}^n(k-x_i))

所以拉格朗日插值公式的分子部分就可以表示为，![](http://latex.codecogs.com/svg.latex?pre(i-1)*suf(i+1))

因为x的取值是连续的，推一下不难发现分母其实是阶乘的形式。如果不考虑符号问题，就可以表示为![](http://latex.codecogs.com/svg.latex?(i-1)!*(n-i)!)

符号和(n-i)的奇偶性有关，如果(n-1)是奇数，就是负的，否则就是正的

那么只要预处理出阶乘的逆元，分母部分也就预处理出来了，那么整个插值就可以只枚举i了

预处理O(n)，插值O(n)，所以整体是O(n)的

贴一下主要代码。代码中的ifac是阶乘逆元，ifac的预处理，相信写过组合数学题的人都知道吧

```cpp
int Lagrange(int *y,int n,int k)
{
	pre[0]=suf[n+1]=1;
	for(int i=1;i<=n;i++) pre[i]=1ll*pre[i-1]*(k-i)%ha;
	for(int i=n;i>=1;i--) suf[i]=1ll*suf[i+1]*(k-i)%ha;
	int ans=0;
	for(int i=1;i<=n;i++)
	{
		int s1=1ll*pre[i-1]*suf[i+1]%ha;
		int s2=1ll*ifac[i-1]*ifac[n-i]%ha;
		int fg=(n-i)&1?-1:1;
		ans=(ans+1ll*fg*s1*s2%ha*y[i])%ha;
	}
	return (ans+ha)%ha;
}
```
