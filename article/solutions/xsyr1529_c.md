# 米缸

### 题意简述

有一个n\*m的网格（不超过2000\*2000），每个格子上有一个数字。每次移动，会选择当前位置右边相邻列、行与当前行相邻或相同的三个格子中，取一个数字最大的位置移过去。网格是循环的，也就是说，认为最右边一列与最左边相邻，最上面一行与最下面相邻。

现在，你需要支持两个操作，一个是问你移动k步后到达哪里，另一个是修改某个格子上的数字。操作次数不超过2000次，k会很大

### 题解

忽然发现自己是傻子，这其实就是一道暴力题

记录next(x)表示第1列的第x行往右走m步后到达了第1列的哪一行

对于一次移动，先暴力移动到第1行，然后借助next指针可以一下移动m次，复杂度可以降到O(k/m)。但还是不够。我们发现，沿着next指针不断走，最终会走入一个环，在这个环中一步一步模拟是没有意义的，所以可以先沿next指针暴力走入这个环，然后将剩余的k对**真实环长**取个模（注意next指针的一步实际上是m步，所以真实环长应该是next图环长\*m）。然后剩下的k，先沿next指针走到小于m为止，最后那部分暴力走掉。单次移动的复杂度是O(n+m)

对于一次修改，直接影响到的应该是前一列相邻三个格子的下一步取向。于是我们先更新这三个格子的下一步取向，并暴力求出不断往右走会到第1列的哪一行（记做final）。现在要往左更新一直到第一列。发现对于每一列，受影响的位置在模n意义下必然是一段连续的区间，所以我们可以只管端点。如果(l-1,y-1)的新取向是(l,y)，则l可以向上扩展一位；如果(l,y-1)的新取向是(l-1,y)，则l需要向下缩减一位。右端点也是类似的。往左推到第1列后，就知道有哪些位置的next指针要更新成final了。单次更新复杂度O(m)

这样就得到了优秀的复杂度O(Q(n+m))，比官解少了一个log

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
inline void reads(char *s)
{
    char c=Get();int tot=0;
    while(c<'a'||c>'z') c=Get();
    while(c>='a'&&c<='z') s[tot++]=c,c=Get();
    s[tot]='\0';
}
 
struct pt{int x,y;};
const int N=2010;
int num[N][N],n,m;
int nxt[N],vis[N],p[N];
 
pt operator + (const pt &a,const pt &b){return {(a.x+b.x+n)%n,(a.y+b.y+m)%m};}
pt maxp(const pt &a,const pt &b){return num[a.x][a.y]>num[b.x][b.y]?a:b;}
void move(pt &t){t=maxp(t+(pt){0,1},maxp(t+(pt){-1,1},t+(pt){1,1}));}
void move(pt &t,int k){while(k--)move(t);}
 
void change(pt t)
{
    int l=t.x,r=t.x;
    pt ed=t;move(ed,m-t.y);
    for(int i=t.y;i;i--)
    {
        if(l>r) break;
        int L=(l+n)%n,R=(r+n)%n;
        pt tmp={L,i-1};move(tmp);
        if(tmp.x==(L-1+n)%n) l++;
        tmp={(L-1+n)%n,i-1};move(tmp);
        if(tmp.x==L) l--;
        tmp={R,i-1};move(tmp);
        if(tmp.x==(R+1)%n) r--;
        tmp={(R+1)%n,i-1};move(tmp);
        if(tmp.x==R) r++;
    }
    for(int i=l;i<=r;i++) nxt[(i+n)%n]=ed.x;
}
 
int main()
{
    n=read();m=read();
    for(int i=0;i<n;i++)
        for(int j=0;j<m;j++)
            num[i][j]=read();
    for(int i=0;i<n;i++)
    {
        pt t={i,0};
        move(t,m);
        nxt[i]=t.x;
    }
    char opt[10];pt t={0,0};
    for(int T=read();T;T--)
    {
        reads(opt);
        if(opt[0]=='m')
        {
            int k=read(),cnt=0,x=t.x,st=1;
            while(k&&t.y) move(t),k--;
            while(vis[x]!=T) vis[p[++cnt]=x]=T,x=nxt[x];
            while(nxt[p[cnt]]!=p[st]) st++;
            while(k>=m&&t.x!=p[st]) t.x=nxt[t.x],k-=m;
            k%=m*(cnt-st+1);
            while(k>=m) t.x=nxt[t.x],k-=m;
            while(k) move(t),k--;
            printf("%d %d\n",t.x+1,t.y+1);
        }
        if(opt[0]=='c')
        {
            int a=read(),b=read();
            num[a-1][b-1]=read();
            pt cp={a-1,b-1};
            change(cp+(pt){-1,-1});
            change(cp+(pt){0,-1});
            change(cp+(pt){1,-1});
        }
    }
    return 0;
}
```
