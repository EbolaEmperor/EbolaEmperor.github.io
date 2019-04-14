# B. 一道拉格朗日反演好题

**题目链接：[UOJ#428](http://uoj.ac/problem/428)**

这道题真是秀到我了qwq，第一步就非常神仙qwq

将操作视作一棵树，点的编号就是操作序号。对于一个插入$1$的操作，将删去的位置作为该操作的子节点。于是整个操作就与一棵树互相对应了，其中插入$0$的操作是叶子节点。

于是我们现在就是要求n个点的合法树的数量。合法的树应该满足：
对于非叶子节点，若它的儿子均为叶子，则子节点数量在集合$B$中；若它的儿子不只有叶子节点，则它是叶子的子节点数量在集合$A$中

设$f_n$表示$n$个节点的不同合法树数量，$g_n$表示$n$个节点的不同合法树森林数量，于是得到以下转移方程：

$$f_i=[i-1 \in B]+\sum\limits_{j=0}^{i-2}[j \in A]g_{i-1-j}\binom{i-1}{j}$$

$$g_i=\sum\limits_{j \geq 2}f_jg_{i-j}\binom{i-1}{j-1}$$

直接dp是$O(n^2)$的，考虑用CDQ分治NTT优化

对于一个区间$[l,r]$，我们考虑$[l,mid]$对$[mid+1,r]$的贡献。先来考虑$f$

对于$f_i$，区间$[l,mid]$中的$g$值对它的贡献可以表述成：$(i-1)!\sum\limits_{j=i-1-r}^{i-1-l}[j \in A]\frac{g_{i-1-j}}{j!(i-1-j)!}$

令$x=i-j,y=j$，则可以化为：$(i-1)!\sum\limits_{x+y=i,x\in[l+1,mid+1]}\frac{[j \in A]}{j!}\times\frac{g_{x-1}}{(x-1)!}$，这是一个卷积形式，可以用NTT优化

再来考虑$g$

对于$g_i$，区间$[l,mid]$对它的贡献可以表述为：$(i-1)!\left[\left(\sum\limits_{j=\max(l,i-l)}^{mid}\frac{f_jg_{i-j}}{(i-j)!(j-1)!}\right)+\left(\sum\limits_{i-mid}^{\min(i-l,mid)}\frac{f_jg_{i-j}}{(i-j)!(j-1)!}\right)\right]$

令$x=i-j,y=j$，则可以化为两个卷积相加的形式，两个卷积式子是相同的，都是：$\sum\limits_{x+y=i}\frac{g_x}{x!}\times\frac{f_y}{(y-1)!}$。但$x$、$y$的范围不一样，具体地可以自己推敲一下或参考代码

然后就做完了，最后的答案就是$f_n$。复杂度$O(n \log^2 n)$，实测比std一个log跑的更快，当然最重要的是好写

***

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

typedef long long LL;
const int ha=998244353;
const int N=400010;
int A[N],B[N],n,ma,mb;
int rev[N],omg[N],iomg[N],inv[N];
int t1[N],t2[N],t3[N],t4[N],t5[N];
int f[N],g[N],fac[N],ifac[N];

inline int add(const int &x,const int &y){return (x+y>=ha)?(x+y-ha):(x+y);}
inline int dev(const int &x,const int &y){return (x-y<0)?(x-y+ha):(x-y);}

int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=(LL)a*a%ha)
        if(b&1) ans=(LL)ans*a%ha;
    return ans;
}

void NTT_init()
{
    for(int i=1;i<N;i<<=1)
    {
        int p=i<<1;
        omg[i]=Pow(3,(ha-1)/p);
        iomg[i]=Pow(omg[i],ha-2);
        inv[i]=Pow(i,ha-2);
    }
}

void fac_init(int n)
{
    fac[0]=1;
    for(int i=1;i<=n;i++)
        fac[i]=(LL)fac[i-1]*i%ha;
    ifac[n]=Pow(fac[n],ha-2);
    for(int i=n-1;i>=0;i--)
        ifac[i]=(LL)ifac[i+1]*(i+1)%ha;
}

void NTT(int *A,int n,int v)
{
    for(int i=0;i<n;i++) if(i<rev[i]) swap(A[i],A[rev[i]]);
    for(int i=1;i<n;i<<=1)
    {
        int wn=(v>0)?omg[i]:iomg[i];
        for(int j=0;j<n;j+=(i<<1))
            for(int k=0,w=1;k<i;k++)
            {
                int x=A[j+k],y=(LL)w*A[i+j+k]%ha;
                A[j+k]=add(x,y);
                A[i+j+k]=dev(x,y);
                w=(LL)w*wn%ha;
            }
    }
    if(v<0) for(int i=0;i<n;i++) A[i]=(LL)A[i]*inv[n]%ha;
}

void clear(int n)
{
    memset(t1,0,sizeof(int)*(n+1));
    memset(t2,0,sizeof(int)*(n+1));
    memset(t3,0,sizeof(int)*(n+1));
    memset(t4,0,sizeof(int)*(n+1));
    memset(t5,0,sizeof(int)*(n+1));
}

void gao(int l,int r)
{
    if(l==r)
    {
        f[l]=add(f[l],B[l-1]);
        g[l]=add(g[l],f[l]);
        return;
    }
    int mid=(l+r)/2;gao(l,mid);
    int len=1,pl=0;for(;len<=r-l;len<<=1,pl++);clear(len);
    for(int i=0;i<len;i++) rev[i]=(rev[i>>1]>>1)|((i&1)<<pl-1);
    for(int i=l;i<=mid;i++) t1[i-l]=(LL)g[i]*ifac[i]%ha;
    for(int i=0;i<r-l;i++) t2[i]=A[i]*ifac[i];
    NTT(t1,len,1);NTT(t2,len,1);
    for(int i=0;i<len;i++) t2[i]=(LL)t1[i]*t2[i]%ha;
    NTT(t2,len,-1);
    for(int i=mid+1;i<=r;i++) f[i]=add(f[i],(LL)fac[i-1]*t2[i-l-1]%ha);
    for(int i=2;i<=min(r-l,mid);i++) t3[i-1]=(LL)f[i]*ifac[i-1]%ha;
    for(int i=1;i<=min(r-l,l-1);i++) t4[i-1]=(LL)g[i]*ifac[i]%ha;
    for(int i=max(l,2);i<=mid;i++) t5[i-l]=(LL)f[i]*ifac[i-1]%ha;
    NTT(t3,len,1);NTT(t4,len,1);NTT(t5,len,1);
    for(int i=0;i<len;i++) t1[i]=add((LL)t1[i]*t3[i]%ha,(LL)t4[i]*t5[i]%ha);
    NTT(t1,len,-1);
    for(int i=mid+1;i<=r;i++) g[i]=add(g[i],(LL)fac[i-1]*t1[i-l-1]%ha);
    gao(mid+1,r);
}

int main()
{
    NTT_init();fac_init(114514);
    n=read();ma=read();mb=read();
    if(n==1) return puts("1"),0;
    for(int i=1;i<=ma;i++) A[read()]=1;
    for(int i=1;i<=mb;i++) B[read()]=1;
    B[0]=0;gao(1,n);printf("%d\n",f[n]);
    return 0;
}
```