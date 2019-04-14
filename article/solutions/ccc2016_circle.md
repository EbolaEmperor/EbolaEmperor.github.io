# 【CCC2016】生命中的圆

题目中的条件其实就是说：![](http://latex.codecogs.com/svg.latex?f_{i,j}=f_{i-1,j-1}\oplus&space;f_{i-1,j+1})，其中![](http://latex.codecogs.com/svg.latex?\oplus)表示异或

不难推出结论：![](http://latex.codecogs.com/svg.latex?f_{i,j}=f_{i-2^k,j-2^k}\oplus&space;f_{i-2^k,j+2^k},\quad&space;k\in\mathbb{N})。数学归纳法证明如下：

1. 当k=0时，这就是题目给出的条件
2. 当k>0时，假设结论对于k-1成立，则有：![](http://latex.codecogs.com/svg.latex?f_{i,j}=f_{i-2^{k-1},j-2^{k-1}}\oplus&space;f_{i-2^{k-1},j+2^{k-1}}=(f_{i-2^{k},j-2^{k}}\oplus&space;f_{i-2^{k},j})\oplus(f_{i-2^{k},j}\oplus&space;f_{i-2^{k},j+2^k})=f_{i-2^k,j-2^k}\oplus&space;f_{i-2^k,j+2^k})

于是问题就变得非常简单了。将T二进制分解，然后套用上述结论推log T行就行了，复杂度O(n log T)

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=100010;
int n,f[2][N],p=0;
char s[N];
LL T;

int main()
{
    cin>>n>>T>>s;
    for(int i=0;i<n;i++) f[0][i]=s[i]-'0';
    for(int k=0;k<60;k++)if(T&(1ll<<k))
    {
        int pr=(1ll<<k)%n,pl=(n-pr)%n;
        for(int i=0;i<n;i++)
        {
            f[p^1][i]=f[p][pl]^f[p][pr];
            if(++pl>=n) pl-=n;
            if(++pr>=n) pr-=n;
        }
        p^=1;
    }
    for(int i=0;i<n;i++)
        printf("%d",f[p][i]);
    return 0;
}
```