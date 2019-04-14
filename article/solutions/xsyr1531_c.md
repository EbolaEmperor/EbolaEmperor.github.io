# C. 地转偏向力

### 题意简述

你有一个n\*n的网格，其中n是5的倍数。当你位于(x,y)时，下一步可以跳到(x+3,y),(x-3,y),(x,y+3),(x,y-3),(x-2,y-2),(x-2,y+2),(x+2,y-2),(x+2,y+2)。请你求一个从(1,1)出发，经过每个格子恰好一次，最终回到(1,1)的方案

### 题解

将n\*n的方格拆成若干个5\*5的单元来考虑。我们需要构造方格之间的转移

考虑在一个5\*5的单元中的一条移动路径，一定存在某相邻的两步，满足那两个位置p1、p2均可以跳一步到达某个特定的相邻单元。于是我们可以到达p1时，跳到那个特定的相邻单元，p2就相当于是我们预留的返回接口，从那个特定单元回来时，可以通过这个接口走完尚未完成的路径。这么一来，我们相当于钦定了从哪个格子进入那个特定单元、从哪个格子离开那个特定单元

那么我们就可以蛇形地从左上角地单元走到右下角的单元，最后沿着预留的接口返回即可

我们需要处理出一个表。在5\*5方格内，对于每对给定的起点(sx,sy)和终点(tx,ty)，求出一条经过每个格子恰好一次的路径。这样我们进行主过程时就不需要重新搜了

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int S=1<<20;
char obuf[S],*oS=obuf,*oT=oS+S-1,c,qu[55];int qr;
inline void flush(){fwrite(obuf,1,oS-obuf,stdout);oS=obuf;}
inline void putc(char x){*oS++ =x;if(oS==oT) flush();}
template <class I>inline void print(I x)
{
    if(!x) putc('0');
    while(x) qu[++qr]=x%10+'0',x/=10;
    while(qr) putc(qu[qr--]);
}
 
const int N=1010;
int a[6][6][6][6][6][6];
int sx,sy,tx,ty,vis[6][6];
int ans[N][N],n,m,cnt=0;
bool done[210][210];
 
bool check(int x,int y)
{
    if(x<1||x>5) return 0;
    if(y<1||y>5) return 0;
    if(vis[x][y]) return 0;
    return 1;
}
 
bool predfs(int x,int y,int d)
{
    if(d==25)
    {
        if(x!=tx||y!=ty) return 0;
        memcpy(a[sx][sy][tx][ty],vis,sizeof(vis));
        return 1;
    }
    vis[x][y]=d;
    if(check(x,y-3)&&predfs(x,y-3,d+1)) return 1;
    if(check(x,y+3)&&predfs(x,y+3,d+1)) return 1;
    if(check(x-3,y)&&predfs(x-3,y,d+1)) return 1;
    if(check(x+3,y)&&predfs(x+3,y,d+1)) return 1;
    if(check(x-2,y-2)&&predfs(x-2,y-2,d+1)) return 1;
    if(check(x-2,y+2)&&predfs(x-2,y+2,d+1)) return 1;
    if(check(x+2,y-2)&&predfs(x+2,y-2,d+1)) return 1;
    if(check(x+2,y+2)&&predfs(x+2,y+2,d+1)) return 1;
    vis[x][y]=0;
    return 0;
}
 
void gao(int kx,int ky,int sx,int sy,int tx,int ty)
{
    done[kx][ky]=1;
    int stx=(kx-1)*5,sty=(ky-1)*5;
    int typ=0,pos[26]={0};
    if(kx<m&&!done[kx+1][ky]) typ=1;
    else if(kx>1&&!done[kx-1][ky]) typ=2;
    else if(ky<m) typ=3;
    for(int i=1;i<=5;i++)
        for(int j=1;j<=5;j++)
            pos[a[sx][sy][tx][ty][i][j]]=i<<3|j;
    for(int i=1;i<=25;i++)
    {
        int x=pos[i]>>3,y=pos[i]&7;
        ans[stx+x][sty+y]=++cnt;
        if(!typ) continue;
        int nxtx=pos[i+1]>>3,nxty=pos[i+1]&7;
        if(typ==1&&x>=3&&nxtx>=3) gao(kx+1,ky,x-2,y,nxtx-2,nxty),typ=0;
        if(typ==2&&x<=3&&nxtx<=3) gao(kx-1,ky,x+2,y,nxtx+2,nxty),typ=0;
        if(typ==3&&y>=3&&nxty>=3) gao(kx,ky+1,x,y-2,nxtx,nxty-2),typ=0;
    }
}
 
int main()
{
    for(sx=1;sx<=5;sx++)
        for(sy=1;sy<=5;sy++)
            for(tx=1;tx<=5;tx++)
                for(ty=1;ty<=5;ty++)
                {
                    memset(vis,0,sizeof(vis));
                    predfs(sx,sy,1);
                    a[sx][sy][tx][ty][tx][ty]=25;
                }
    scanf("%d",&n);m=n/5;
    memset(vis,0,sizeof(vis));
    gao(1,1,1,1,3,3);
    for(int i=1;i<=n;i++,putc('\n'))
        for(int j=1;j<=n;j++)
            print(ans[i][j]),putc(' ');
    flush();
    return 0;
}

```