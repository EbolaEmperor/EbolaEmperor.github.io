[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/LinearBasis)

# 【CQOI2013】新Nim游戏 题解

其实就是要求极大线性无关组，也就是说，选取一些数，使得它们不靠“外力”无法异或出0，最大化它们的和

什么？你问我证明？我也不会啊！如果你真的想要证明，请移步[wyfcyx_forever的博客](https://blog.csdn.net/wyfcyx_forever/article/details/39477673)

那么就是一个线性基的经典问题了

把给定的数排序，然后从大到小顺次枚举，如果线性基中不能插入这个数，则答案贪心地加上它，否则插入它。就是异或线性基经典的存在性询问

```cpp
#include<bits/stdc++.h>
using namespace std;

int a[110],base[35],n;

bool insert(int x)
{
	for(int i=30;i>=0;i--)
		if(x&(1<<i))
		{
			if(base[i]) x^=base[i];
			else{base[i]=x;break;}
		}
	return x==0;
}

int main()
{
	scanf("%d",&n);
	for(int i=1;i<=n;i++)
		scanf("%d",a+i);
	sort(a+1,a+1+n);
	long long ans=0;
	for(int i=n;i>0;i--)
		if(insert(a[i])) ans+=1ll*a[i];
	cout<<ans<<endl;
	return 0;
}
```
