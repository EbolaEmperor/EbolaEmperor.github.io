# 【清华集训2014】玛里苟斯

#### 题目链接：[36. 玛里苟斯  -  UOJ](http://uoj.ac/problem/36)

#### 看着不爽？--->[公式问题，转到这里去吧](https://ebola-emperor.blog.luogu.org/post-qing-hua-ji-xun-2014-ma-li-gou-si)

分类讨论题。如果你不想看分类讨论的题解就左转百度吧，不分类讨论也是可做的

不难发现，把原序列换成它的线性基，答案不变，这个应该不用解释一眼就能看出吧

那接下来就根据k来分类讨论喽

### k≥3

我们可以在线性基上跑暴力，大力枚举选取的子集，把所有子集异或和加起来除一个$2^m$就是答案了，其中$m$表示线性基的规模。算上求$k$次幂的时间，该算法复杂度为$O(k\times 2^m)$。接下来我们来算算$m$大概是多少

为了方便表述，我们现在约定$\oplus(S)$表示集合$S$中所有元素的异或和

据题意，有：$O\left(\frac{\sum\limits_{S\subseteq base}\oplus(S)^k}{2^m}\right)=O\left(2^{63}\right)$，也即$O\left(\sum\limits_{S\subseteq base}\oplus(S)^k\right)=O\left(2^{63+m}\right)$

继续化简：$O(2^m\times\oplus(S)^k)=O(2^{63+m})$，于是有$O(\oplus(S)^k)=O(2^{63})$，也即$O(\oplus(S))=O(2^{\frac{63}{k}})$

所以说线性基的位数是$2^{\frac{63}{k}}$级别的！那么当$k\geq 3$时，暴力可以通过！（事实上还可以额外通过$k=1$和$k=2$的点各$15$分，也就是说只要写了这个暴力，$90$分就到手了）注意爆long long的问题，如果是场上也许得手写128位整数，但是在OJ上就懒得写了，直接调用现成的__int128即可

暴力给$90$，这真的是清华集训？？？？算了不纠结这个，我们来考虑剩下的$10$分

### k=1

设$P=\{\oplus(S_1),\oplus(S_2),...,\oplus(S_{2^m})\}$

那么答案可以表示为：$\frac{\sum\limits_{i=1}^{2^m}P_i}{2^m}$

我们设$bit_{i,j}$表示$P_i$的第$j$个二进制位是多少，于是答案表示为：$\frac{\sum\limits_{i=1}^{2^m}\sum\limits_{j=0}^{63}bit_{i,j}\times2^j}{2^m}$

我们可以调换$i,j$的枚举顺序，然后提出$2^j$，得到：$\sum\limits_{j=0}^{63}2^{j-m}\sum\limits_{i=1}^{2^m}bit_{i,j}$

我们考虑式子中$\sum\limits_{i=1}^{2^m}bit_{i,j}$的组合意义。不难想到，就是把线性基中第$j$位为$1$的数看作黑球，其它看作白球，线性基中的所有球不一样。然后求：在这些球中取若干个球，使得取出的球中有奇数个黑球，方案数是多少。

对于每个$j$，我们设$c$表示线性基中第$j$位为$1$的元素个数，根据上述组合意义，有：$\sum\limits_{i=1}^{2^m}bit_{i,j}=\sum\limits_{i=1}^{\left\lfloor\frac{c}{2}\right\rfloor}\binom{c}{2i-1}2^{m-c}$，那么只要预处理组合数，这个东西就可以$O(m)$求出了！

套入上述公式，就能得到一个复杂度为$O(64m)$的优秀做法了！

### k=2

这就是最恶心的部分了。说实话要是场上我绝对不会为了$5$分去写这个部分的东西

$bit_{i,j}$的定义仍然与上面相同。答案可以表示为：$\frac{\sum\limits_{i=1}^{2^m}\left(\sum\limits_{j=0}^{63}bit_{i,j}\times2^j\right)\times \left(\sum\limits_{j=0}^{63}bit_{i,j}\times2^j\right)}{2^m}$

卷积一下，就是$\frac{\sum\limits_{i=1}^{2^m}\left(\sum\limits_{j=0}^{63}\sum\limits_{k=0}^{63}bit_{i,j}\times bit_{i,k}\times2^{j+k}\right)}{2^m}$

然后老套路，调换枚举顺序，得到：$\sum\limits_{j=0}^{63}\sum\limits_{k=0}^{63}2^{j+k-m}\sum\limits_{i=1}^{2^m}bit_{i,j}\times bit_{i,k}$

还是思考$\sum\limits_{i=1}^{2^m}bit_{i,j}\times bit_{i,k}$的组合意义。大菜逼我睡了整整一中午才想到……就是设$m$为线性基中元素的数量，现在你有一个含有$m$个不同球的盒子，每个球有++/+-/-+/--四种属性，第一个位置为+可以让$A$加一，第二个位置为+可以让$B$加一，若为-则没有影响。现在要在$m$个球中取若干个，求让$A,B$均为奇数的方案数。若$base_i$的第$j$个二进制位为$1$，则第一个位置是+，若$base_i$的第$k$个二进制位为$1$，则第二个位置是+（感觉都可以出一道提高T2了）

这个用组合数解决不了，得用状压dp。将球的属性表示成一个两位的二进制数，然后将$A,B$的奇偶性也表示为两位二进制数（1表示奇），设$f_{x,s}$表示考虑到第$x$个元素，当前$A,B$的状态是$s$的方案数。不难写出转移：$f_{x,s}=f_{x-1,s}+f_{x-1,s\oplus cur}$，其中$cur$表示当前这个元素的二进制状态，$\oplus$表示异或

那么对于每组$j,k$，都做一遍这样的dp，然后套入$\sum\limits_{j=0}^{63}\sum\limits_{k=0}^{63}2^{j+k-m}f_{m,3}$计算就行了。复杂度$O(63^2\times 4m)$

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef unsigned long long ULL;
typedef long double LD;
const int S=(1<<20)+5;
char buf[S],*H,*T;
inline char Get()
{
    if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
    if(H==T) return -1;return *H++;
}
inline ULL read()
{
    ULL x=0;char c=Get();
    while(!isdigit(c)) c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x;
}

ULL base[70],b[70];
int n,m,K;
__int128 ans=0;
ULL tot=1;
ULL C[65][65];

void insert(ULL x)
{
    for(int i=63;i>=0;i--)
        if(x&(1ull<<i))
        {
            if(base[i]) x^=base[i];
            else{base[i]=x;break;}
        }
}

__int128 P(ULL x){__int128 res=1;for(int i=1;i<=K;i++) res*=x;return res;}

void dfs(int p,ULL cur)
{
    if(p==m){ans+=P(cur);return;}
    dfs(p+1,cur^b[p]);
    dfs(p+1,cur);
}

void Task1()
{
    for(int i=1;i<=m;i++) tot*=2ull;
    dfs(0,0);
    if(ans==0){puts("0");return;}
    __int128 tmp=ans/(tot/2);
    ULL ttt=tmp/2;cout<<ttt;
    if(tmp&1) puts(".5");
}

void Task2()
{
    for(int i=0;i<=63;i++)
    {
        C[i][0]=1ull;
        for(int j=1;j<=i;j++)
            C[i][j]=C[i-1][j]+C[i-1][j-1];
    }
    LD pw2=1,ans=0;
    for(int k=0;k<=63;k++,pw2*=2)
    {
        LD res=0;int c=0;
        for(int i=0;i<=63;i++) c+=bool(base[i]&(1ull<<k));
        for(int i=1;i<=(c+1)/2;i++) res+=C[c][2*i-1];
        LD tmp=1;for(int i=1;i<=c;i++) tmp*=2;
        ans+=res*pw2/tmp;
    }
    if(abs(ans-round(ans))<0.1) printf("%.0Lf",ans);
    else printf("%.1Lf\n",ans);
}

__int128 dp(int x,int y)
{
    int S[65]={0};
    __int128 f[2][4]={0};
    for(int i=0;i<m;i++)
        S[i]=(((b[i]>>x)&1)<<1)+((b[i]>>y)&1);
    f[0][0]=1;int k=1;
    for(int i=0;i<m;i++,k^=1)
        for(int s=0;s<=3;s++)
            f[k][s]=f[k^1][s]+f[k^1][s^S[i]];
    return f[k^1][3];
}

void Task3()
{
    __int128 ans1=0,ans2=0,pwm=1;
    for(int i=1;i<=m;i++) pwm*=2;
    for(int j=0;j<=63;j++)
        for(int k=0;k<=63;k++)
        {
            __int128 pw2=1,res=dp(j,k);
            for(int i=1;i<=j+k;i++) pw2*=2;
            if(j+k>=m) ans1+=res*(pw2/pwm);
            else ans2+=res*pw2;
        }
    __int128 tmp=ans2/(pwm/2)+ans1*2;
    ULL ttt=tmp/2;cout<<ttt;
    if(tmp&1) puts(".5");
}

int main()
{
    n=read();K=read();
    for(int i=1;i<=n;i++) insert(read());
    for(int i=0;i<=63;i++) if(base[i]) b[m++]=base[i];
    if(K>=3) Task1();
    if(K==1) Task2();
    if(K==2) Task3();
    return 0;
}
```

