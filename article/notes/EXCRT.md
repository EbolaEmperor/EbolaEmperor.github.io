# 扩展中国剩余定理  学习笔记

首先要明确一点，扩展中国剩余定理和中国剩余定理没有太大的关系，也就是说你就算不会中国剩余定理，你也可以学这个，而且学了这个就可以不用学中国剩余定理了！

### 提出问题

给定同余方程组，每个方程形如：x mod p = b，p不一定是质数，甚至不保证所有p互质，求解一个最小的x使得所有方程成立

### 解决问题

扩展中国剩余定理是基于同余方程合并的一种做法。其基本思想是每次合并两个方程，在n-1次合并之后变成一个方程，然后就直接得到答案

现在需要解决的，就是两个同余方程如何合并

![](http://latex.codecogs.com/svg.latex?\left\{\begin{aligned}x\equiv&space;b_1\;(mod\;\;p_1)\\x\equiv&space;b_2\;(mod\;\;p_2)\end{aligned}\right)

对于这样的一个方程组，我们可以将其化为下面的形式

![](http://latex.codecogs.com/svg.latex?\left\{\begin{aligned}x=k_1p_1+b_1\\x=k_2p_2+b_2\end{aligned}\right.)

其中k1,k2是未知量

~~通过小学知识~~，我们可以得到一个关于k1,k2的不定方程

![](http://latex.codecogs.com/svg.latex?p_1k_1-p_2k_2=b_2-b_1)

用扩展欧几里得定理求解即可。但这个不定方程方程不一定有解，如果它没有解，那么整个同余方程组就是无解的

用扩欧求出的是一组特解。我们知道，不定方程ax+by=c的通解可以表示为x=x0+bk/gcd(a,b)，因此将求出的k1特解对p2/gcd(p1,p2)取模，即可得到最小的解，避免后面的溢出问题

得到k1之后，就可以代入原方程，求得x。我们知道，lcm(p1,p2)对原方程没有贡献，因此x加上lcm(p1,p2)的任意倍数，原来的两个方程依然成立。所以我们合并之后的新方程就是：

![](http://latex.codecogs.com/svg.latex?y\equiv&space;x\;(mod\;\;lcm(p_1,p_2)))

简单的理解最后这一步合并，就是我们求出的x已经满足原方程组了，而x加上lcm(p1,p2)的任意倍数都还满足原方程组，因此x+k*lcm(p1,p2)都是可以的解，写成同余方程的形式就是上面那样了

为了保证x求出来是最小的，我们要让x模一下lcm(p1,p2)

重复进行上面的过程，将方程合并到只剩一个，最后剩下那个方程的余数就是答案了

两个同余方程的合并代码如下：

```cpp
typedef __int128 LLL;
struct Equ{LLL b,m;};

Equ ExCRT(Equ a1,Equ a2)
{
	LLL k1,k2;
	LLL g=ExGcd(a1.m,a2.m,k1,k2);
	LLL c=a2.b-a1.b;
	if(c%g!=0) puts("No!"),exit(0);
	k1=k1*(c/g)%(a2.m/g);
	LLL x=k1*a1.m+a1.b;
	LLL lcm=a1.m/g*a2.m;
	x=(x%lcm+lcm)%lcm;
	return Equ{x,lcm};
}
```
