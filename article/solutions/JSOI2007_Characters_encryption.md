[返回首页](https://EbolaEmperor.github.io)
# 【JSOI2007】字符加密 题解

[我是题目链接](https://www.lydsy.com/JudgeOnline/problem.php?id=1031)

### 题解

·JS省选居然考这样的模板题……

·肯定不会傻到真的把全部搞出来然后排序，毕竟它们都是重复的，所以我们可以首尾相接搞出一个新串，然后对新串进行后缀排序

·对于所有≤n的位置p，都是可行的决策位置。因此我们按排好的顺序遍历所有后缀，对于所有长度大于n的后缀，输出它的第n个字符即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=200010;
char s[N];
int sa[N],c[N];
int x[N],y[N],tmp[N];

void get_sa(int n,int m)
{
	for(int i=1;i<=n;i++) c[x[i]=s[i]]++;
	for(int i=1;i<m;i++) c[i+1]+=c[i];
	for(int i=n;i>=1;i--) sa[c[x[i]]--]=i;
	for(int k=1;k<=n;k<<=1)
	{
		int num=0;
		for(int i=n-k+1;i<=n;i++) y[++num]=i;
		for(int i=1;i<=n;i++) if(sa[i]>k) y[++num]=sa[i]-k;
		for(int i=1;i<=m;i++) c[i]=0;
		for(int i=1;i<=n;i++) tmp[i]=x[y[i]];
		for(int i=1;i<=n;i++) c[tmp[i]]++;
		for(int i=1;i<m;i++) c[i+1]+=c[i];
		for(int i=n;i>=1;i--) sa[c[tmp[i]]--]=y[i];
		num=1;swap(x,y);x[sa[1]]=1;
		for(int i=2;i<=n;i++)
			x[sa[i]]=(y[sa[i]]!=y[sa[i-1]]||y[sa[i]+k]!=y[sa[i-1]+k])?++num:num;
		if(num==n) break;
		m=num;
	}
}

int main()
{
	scanf("%s",s+1);
	int len=strlen(s+1),n=2*len;
	for(int i=len+1;i<=n;i++)
		s[i]=s[i-len];
	get_sa(n,255);
	for(int i=1;i<=n;i++)
		if(sa[i]<=n-len) putchar(s[sa[i]+len-1]);
	return 0;
}
```