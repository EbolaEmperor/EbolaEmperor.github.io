# 【AGC001D】Arrays and Palindrome

题目链接：[AGC001 D. Arrays and Palindrome  -  AtCoder](https://agc001.contest.atcoder.jp/tasks/agc001_d)

建立一个图论模型。对于钦定相等的两个点，给它们连一条边，然后一个联通块内的所有点都要相等

那对于一个长度为n的回文串，其实就是钦定了i与n-i+1要相等，相当于在图中连了n/2条边（下取整）

为了让n个点连通，我们需要至少n-1条边。对于一个由若干段回文构成的序列，它至多连出(n-odd)/2条边，其中odd表示大小为奇数的段数

可以看出，若一开始给定的数列中，奇数大于2个，必然无解

那么对于一个有解的数列，把奇数放在首尾得到序列1，第一个数减一、最后一个数加一得到序列2

这样，每一段都是错开的，每一条边都必然连接两个原本不连通的块

因为题目要求不能输出0，所以需要再特判一下

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=110;
int a[N],n,m;
int oddnum[2];

int main()
{
    int odd=0,tot=0;
    scanf("%d%d",&n,&m);
    oddnum[0]=1;oddnum[1]=m;
    for(int i=1;i<=m;i++)
    {
        scanf("%d",a+i);
        if(~a[i]&1) continue;
        oddnum[odd++]=i;
        if(odd>2) return puts("Impossible"),0;
    }
    swap(a[1],a[oddnum[0]]);
    swap(a[m],a[oddnum[1]]);
    if(m==1&&a[1]==1) return puts("1\n1\n1"),0;
    for(int i=1;i<=m;i++) printf("%d ",a[i]);
    if(m==1) return printf("\n2\n%d 1\n",a[1]-1),0;
    printf("\n%d\n%d ",a[m]>1?m:m-1,a[1]+1);
    for(int i=2;i<m;i++) printf("%d ",a[i]);
    if(a[m]>1) printf("%d\n",a[m]-1);
    return 0;
}
```

