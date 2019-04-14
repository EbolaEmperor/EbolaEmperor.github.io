[返回首页](https://EbolaEmperor.github.io)
# 【CF17E】Palisection 题解

### 题解

·相交的回文串对数不好求，我们考虑“正难则反”，求不相交的回文串对数ni，然后用总对数sum减去它就是答案

·对于每一个manacher填充后的位置i，显然它对sum的贡献是p[i]/2，这样求出了sum

·那么对于任意一个原串位置x，在它后面的回文串个数设为m，以它结尾的回文串个数设为n，那么这个点对ni的贡献就是n*m

·我们在manacher匹配过程中，对于任意一个位置i，可以把i~i+p[i]的n值+1，把i-p[i]~i的mp值+1，那么m数组就是mp数组的后缀和

·暴力加的时间复杂度显然不对，我们可以差分，然后O(n)时间恢复出n数组、mp数组

·此题实现细节诸多，具体参考代码

```cpp
#include<bits/stdc++.h>
#define Mod 51123987
using namespace std;
 
typedef long long LL;
const int N=2000010;
char s[N],ss[2*N];
int p[2*N];
LL sum=0;
LL bg[2*N],ed[2*N];
 
int manacher()
{
	int l=strlen(s),len=0;
	ss[len++]='$';ss[len++]='#';
	for(int i=0;i<l;i++)
		ss[len++]=s[i],ss[len++]='#';
	ss[len++]='?';
	int mx=0,id;
	for(int i=1;i<=len;i++)
	{
		if(i<mx) p[i]=min(p[2*id-i],mx-i);
		else p[i]=1;
		while(ss[i-p[i]]==ss[i+p[i]]) p[i]++;
		sum=(sum+p[i]/2)%Mod;
		if(i+p[i]>mx) mx=i+p[i],id=i;
	}
	return len;
}
 
int main()
{
	int n;
	scanf("%d%s",&n,s);
	int len=manacher();
	sum=(sum*(sum-1)/2)%Mod;
	for(int i=2;i<=len;i+=2)
		bg[i-p[i]+2]++,bg[i+2]--,
		ed[i]++,ed[i+p[i]]--;
	for(int i=1;i<=len;i+=2)
		bg[i-p[i]+2]++,bg[i+1]--,
		ed[i+1]++,ed[i+p[i]]--;
	for(int i=2;i<=len;i+=2)
		bg[i]=(bg[i]+bg[i-2])%Mod,
		ed[i]=(ed[i]+ed[i-2])%Mod;
	bg[len+1]=0;
	for(int i=len-1;i>0;i-=2)
		bg[i]=(bg[i]+bg[i+2])%Mod;
	for(int i=2;i<=len;i+=2)
		sum=((sum-ed[i]*bg[i+2]%Mod)+Mod)%Mod;
	cout<<sum<<endl;
	return 0;
}
```