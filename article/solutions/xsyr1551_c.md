# C. 排序II

### 题面

![](http://www.ebola.pro/images/xsyr1551_c_1.png)

### 题解

不难发现答案就是i xor pi的最高二进制位（i从0开始编号）

直接对每个二进制位统计有多少个i xor pi的这一位为1。更新和询问都是log的复杂度

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=500010;
int cnt[30],p[N],n,m;

void update(int x,int k){for(int i=0;i<=20;i++) if((x^p[x])&(1<<i)) cnt[i]+=k;}

int main()
{
	scanf("%d%d",&n,&m);
	for(int i=0;i<n;i++)
		scanf("%d",p+i),update(i,1);
	int x,y;
	while(m--)
	{
		scanf("%d%d",&x,&y);x--;y--;
		update(x,-1);update(y,-1);
		swap(p[x],p[y]);
		update(x,1);update(y,1);
		int ans=20;while(ans>=0&&!cnt[ans]) ans--;
		printf("%d\n",(ans>=0?1<<ans:0));
	}
	return 0;
}
```

