[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Katalan)

# 【原创】二叉排序树

## 题目背景

这是一道Ebola的原创题

## 题目描述

一颗二叉排序树的插入操作伪代码如下：

```pas
Procedure Insert(int node,int x)
Begin
        If node=NULL Then node=new, val(node)=x
        If val(node)=x Then Exit
        If val(node)<x Then Insert(RightSon(node),x)
        If val(node)>x Then Insert(LeftSon(node),x)
End
```

现给定一个长度为n的序列A，请你计算用不同的顺序将序列中所有元素插入这个二叉排序树中，最终二叉排序树有多少种可能的形态

答案可能很大，对998244353取模

## 输入输出格式

### 输入格式：

第一行一个整数n，含义如题

第二行n个数，表示给定序列

### 输出格式：

一行一个数表示答案

## 输入输出样例

### 输入样例#1
```
3
2 5 4
```
### 输出样例#1:
```
5
```
### 输入样例#2：
```
20
41 67 34 0 69 24 78 58 62 64 5 45 81 27 61 91 95 42 27 36 
```
### 输出样例#2：
```
769018837
```
## 说明

n<10^6 , Ai<10^9 n<10 
[提交地址](https://www.luogu.org/problemnew/show/U25697)

## 题解

若元素不重复，那么容易发现，答案就是n个结点的不同形态的二叉树数量

这就变成了林厚从《数学一本通》里面的一道例题，答案就是卡特兰数

那么我们排序去重后，输出对应的卡特兰数即可

```cpp
#include<iostream>
#include<cstdio>
#include<cmath>
#include<algorithm>
#define MOD 998244353
using namespace std;

typedef long long LL;
LL a[10000010];

LL inv(LL a)
{
	LL b=MOD-2,ans=1;
	while(b)
	{
		if(b&1) ans=ans*a%MOD;
		a=a*a%MOD;
		b>>=1;
	}
	return ans;
}

LL C(LL a,LL b)
{
	LL ans1=1;
	for(LL i=a;i>b;i--)
		ans1=ans1*i%MOD;
	LL ans2=1;
	for(LL i=a-b;i>1;i--)
		ans2=ans2*i%MOD;
	return ans1*inv(ans2)%MOD;
}

int main()
{
	int n;
	cin>>n;
	for(int i=0;i<n;i++) scanf("%lld",a+i);
	sort(a,a+n);
	int m=unique(a,a+n)-a;
	cout<<C(2*m,m)*inv(m+1)%MOD<<endl;
	return 0;
}
```
