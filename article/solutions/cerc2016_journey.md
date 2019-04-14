# 【CERC2016】爵士之旅

一道非常繁琐的贪心题，需要高超的实现技巧，否则代码会很丑

在这里先%一下Mys_C_K大爷，他的写法实在是太优美了

先根据给定的这些机票，处理出每种类型机票的最小花费，用哈希表保存一下（懒人用unordered_map）

然后对于一种x与y之间的飞行，记A表示x到y的单程最小花费，B表示y到x的单程最小花费，AB表示x到y的来回最小花费，BA表示y到x的来回最小花费

单程最小花费需要考虑往返机票反而更便宜的情况。来回最小花费需要考虑两张单程反而比一张往返便宜的情况

然后如果AB比BA小，就优先选择x到y往返。注意到一个往返机票买了之后是可以任意时刻免费返程的，所以可以做类似括号匹配的事情，将匹配的位置打上删除标记。然后再选择y到x的往返。最后再处理x到y的单程、y到x的单程

如果AB比BA大，那也是一样的，就把优先顺序换一下就好了。注意到这是完全一样的过程，所以如果AB比BA大，我们可以swap(A,B)，swap(AB,BA)，然后将x与y的意义交换一下，代码就不用再多写一遍了

贪心的正确性显然

```cpp
#include<bits/stdc++.h>
#define FR first
#define SE second
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
inline bool readc()
{
    char c=Get();
    while(c!='O'&&c!='R') c=Get();
    return c=='R';
}
template<class I> inline void upmin(I &x,const I &y){if(y<x) x=y;}

typedef long long LL;
typedef pair<int,int> pii;
const int N=300010;
struct Type
{
    int u,v,ty;
    Type(int _u=0,int _v=0,int _ty=0):u(_u),v(_v),ty(_ty){}
    LL gethash(){return (LL)u*N*2+v*2+ty;}
};

unordered_map<LL,int> h;
map<pii,int> idx;
int n,m,pfm[N],d[N],cnt=0;
vector<int> V[N];
int stk[N];
bool del[N];
LL ans=0;

int val(pii pr,int d)
{
    LL tmp=Type(pr.FR,pr.SE,d).gethash();
    if(!h.count(tmp)) return INT_MAX;
    else return h[tmp];
}

int main()
{
    n=read();m=read();
    for(int i=1;i<=m;i++) pfm[i]=read();
    int tot=read();
    for(int i=1;i<=tot;i++)
    {
        int x=read(),y=read(),z=readc(),w=read();
        if(!h.count(Type(x,y,z).gethash())) h[Type(x,y,z).gethash()]=w;
        else upmin(h[Type(x,y,z).gethash()],w);
    }
    for(int i=1;i<m;i++)
    {
        int x=pfm[i],y=pfm[i+1];
        if(x>y) swap(x,y),d[i]=1;
        if(!idx.count(pii(x,y))) idx[pii(x,y)]=++cnt;
        V[idx[pii(x,y)]].push_back(i);
    }
    for(auto mpr : idx)
    {
        if(!V[mpr.SE].size()) continue;
        pii pr=mpr.FR,ipr=pii(pr.SE,pr.FR);
        LL A=val(pr,0),B=val(ipr,0),AB=val(pr,1),BA=val(ipr,1),dd=0;
        upmin(A,AB);upmin(B,BA);upmin(AB,A+B);upmin(BA,B+A);
        if(AB>BA) swap(A,B),swap(AB,BA),dd=1;
        memset(del,0,sizeof(bool)*V[mpr.SE].size());
        for(int i=0,top=0;i<V[mpr.SE].size();i++)
        {
            if(d[V[mpr.SE][i]]==dd) stk[++top]=i;
            else if(top) del[stk[top]]=del[i]=1,top--,ans+=AB;
        }
        for(int i=0,top=0;i<V[mpr.SE].size();i++)
        {
            if(del[i]) continue;
            if(d[V[mpr.SE][i]]!=dd) stk[++top]=i;
            else if(top) del[stk[top]]=del[i]=1,top--,ans+=BA;
        }
        for(int i=0;i<V[mpr.SE].size();i++)
        {
            if(del[i]) continue;
            if(d[V[mpr.SE][i]]==dd) ans+=A;
            else ans+=B;
        }
    }
    printf("%lld\n",ans);
    return 0;
}
```

