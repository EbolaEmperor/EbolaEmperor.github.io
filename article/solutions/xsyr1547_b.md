# B. 排行

### 题面

![](http://www.ebola.pro/images/xsyr1547_b_1.png)

要求询问次数不超过11500

### 题解

发现本题的关键在找到排名3这个分界点，但直接找3非常困难

考虑随机三个不同的人a,b,c，如果a觉得b和c都很菜，那a的排名一定<1或=3。不难证明找到一个这样的a，期望随机次数不超过n次，每次随机做两次询问，所以这里期望最多使用2n次询问

找到a之后，可以对每个人都问一遍：你觉得a强不强？如果被问到的人说a强，那他的排名一定>3，否则他的排名一定≤3

对于所有排名>3的人，我们对他们进行归并排序。比较两个人x,y的排名时，只需对着x问y强不强，如果他说不知道，那说明x比y强。归并排序的比较次数是O(n log n)级别的，然而本题中，归并排序的表现相当优秀，几乎只跑了一半的复杂度

对于所有排名≤3的人，我们也可以归并排序。比较两个人x,y的排名时，只需对着x问y强不强，如果他说y菜，那说明x比y强。复杂度同上。假设有m个人的排名<1，那么会有m个人觉得排名1和2的人菜，有m+1个人觉得排名3的菜，所以一通排序之后排名1,2,3的就都到了最后。但它们三个是乱序的，两两组合特判一下就好了

总的询问次数是2n + n log n，得分80到100都是正常的，没过多交几次就行了

```cpp
#include<bits/stdc++.h>
#include "rank.hpp"
using namespace std;

const int N=1010;
int gc,bc,nc,bgn,n;
vector<int> aft3,pre3,rk,sp;
bool done[N];

void rank_aft3(int l,int r)
{
    if(l==r) return;
    int mid=(l+r)/2;
    rank_aft3(l,mid);
    rank_aft3(mid+1,r);
    static int tmp[N];
    int pl=l,pr=mid+1,p=0;
    while(pl<=mid||pr<=r)
    {
        if(pl>mid) tmp[p++]=aft3[pr++];
        else if(pr>r) tmp[p++]=aft3[pl++];
        else if(ask(aft3[pl],aft3[pr])=='n') tmp[p++]=aft3[pl++];
        else tmp[p++]=aft3[pr++];
    }
    for(int i=0;i<p;i++) aft3[l+i]=tmp[i];
}

void rank_pre3(int l,int r)
{
    if(l==r) return;
    int mid=(l+r)/2;
    rank_pre3(l,mid);
    rank_pre3(mid+1,r);
    static int tmp[N];
    int pl=l,pr=mid+1,p=0;
    while(pl<=mid||pr<=r)
    {
        if(pl>mid) tmp[p++]=pre3[pr++];
        else if(pr>r) tmp[p++]=pre3[pl++];
        else if(ask(pre3[pl],pre3[pr])=='b') tmp[p++]=pre3[pl++];
        else tmp[p++]=pre3[pr++];
    }
    for(int i=0;i<p;i++) pre3[l+i]=tmp[i];
}

void gao_aft3()
{
    int a=-1;
    while(a==-1)
    {
        int x=rand()%n,y=rand()%n,z=rand()%n;
        while(x==y||x==z||y==z) y=rand()%n,z=rand()%n;
        if(ask(x,y)=='b'&&ask(x,z)=='b') a=x;
    }
    for(int i=0;i<n;i++)
        if(i!=a&&ask(i,a)=='g') aft3.push_back(i);
    if(!aft3.empty()) rank_aft3(0,aft3.size()-1);
    for(int i=0;i<aft3.size();i++)
        rk[aft3[i]]=i+4,done[aft3[i]]=1;
}

void gao_pre3()
{
    for(int i=0;i<n;i++)
        if(!done[i]) pre3.push_back(i);
    bgn=-pre3.size()+4;
    rank_pre3(0,pre3.size()-1);
    for(int i=0;i<pre3.size()-3;i++) rk[pre3[i]]=i+bgn;
    sp.push_back(pre3[pre3.size()-3]);
    sp.push_back(pre3[pre3.size()-2]);
    sp.push_back(pre3[pre3.size()-1]);
}

void gao_123()
{
    int a=sp[0],b=sp[1],c=sp[2];
    if(ask(a,b)=='b') rk[a]=2,rk[b]=3,rk[c]=1;
    if(ask(a,c)=='b') rk[a]=2,rk[b]=1,rk[c]=3;
    if(ask(b,a)=='b') rk[a]=3,rk[b]=2,rk[c]=1;
    if(ask(b,c)=='b') rk[a]=1,rk[b]=2,rk[c]=3;
    if(ask(c,a)=='b') rk[a]=3,rk[b]=1,rk[c]=2;
    if(ask(c,b)=='b') rk[a]=1,rk[b]=3,rk[c]=2;
}

vector<int> work(int m)
{
    srand(time(0));
    rk.resize(n=m);
    gao_aft3();
    gao_pre3();
    gao_123();
    return rk;
}
```

