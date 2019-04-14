[返回首页](https://EbolaEmperor.github.io)
# 【BZOJ2565】最长双回文串 题解

### 题解

·用manacher算法，计算出以每个位置p结尾的最长回文串长度lx[p]，以及以p开头的最长回文串长度rx[p]，然后枚举位置，把对应的lx和rx加起来即可

·计算lx与rx只需计算出每个位置的p数组后更新即可

```cpp
#include<bits/stdc++.h>
using namespace std;

char s[100010],ss[200010];
int p[200010],lx[200010],rx[200010];

int manacher()
{
	int l=strlen(s),len=1;
	ss[len++]='$';ss[len++]='#';
	for(int i=0;i<l;i++)
		ss[len++]=s[i],ss[len++]='#';
	ss[len]='\0';
	int mx=-1,id=0;
	for(int i=1;i<=len;i++)
	{
		if(i<mx) p[i]=min(p[2*id-i],mx-i);
		else p[i]=1;
		while(ss[i+p[i]]==ss[i-p[i]]) p[i]++;
		if(i+p[i]>mx) mx=i+p[i],id=i;
		lx[i+p[i]-1]=max(lx[i+p[i]-1],p[i]-1);
		rx[i-p[i]+1]=max(rx[i-p[i]+1],p[i]-1);
	}
	return len;
}

int main()
{
	scanf("%s",s);
	int len=manacher(),ans=0;
	for(int i=2;i<=len;i+=2) rx[i]=max(rx[i],rx[i-2]-2);
	for(int i=len;i>=2;i-=2) lx[i]=max(lx[i],lx[i+2]-2);
	for(int i=2;i<=len;i+=2)
		ans=max(ans,lx[i]+rx[i]);
	cout<<ans<<endl;
	return 0;
}
```