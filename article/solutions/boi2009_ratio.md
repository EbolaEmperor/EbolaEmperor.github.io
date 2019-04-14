[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/KMP)

# 【BOI2009】Radio Transmission 题解

不知道为什么二中的OJ混进了这么一道水题，洛谷评级入门……

构造出给定字符串的next数组，那么next[n]~n一定构成最小循环节，所以答案就是n-next[n]

```cpp
#include<bits/stdc++.h>
using namespace std;

char s[1000010];
int nxt[1000010];

int main()
{
	int n,j=0;
	scanf("%d%s",&n,s+1);
	for(int i=2;i<=n;i++)
	{
		while(j&&s[i]!=s[j+1]) j=nxt[j];
		if(s[i]==s[j+1]) j++;
		nxt[i]=j;
	}
	cout<<n-nxt[n]<<endl;
	return 0;
}
```
