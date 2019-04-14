# A. 线性代数

### 题意

你有一个空串，需要支持前后端插入，每次插入完后回答：回文串数量、本质不同的回文串数量

串长2e5

### 题解

前70分是裸的回文自动机。然而我前50分写挂了，回文自动机重构时对着0到m+3号点清空（m是总操作次数），但令人惊喜的是，我的最外层操作大循环写的是while(m--)，然后就……

回文自动机应该都会，就不讲了。回文自动机的结点数量-2就是本质不同的回文串数量。至于回文串数量，往后加一个字母，增加的回文串数量就是后缀回文串数量，其实也就是对应点的fail树深度，稍微维护一下就好了

现在我们要资瓷前后端插入。于是我们维护两个last指针，指向前后端对应的结点。回文自动机的子节点不存在方向问题，前缀即后缀，不需另外维护前缀fail指针。find函数需要稍作改动，字符串的初始位置应设在中间。需要注意的是，当某个last指针到达的新点长度恰好等于当前串长时，应该把另一个last指针移动过来

要是理解了PAM的精髓，其实看一眼代码就全明白了

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int N=400010;
int s[N],m;
int ch[N][4],sz;
int fail[N],dep[N];
int lst[2],pos[2];
int len[N];
long long ans=0;
 
int newnode(int l){len[++sz]=l;return sz;}
 
void init()
{
    sz=-1;
    pos[0]=m+5;pos[1]=m+4;
    lst[0]=lst[1]=0;
    newnode(0);
    newnode(-1);
    fail[0]=1;
    memset(s,-1,sizeof(s));
}
 
int find(int p,int d)
{
    while(s[pos[d]]!=s[pos[d]+(d?-len[p]-1:len[p]+1)]) p=fail[p];
    return p;
}
 
void insert(int c,int d)
{
    if(d) s[++pos[1]]=c;
    else s[--pos[0]]=c;
    int cur=find(lst[d],d);
    if(!ch[cur][c])
    {
        int now=newnode(len[cur]+2);
        fail[now]=ch[find(fail[cur],d)][c];
        dep[now]=dep[fail[now]]+1;
        if(len[now]==pos[1]-pos[0]+1) lst[d^1]=now;
        ch[cur][c]=now;
    }
    lst[d]=ch[cur][c];
    ans+=dep[lst[d]];
}
 
int main()
{
    scanf("%d",&m);init();
    char opt[10],c[10];
    while(m--)
    {
        scanf("%s%s",opt,c);
        insert(c[0]-'a',opt[0]=='r');
        printf("%lld %d\n",ans,sz-1);
    }
    return 0;
}
```