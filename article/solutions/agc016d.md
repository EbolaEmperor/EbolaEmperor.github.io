# 【AGC016D】XOR Replace

题目链接：[AGC016 D. XOR Replace  -  AtCoder](https://agc016.contest.atcoder.jp)

首先不难发现，这个异或是假的。对于第一次操作，的确是把ai换成了异或和，但是接下来，根据异或的相消原则，数列的异或和就是上次操作的ai，再接下来的操作都是一样的了

于是可以想象这么一个过程：你一开始有一个数，然后你把一个数换成这个数，同时拿走换的那个数，再继续去换别的

那根据质量守恒定律（网友表示：什么鬼），没有任何新元素产生，也没有任何元素泯灭，你所能做的只有交换，因此b中不能出现多于一个与a中不同的元素。就算有一个与a中不同的元素，那它也必然要等于a的异或和。

为了方便判定，我们可以将a的异或和加到a尾部，b的异或和加到b的尾部（在有解的情况下，b的异或和必然是a中与b不同的那个元素，证明显然），这样一来排序后的a和b必然是相同的，否则无解

已经相同了的位置我们就不用再去考虑了，只取考虑那些还需要操作的位置

在最优情况下，必然是每一步操作都能解决一个位置，然后拿走这个位置原来的数，去解决下一个位置。如果将不同的元素看成若干个点，一个位置x就相当于b\[x\]向a\[x\]连了一条边，然后我们要沿着边去走，出发点就是a的异或和。

现在问题变成了：你有一个由若干个环构成的图，还有编号为1-m的边（m表示待操作位置的数量），另加一条编号为m+1的边（就是我们新加的两个异或和）。你需要从点a\[n+1\]出发，把所有1-m的边走一遍，在两个联通块之间传送需要额外付出1的代价，求最小代价

那答案就是m+环的数量-1，此外若点a\[n+1\]是独立的联通块，答案还需要加一

```cpp
#include<bits/stdc++.h>
using namespace std;

const int S=(1<<20)+5;
char buf[S],*H,*T;
inline char Get()
{
    if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
    if(H==T) return -1;return *H++;
}
inline int read()
{
    int x=0;char c=Get();
    while(!isdigit(c)) c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x;
}

const int N=100010;
int a[N],b[N],n;
int _a[N<<1],_b[N];
int fa[N<<1],sz[N<<1];

int find(int x){return fa[x]==x?x:fa[x]=find(fa[x]);}

int main()
{
    n=read();
    int xora=0,xorb=0,ans=0;
    for(int i=1;i<=n;i++) xora^=(_a[i]=a[i]=read());
    for(int i=1;i<=n;i++) xorb^=(_b[i]=b[i]=read());
    a[++n]=xora;b[n]=xorb;_a[n]=a[n];_b[n]=b[n];
    sort(_a+1,_a+1+n);sort(_b+1,_b+1+n);
    for(int i=1;i<=n;i++) if(_a[i]!=_b[i]) return !puts("-1");
    for(int i=1;i<n;i++) ans+=(a[i]!=b[i]);
    if(ans==0) return !puts("0");
    for(int i=1;i<=n;i++) _a[i+n]=_b[i];
    sort(_a+1,_a+1+n);
    int hs=unique(_a+1,_a+1+n)-(_a+1);
    for(int i=1;i<=hs;i++) fa[i]=i,sz[i]=1;
    for(int i=1;i<=n;i++)
    {
        a[i]=lower_bound(_a+1,_a+1+hs,a[i])-_a;
        b[i]=lower_bound(_a+1,_a+1+hs,b[i])-_a;
        int x=find(a[i]),y=find(b[i]);
        if(x!=y) fa[x]=y,sz[y]+=sz[x];
    }
    if(a[n]==b[n]&&sz[find(a[n])]==1) ans++;
    for(int i=1;i<=n;i++) ans+=(fa[i]==i&&sz[i]>1);
    cout<<ans-1<<endl;
    return 0;
}
```

