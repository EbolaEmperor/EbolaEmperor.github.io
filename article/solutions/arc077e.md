# 【ARC077E】guruguru

题目链接：[ARC077 E. guruguru](https://arc077.contest.atcoder.jp/tasks/arc077_c)

可以将给定的数列看作若干个线段![](http://latex.codecogs.com/svg.latex?[a_i,a_{i+1}])，若设定点在这个线段内（模意义下），则相当于可以从![](http://latex.codecogs.com/svg.latex?a_i)跳到设定点，然后再走到![](http://latex.codecogs.com/svg.latex?a_{i+1})

于是我们可以将所有线段按左端点排序，然后枚举设定点的位置。对于枚举到的位置，将包含它的线段全部加入即可。这相当于一个扫描线的过程，扫到一个点，将左端点位于该位置的线段全部加入，右端点位于该位置的线段全部删除。然后维护一个当前答案以及当前加入的线段数量即可

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

typedef long long LL;
const int N=100010;
struct Seg{int l,r,extra;} seg[N<<1];
vector<Seg> R[N];
int n,m,a[N],tot=0;

int main()
{
    int cross=0,p=1;
    LL ans=INT64_MAX,cur=0;
    n=read();m=read();
    for(int i=1;i<=n;i++) a[i]=read();
    for(int i=1;i<n;i++)
        if(a[i]<a[i+1])
        {
            seg[++tot]={a[i],a[i+1],0};
            R[a[i+1]].push_back(seg[tot]);
            cur+=a[i+1]-a[i];
        }
        else
        {
            seg[++tot]={a[i],m,0};
            R[m].push_back(seg[tot]);
            seg[++tot]={0,a[i+1],m-a[i]};  //模意义下，[a[i],m]也在该线段左边，算入额外长度
            R[a[i+1]].push_back(seg[tot]);
            cur+=a[i+1]+m-a[i];
        }
    sort(seg+1,seg+1+tot,[](Seg a,Seg b){return a.l<b.l;});
    for(int i=0;i<=m;i++)
    {
        cur-=cross;  //所有已加入的线段都可以往右多跳一步了
        ans=min(ans,cur);
        for(Seg s : R[i])
        {
            cur+=s.extra+i-s.l-1;  //删除线段
            cross--;  //维护线段数量
        }
        while(p<=tot&&seg[p].l<=i)
        {
            Seg s=seg[p];
            cur-=s.extra+i-s.l-1;  //加入线段
            cross++;p++;
        }
    }
    printf("%lld\n",ans);
    return 0;
}
```



