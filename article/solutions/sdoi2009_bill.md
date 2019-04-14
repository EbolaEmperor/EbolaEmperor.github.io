# 【SDOI2009】Bill的挑战

设match\[p\]\[c\]表示T的第p位为c时，与T匹配的字符串集合，用二进制表示

设f\[p\]\[s\]表示当前考虑第p位，前缀匹配状态是s，此时的方案数是多少

考虑在p+1位置任选一个字母c转移过去，然后s集合与match\[p+1\]\[c\]取与就是转移后的匹配状态，枚举所有字母c，将f\[p\]\[s\]转移过去即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=17,M=55;
const int ha=1000003;
int match[M][27],n,m,k;
int f[M][1<<N];
char s[N][M];

inline void add(int &x,const int &y){x=(x+y>=ha)?(x+y-ha):(x+y);}

int Main()
{
    memset(f,0,sizeof(f));
    memset(match,0,sizeof(match));
    scanf("%d%d",&n,&k);
    for(int i=1;i<=n;i++)
        scanf("%s",s[i]+1);
    m=strlen(s[1]+1);
    for(int j=1;j<=m;j++)
        for(int c=0;c<26;c++)
            for(int i=1;i<=n;i++)
                if(s[i][j]==c+'a'||s[i][j]=='?')
                    match[j][c]|=1<<i-1;
    f[0][(1<<n)-1]=1;
    for(int i=0;i<m;i++)
        for(int s=0;s<(1<<n);s++)
            for(int c=0;c<26;c++)
                add(f[i+1][s&match[i+1][c]],f[i][s]);
    int ans=0;
    for(int s=0;s<(1<<n);s++)
    {
        int cnt=0;
        for(int i=0;i<n;i++)
            if(s&(1<<i)) cnt++;
        if(cnt==k) add(ans,f[m][s]);
    }
    return ans;
}

int main()
{
    int T;scanf("%d",&T);
    while(T--) printf("%d\n",Main());
    return 0;
}
```

