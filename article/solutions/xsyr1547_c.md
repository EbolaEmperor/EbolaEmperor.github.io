# C. 排序

### 题面

![](http://www.ebola.pro/images/xsyr1547_c_1.png)

数据范围：n≤2e6

### 题解

考虑完整执行一次外层循环发生的变化。第i次外层循环，会找到从位置i的值开始，值不断下降的一个序列，然后将这个序列循环右移一位

假设外层循环共执行k轮，位置p上的数m是前k+1个数中最大的。那么，前p-1轮循环都不会影响m的位置，从第p次循环开始，m每次往右移动1位，最后移动到位置k+1

假设x是前k+2个数中除m以外最大的。那么它移动时，遇到m会移动两格来跨过它，其它时候都是移动一格，最后会移动到k+2位置。类似的，还可以推广到k+3位置、k+4位置

那么，前k个位置显然就是排好序的1到k。k+1到n位置，用一个堆来维护，每次将最大值放到k+i位置，然后弹出，再将k+i+1位置的初始值入堆即可

最后那些零散的步数暴力模拟内层循环即可

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
namespace IO
{
    const int S=(1<<20)+5;
    char buf[S],*H,*T;
    inline char Get()
    {
        if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
        if(H==T) return -1;return *H++;
    }
    inline LL read()
    {
        LL x=0;char c=Get();
        while(!isdigit(c)) c=Get();
        while(isdigit(c)) x=x*10+c-'0',c=Get();
        return x;
    }
    char obuf[S],*oS=obuf,*oT=oS+S-1,c,qu[55];int qr;
    inline void flush(){fwrite(obuf,1,oS-obuf,stdout);oS=obuf;}
    inline void putc(char x){*oS++ =x;if(oS==oT) flush();}
    template <class I>inline void print(I x)
    {
        if(!x) putc('0');
        while(x) qu[++qr]=x%10+'0',x/=10;
        while(qr) putc(qu[qr--]);
    }
}

using namespace IO;
const int N=2000010;
priority_queue<int> pq;
int n,a[N],k=0;
LL cnt;

int main()
{
    n=read();cnt=read();
    for(int i=1;i<=n;i++) a[i]=read();
    for(int i=1;i<=n;i++)
    {
        if(cnt<n-i) break;
        cnt-=n-i;k++;
    }
    for(int i=1;i<=k+1;i++) pq.push(a[i]);
    for(int i=k+1;i<=n;i++) a[i]=pq.top(),pq.pop(),pq.push(a[i+1]);
    for(int i=1;i<=k;i++) a[i]=i;
    for(int j=k+2;j<=n;j++)
    {
        if(a[j]<a[k+1]) swap(a[j],a[k+1]);
        if(!--cnt) break;
    }
    for(int i=1;i<=n;i++) print(a[i]),putc(' ');
    return flush(),0;
}
```

