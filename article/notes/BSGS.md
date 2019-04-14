[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/BSGS)

# Baby-Step-Giant-Step(BSGS)算法

### 提出问题

现有指数同余方程 a ^ x ≡ b (mod p)，其中p是质数，a^x表示a的x次方， a、b均为小于等于p的非负整数。请你求出这个方程的最小正整数解。

### 智障想法

看到这个问题，很容易想到，我们不断枚举x，直到得出解。就像这样：

```cpp
int x=0,d=1;
while(true)
{
  if(d==b) {cout<<x<<endl;return 0;}
  d=d*a%p;
  x++;
}
```

### 有点智商！

我可没告诉你上面那个方程一定有解。万一没有解，你不就死循环了？

我们思考这样一个问题：为什么会没有解？

因为我们的计算进入了一个循环，而循环节内没有解！

因此，我们只要发现我们进入了循环，我们就可以说方程无解。

怎样判断进入了循环？很简单，只要计算出的结果在之前算出来过，那后面就是循环了。

利用哈希容器map，写出如下代码。（map的用法在百度里搜出来有很多，自行学习，一定要学会，不然没法写最后出场的BSGS算法）

```cpp
int x=0,d=1;
map<int,bool> f;
while(true)
{
  if(d==b) {cout<<x<<endl;return 0;}
  if(f.count(d)!=0) {cout<<“No solution!”<<endl;return 0;}
  f[d]=true;
  d=d*a%p;
  x++;
}
```

### 继续思考

上面那个算法的时间复杂度似乎无法确定。但肯定非常糟糕！而且哈希容器map会额外带来一个log的时间复杂度。

我们能不能不用map？

我们用数学方法分析一下，什么时候会出现循环。

你是不是很快想到了什么？没错！费马小定理！（见我的另一篇文章“Fermat素数判定”）

费马小定理告诉我们，当计算到a的p次方时，就会出现循环！

因此，我们只要在0到p之间枚举x就行了！时间复杂度O(p)

```cpp
int d=1;
for(int x=0;x<=p;x++)
{
  if(d==b) {cout<<x<<endl;return 0;}
  d=d*a%p;
}
```

### BSGS正式出场！

O(p)的时间复杂度太高了。我们能不能设法优化一下？O(log p)显然不现实，那我们就想想能不能优化到O(sqrt(p))

我们设m=sqrt(p)向上取整，然后将a的0~m次方(mod p)预处理出来，并存入哈希表中，这样我们就能O(1)时间查询某个数是不是a的0~m次幂(mod p)，并快速知道它是a的几次幂(mod p)

这个预处理有什么用呢？请看下面的推导

设 x = i * m +ｊ，则 a ^ x ≡ ((a ^ m) ^ i) * (a^ j)

原方程化为 ((a ^ m) ^ i) * (a^ j) ≡ b (mod p)

我们可以在0~m之间枚举i，然后把((a ^ m) ^ i)视作常数d，(a ^ j)视作未知数y，这样就得到了一个标准的线性同余方程 dy ≡ b (mod p)


线性同余方程可以用扩展欧几里得定理，在常数时间内求解

利用预处理出的哈希表判定y是不是a的0~m次幂，如果是，说明我们得到了原方程的解：x = i * m + j

我们会发现，假如原方程有解，那么一定存在一个i，使得解出的y是a的0~m次幂

用扩展欧几里得算法要记得 ((结果%p)+p)%p，要不然搞出负数就不好玩了

上述算法时间复杂度为O(sqrt(p))，但是我比较懒，不想手写哈希，然后就用了哈希容器map，因此我代码的实际时间复杂度是O(sqrt(p) log p)，只要把map改成手写的哈希就可以变成O(sqrt(p))了

```cpp
typedef long long LL;

void ExGcd(LL a,LL b,LL &x,LL &y)
{
	if(b==0) {x=1,y=0;return;}
	ExGcd(b,a%b,x,y);
	LL t=x;
	x=y;
	y=t-(a/b)*y;
}

LL BSGS(LL a,LL b,LL p)
{
	map<LL,LL> f;
	LL m=(LL)ceil(sqrt(p));
	LL base=1;
	for(LL i=0;i<m;i++)
	{
		f.insert(pair<LL,LL>(base,i));
		base=base*a%p;
	}
	LL d=1;
	for(LL i=0;i<m;i++)
	{
		LL x,y;
		ExGcd(d,p,x,y);
		x=(x*b%p+p)%p;
		if(f.count(x)!=0) return i*m+f[x];
		d=d*base%p;
	}
	return -1;
}
```
