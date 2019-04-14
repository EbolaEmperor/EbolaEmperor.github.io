# 【AGC004C】AND Grid

题目链接：[AGC004 C. AND Grid  -  AtCoder](https://agc004.contest.atcoder.jp/tasks/agc004_c)

### 题意简述

因为洛谷的翻译完全错误（或者说正常人完全无法理解），我再来翻译一下

给定一个黑白网格，你需要构造两个新的黑白网格，满足：

1、若原网格的一个位置是黑，则两个新网格的那个位置都要是黑

2、若原网格的一个位置是白，则至多只能有一个新网格的那个位置是黑

3、两个新网格中的黑格子均需要构成四联通

另：原网格的边界全为白色格子

### 题解

边界全白？那不是随便乱搞……

对于新网格一，把奇数列全部染黑，第一排和最后一排不要染，这样所有原本就有的黑色，就至少与一列构成了四联通。再把第一排全部染黑，所有列就四联通了，搞定！

对于新网格二，奇数列染不得了，那就染偶数列，再把最后一排染黑，搞定！

```cpp
#include<bits/stdc++.h>
using namespace std;

namespace IO
{
    const int S=(1<<20)+5;
    char buf[S],*H,*T;
    inline char Get()
    {
        if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
        if(H==T) return -1;return *H++;
    }
    inline void reads(char *s)
    {
        char c=Get();int tot=0;
        while(c!='#'&&c!='.') c=Get();
        while(c=='#'||c=='.') s[++tot]=c,c=Get();
        s[++tot]='\0';
    }
    char obuf[S],*oS=obuf,*oT=oS+S-1,c,qu[55];int qr;
    inline void flush(){fwrite(obuf,1,oS-obuf,stdout);oS=obuf;}
    inline void putc(char x){*oS++ =x;if(oS==oT) flush();}
    inline void prints(const char *s)
    {
        int len=strlen(s);
        for(int i=0;i<len;i++) putc(s[i]);
        putc('\n');
    }
}

using namespace IO;
const int N=510;
char A[N][N],B[N][N],C[N][N];
int n,m;

int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++)
        for(int j=1;j<=m;j++)
            B[i][j]=C[i][j]='.';
    for(int i=1;i<=n;i++)
    {
        reads(A[i]);
        for(int j=1;j<=m;j++)
            if(A[i][j]=='#') B[i][j]=C[i][j]='#';
    }
    for(int i=2;i<n;i++)
        for(int j=1;j<=m;j+=2)
            B[i][j]='#';
    for(int i=2;i<n;i++)
        for(int j=2;j<=m;j+=2)
            C[i][j]='#';
    for(int i=1;i<=m;i++)
        B[1][i]=C[n][i]='#';
    for(int i=1;i<=n;i++)
        B[i][m+1]='\0',prints(B[i]+1);
    putc('\n');
    for(int i=1;i<=n;i++)
        C[i][m+1]='\0',prints(C[i]+1);
    flush();
    return 0;
}
```

