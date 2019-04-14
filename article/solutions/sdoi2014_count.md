# 【SDOI2014】数数

题目链接：[3530. 数数  -  BZOJ](https://www.lydsy.com/JudgeOnline/problem.php?id=3530)

对给定的若干个串建立AC自动机。对自动机上的每个节点，维护一个标记，表示在fail树上，它到根的路径中是否存在词尾节点。（词尾节点即每次插入一个串时最后到达的那个节点）

设![](http://latex.codecogs.com/svg.latex?f_{i,j,0/1})表示考虑到n的第i位（从高到低），当前在自动机的j号节点，目前为止 没有挨着 / 正在挨着 上界时，答案是多少。设ch(x,c)表示x号节点的c号子节点，于是可以写出转移：

1. 若当前没有挨着上界，前一位不挨着上界，则：![](http://latex.codecogs.com/svg.latex?f_{i-1,p,0}\longrightarrow&space;f_{i,ch(p,1\sim&space;9),0})
2. 若当前没有挨着上界，前一位挨着上界，则：![](http://latex.codecogs.com/svg.latex?f_{i-1,p,1}\longrightarrow&space;f_{i,ch(p,1\sim&space;upper_i-1),0})
3. 若当前要挨着上界，那前一位必然要挨着上界，则 ：![](http://latex.codecogs.com/svg.latex?f_{i-1,p,1}\longrightarrow&space;f_{i,ch(p,upper_i),1})

最后的答案显然就是：![](http://latex.codecogs.com/svg.latex?\sum_{i=0}^{tot}f_{n,i,0}+f_{n,i,1})

经过尝试，使用滚动数组会大大提升运行速度，可以快20ms左右！

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=1510;
const int ha=1000000007;
int ch[N][10],fail[N],tot=0;
int f[2][N][2],m,nlen;
bool fuck[N];
char n[N];

void insert(char* _begin,char* _end)
{
    int o=0;
    for(;_begin!=_end;_begin++)
    {
        int t=*_begin-'0';
        if(!ch[o][t]) ch[o][t]=++tot;
        o=ch[o][t];
    }
    fuck[o]=1;
}

void getfail()
{
    queue<int> q;q.push(0);
    while(!q.empty())
    {
        int u=q.front();q.pop();
        fuck[u]|=fuck[fail[u]];
        for(int i=0;i<10;i++)
            if(ch[u][i])
            {
                if(u) fail[ch[u][i]]=ch[fail[u]][i];
                q.push(ch[u][i]);
            }
            else ch[u][i]=ch[fail[u]][i];
    }
}

inline void add(int &x,const int &y){x=(x+y>=ha)?x+y-ha:x+y;}

int xi_jin_ping_is_good()
{
    int ans=0,k=0;
    for(int i=1;i<n[0]-'0';i++)
        add(f[k][ch[0][i]][0],1);
    add(f[k][ch[0][n[0]-'0']][1],1);
    for(int i=1;i<nlen;i++,k^=1)
    {
        memset(f[k^1],0,sizeof(f[k^1]));
        for(int j=1;j<10;j++)
            add(f[k^1][ch[0][j]][0],1);
        for(int p=0;p<=tot;p++)
        {
            if(fuck[p]) continue;
            if(f[k][p][0])for(int j=0;j<10;j++)
            {
                if(fuck[ch[p][j]]) continue;
                add(f[k^1][ch[p][j]][0],f[k][p][0]);
            }
            if(f[k][p][1])for(int j=0;j<n[i]-'0';j++)
            {
                if(fuck[ch[p][j]]) continue;
                add(f[k^1][ch[p][j]][0],f[k][p][1]);
            }
            if(fuck[ch[p][n[i]-'0']]) continue;
            add(f[k^1][ch[p][n[i]-'0']][1],f[k][p][1]);
        }
    }
    for(int i=0;i<=tot;i++)
    {
        if(fuck[i]) continue;
        add(ans,f[k][i][0]);
        add(ans,f[k][i][1]);
    }
    return ans;
}

int main()
{
    char s[N];
    scanf("%s%d",n,&m);
    nlen=strlen(n);
    for(int i=0;i<m;i++)
    {
        scanf("%s",s);
        int len=strlen(s);
        insert(s,s+len);
    }
    getfail();
    int ans=xi_jin_ping_is_good();
    printf("%d\n",ans);
    return 0;
}
```

