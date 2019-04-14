# 【AGC003D】Anticube

题目链接：[AGC003 D. Anticube](https://agc003.contest.atcoder.jp/tasks/agc003_d)

以前做过一个假的Pollard-Rho练习题，好像是POI2010的什么非凡因子啥的题，那题用Pollard-Rho光荣T了，当时学习了一下题解，发现了一种奇妙分解法。好像这题刚好能用

考虑对于一个数a，我们将它的质因数分解出来，然后所有质因子的幂次对3取模，再乘回来得到一个新的数x。我们发现，若规定质因子幂次小于等于3，那么选了x后不能选的数就只有唯一的一个

若a按上述算法得到x，b按上述算法得到y，选了x后不能选y，那不难发现选了a后不能选b。该结论对任意能通过上述算法算出x、y的a、b均成立

于是我们可以对于所有的x，统计出按上述算法能算出它的a有多少个，记作“x的数量”，用map存储即可。然后对于每个a，按上述算法算出x，再算出选了x之后不能选的数y，我们在x和y中选一个数量更多的，再打上已选标记就行了。因为x和y是一一对应的，所以这个贪心策略的正确性是显然的

现在问题是如何对于一个数x，算出它对应的x

考虑朴素的质因数分解，那非常容易实现，只要事先把1e5内的质数筛出来就行了。可是那样复杂度显然是不可接受的，于是考虑之前做过的那题类似的方法

我们只筛出三次根级别的质数，然后对这些质数跑一遍分解。分解a之后有四种情况：

1. a变成了1。此时已经分解干净了，直接算x即可
2. a是一个大于二次根级别的质数。显然x直接等于a，不能选的就是a平方
3. a是两个介于三次根与二次根级别之间的不同质数之积。处理方式同情况2
4. a是一个介于三次根与二次根级别之间的质数平方。显然x还是a，但是选了x之后不能选的数是sqrt(x)

对于情况2和情况3，我们没必要去区分它，只要它不等于1，又不是完全平方数，那就按那样的方式去处理就行了

此外，在所有立方数中，至多只能选一个，因此贪心选取时要把x=1的情况特判掉

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
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

const int N=100010;
LL a[N],b[N][2];
bool mask[3010];
int prm[3010],tot=0,n;
unordered_map<LL,int> cc;
unordered_map<LL,bool> fucked;

LL maxc(const LL &a,const LL &b){return cc[a]>cc[b]?a:b;}

void preprework()
{
    for(int i=2;i<=2200;i++)
    {
        if(mask[i]) continue;
        prm[++tot]=i;
        for(int j=i;j<=2200;j+=i)
            mask[j]=1;
    }
}

void prework()
{
    for(int i=1;i<=n;i++)
    {
        LL x=1,y=1;
        for(int j=1;j<=tot;j++)
        {
            if(a[i]%prm[j]) continue;
            int cnt=0;
            while(a[i]%prm[j]==0)
                a[i]/=prm[j],cnt++;
            if(!(cnt%=3)) continue;
            for(int k=1;k<=cnt;k++) x=x*prm[j];
            for(int k=cnt+1;k<=3;k++) y=y*prm[j];
        }
        LL t=(LL)sqrt(a[i]);
        if(t*t==a[i]) x=x*t*t,y=y*t;
        else x=x*a[i],y=y*a[i]*a[i];
        b[i][0]=x;b[i][1]=y;
        if(!cc.count(x)) cc[x]=1;
        else cc[x]++;
    }
}

int main()
{
    LL ans=0;n=read();
    for(int i=1;i<=n;i++) a[i]=read();
    preprework();prework();
    for(int i=1;i<=n;i++)
    {
        LL c1=b[i][0],c2=b[i][1];
        if(fucked[c1]||fucked[c2]) continue;
        LL x=maxc(c1,c2);fucked[x]=1;
        ans+=(x==1)?1:cc[x];
    }
    cout<<ans<<endl;
    return 0;
}
```

