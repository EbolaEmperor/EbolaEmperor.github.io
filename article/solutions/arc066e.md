# 【AGC066E】Addition and Subtraction Hard

题目链接：[ARC066 E. Addition and Subtraction Hard  -  AtCoder](https://arc066.contest.atcoder.jp/tasks/arc066_c)

容易看出，两层以上的括号都是无意义的，它一定可以被等价地拆成两层以内。而且括号一定是加在减号后面，不然加了和没加有什么区别

于是设f\[x\]\[0/1/2\]表示考虑到第x个数，前面有0/1/2个未匹配左括号是的最大值，转移需要考虑各种添括号拆括号的情况，代码注释已经非常详细了。注意到f\[x\]\[...\]只与f\[x-1\]\[...\]有关，所以用滚动数组把第一维压掉（不压也行）

注意我们说的“添括号”是指在第i个操作符后添一个左括号，“拆括号”是指在第i个操作符前添一个右括号

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
inline int readopt()
{
    char c=Get();
    while(c!='+'&&c!='-') c=Get();
    return c;
}

typedef long long LL;
int num,n,k;
char opt;
LL f[2][3];

inline LL mymax(const LL &x,const LL &y){return x>y?x:y;}

int main()
{
    n=read();
    f[k=0][0]=read();
    f[0][1]=f[0][2]=-(1ll<<60);
    for(int i=1;i<n;i++,k^=1)
    {
        opt=readopt();num=read();
        if(opt=='+')
        {
            f[k^1][0]=mymax(mymax(f[k][0],f[k][1]),f[k][2])+num;  //原地转移 或 拆1/2个括号
            f[k^1][1]=mymax(f[k][1],f[k][2])-num;  //原地转移 或 拆1个括号
            f[k^1][2]=f[k][2]+num;  //原地转移
        }
        else
        {
            f[k^1][0]=mymax(mymax(f[k][0],f[k][1]),f[k][2])-num;  //原地转移 或 拆1/2个括号
            f[k^1][1]=mymax(mymax(f[k][1],f[k][2])+num,f[k][0]-num);  //原地转移 或 拆1个括号 或 添1个括号
            f[k^1][1]=mymax(mymax(f[k][1]-num,f[k][2]-num),f[k^1][1]);  //拆1/2个括号又添1个括号
            f[k^1][2]=mymax(f[k][2]-num,f[k][1]+num);  //原地转移 或 添1个括号
            f[k^1][2]=mymax(f[k][2]+num,f[k^1][2]);  //拆1个括号又添1个括号
        }
    }
    LL ans=mymax(mymax(f[k][0],f[k][1]),f[k][2]);
    printf("%lld\n",ans);
    return 0;
}
```

