[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Eular)

# 欧拉函数 学习笔记

### 定义

欧拉函数是指在<=n的正整数中，与n互质的正整数的个数。数学描述：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%871-1-230x65.png)

### 性质

如果p是一个素数，则：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%872-230x54.png)

如果p是一个素数，a是一个正整数，则：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%873-230x39.png)

欧拉函数是积性函数。具体地说，若a、b互质，则有：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%874-230x37.png)

如果p是一个素数，且n mod p == 0，则有：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%875-230x45.png)

如果p是一个素数，且n mod p != 0，则有：![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/04/%E5%9B%BE%E7%89%876-230x35.png)

上述性质不必过度追求理解，有兴趣的可以查阅资料看一看上述性质的证明。这些证明这里就不给出了，因为插公式太麻烦了。当然，如果看不懂证明，也没什么关系。其实欧拉函数这种东西，你只要知道它的定义和求法就可以了，这里列出性质，是为了让大家更好地理解下面的求法。这些性质可以直接记忆。

### 求法

单个数欧拉函数的朴素求法是O(sqrt(n))的。在实际应用中，通常不会去用它。因为一般来说，题目一旦考到了欧拉函数，不可能只要你求一个数的欧拉函数。因此，这里也就不介绍朴素的欧拉函数求法了。

欧拉函数可以像素数一样地线性筛出来。建议不会线性筛素数的同学先去学习一下，因为欧拉函数的线性求法是建立在素数的线性筛法的基础之上的。也就是说，欧拉函数是在线性筛素数的同时，顺便求出来的。线性筛欧拉函数应用到了上述性质1、4、5。代码实现如下：

```cpp
int phi[1000010];
bool mark[1000010];
int prime[1000010],tot=0;

void getphi(int n)
{
	memset(mark,0,sizeof(mark));
	phi[1]=1;
	for(int i=2;i<=n;i++)
	{
		if(!mark[i])
		{
			prime[++tot]=i;
			phi[i]=i-1;
		}
		for(int j=1;j<=tot&&i*prime[j]<=n;j++)
		{
			mark[i*prime[j]]=1;
			if(i%prime[j]==0){phi[i*prime[j]]=phi[i]*prime[j];break;}
			else phi[i*prime[j]]=phi[i]*(prime[j]-1);
		}
	}
}
```
