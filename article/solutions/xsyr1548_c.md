# C.看节目

### 题意简述

给定一个长为n的随机置换p，用它对数组a进行重排列，每次重排列将a\[i\]移动到a\[p\[i\]\]。你需要支持两种操作，一种是进行k次重排列，另一种是询问a数组的区间和

数据范围：n≤1e5，k≤1e9，操作次数与n同阶

### 题解

随机置换拥有良好的性质：它的环数量是log n级别的

于是，对每个环分开考虑贡献。考虑对位置分块

设sum\[p\]\[b\]\[x\]表示第x次重排列后，第p个环在前b块中的部分总和是多少。来考虑预处理这个数组

其实每一次重排列，对于一个环来说就是转了一格。假设环p中第u个位置位于前b块，那么，a\[pos\[u\]\],a\[pos\[u-1\]\],...,a\[pos\[u+2\]\],a\[pos\[u+1\]\]就会依次对sum\[p\]\[b\]\[0\],sum\[p\]\[b\]\[1\],...,sum\[p\]\[b\][n-1\]产生贡献。设f\[x\]=1当且仅当环中第x个位置位于前b块，然后做一个循环卷积即可

求出了sum数组，就可以方便地处理询问了。若当前的询问范围是前x个位置，设x所在块为b，枚举所有的环p，然后在sum\[p\]\[b-1\]上找到对应的值计入答案。最后零散的那部分暴力求即可

复杂度懒得分析了，反正算出来块大小设sqrt(n log n)最优

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=998244353;
const int N=300010,M=100;
int omg[N],iomg[N],inv[N];
int A[N],B[N],rev[N];
int dv[N],st[M],ed[M],tot=0;
int n,m,seed,y,bsz,a[N],p[N];
int lpn=0,lpid[N],idx[N];
vector<int> loop[M],sum[M][M];
long long falun=0;
bool vis[N];

int random(int l,int r)
{
    seed=1ll*seed*y%ha;
    return seed%(r-l+1)+l;
}
void make_permutation()
{
    for(int i=1;i<=n;i++)
        p[i]=i,swap(p[random(1,i)],p[i]);
}

void divide()
{
    for(int i=1,cnt=0;i<=n;i++)
    {
        if(!cnt) st[++tot]=i;
        dv[i]=tot;cnt++;
        if(cnt==bsz) ed[tot]=i,cnt=0;
    }
    ed[tot]=n;
}

inline int add(const int &x,const int &y){return x+y>=ha?x+y-ha:x+y;}
inline int mns(const int &x,const int &y){return x-y<0?x-y+ha:x-y;}

int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
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

void NTT(int *A,int n,int d)
{
    for(int i=0;i<n;i++) if(i<rev[i]) swap(A[i],A[rev[i]]);
    for(int i=1;i<n;i<<=1)
    {
        int wn=(d>0)?omg[i]:iomg[i];
        for(int j=0;j<n;j+=(i<<1))
            for(int k=0,w=1;k<i;k++)
            {
                int x=A[j+k],y=1ll*w*A[i+j+k]%ha;
                A[j+k]=add(x,y);
                A[i+j+k]=mns(x,y);
                w=1ll*w*wn%ha;
            }
    }
    if(d<0) for(int i=0;i<n;i++) A[i]=1ll*A[i]*inv[n]%ha;
}

void prework(int x)
{
    int cnt=0;lpn++;
    static int val[N];
    for(;!vis[x];x=p[x])
    {
        loop[lpn].push_back(x);
        val[idx[x]=cnt++]=a[x];
        lpid[x]=lpn;
        vis[x]=1;
    }
    int len=1,l=0;
    while(len<=(cnt<<1)) len<<=1,l++;
    for(int i=0;i<len;i++) rev[i]=(rev[i>>1]>>1)|((i&1)<<l-1);
    for(int b=1;b<=tot;b++)
    {
        memset(A,0,sizeof(int)*(len));
        memset(B,0,sizeof(int)*(len));
        for(int i=0;i<cnt;i++) A[i]=val[i];
        reverse(A,A+cnt);
        for(int i=1;i<=cnt;i++)
            B[i]=(loop[lpn][i-1]<=ed[b]);
        NTT(A,len,1);NTT(B,len,1);
        for(int i=0;i<len;i++)
            A[i]=1ll*A[i]*B[i]%ha;
        NTT(A,len,-1);
        for(int i=0;i<cnt;i++)
            sum[lpn][b].push_back(add(A[i],A[i+cnt]));
    }
}

int query(int x)
{
    if(x==0) return 0;
    int res=0;
    if(dv[x]>1)for(int i=1;i<=lpn;i++)
        res=add(res,sum[i][dv[x]-1][falun%loop[i].size()]);
    for(int i=st[dv[x]];i<=x;i++)
    {
        int d=lpid[i],len=loop[d].size();
        res=add(res,a[loop[d][(idx[i]-falun%len+len)%len]]);
    }
    return res;
}

int main()
{
    scanf("%d%d%d%d",&n,&m,&seed,&y);
    for(int i=1;i<=n;i++) scanf("%d",a+i);
    bsz=sqrt(n*log(n)/log(2));
    divide();make_permutation();NTT_init();
    for(int i=1;i<=n;i++) if(!vis[i]) prework(i);
    int opt,x,y;
    while(m--)
    {
        scanf("%d",&opt);
        if(opt==1) scanf("%d",&x),falun+=x;
        if(opt==2)
        {
            scanf("%d%d",&x,&y);
            int ans=mns(query(y),query(x-1));
            printf("%d\n",ans);
        }
    }
    return 0;
}
```

