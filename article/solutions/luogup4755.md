# 【Luogu P4755】Beautiful Pair

首先对每个点求出左边第一个比它大的位置、右边第一个比它大的位置。分别记作L\[p\], R\[p\]。这个可以直接用单调栈求出，正反各一遍即可

这样，对于一个位置p，如果选取的左端点在(L\[p\], p]中，右端点在[p, R\[p\])中，那么位置p就是最大值了

假如我们枚举左端点l，那么[p, R\[p\])中的一个右端点r应该满足：a\[l\] \* a\[r\] <= a\[p\]。不妨换一种写法：a\[r\] <= a\[p\] / a\[l\]。对于一个枚举到的左端点，不等式右边的东西是全部已知的，于是问题就变成了求区间[p, R\[p\])中有多少个小于等于a\[p\] / a\[l\]的数，这是主席树的经典应用

如果我们枚举的是右端点，那做法和上面是几乎完全一样的

那么对于一个位置p，我们是该枚举左端点还是右端点？根据启发式的思想，当然是哪个区间短枚举哪个啦，反正算出来答案都是一样的

根据启发式思想分析一波复杂度，不难发现枚举端点均摊一个log，再套上主席树的一个log，最终复杂度是两个log

虽然这题不会出现0，但有0是一件非常好玩的事，我们来思考一下。除0会RE，于是我们从初始序列中把0抹除，并记下原数列中0的数量。因为0作为端点的任何区间都合法，因此可以贡献n \* cnt0的答案。因为两个端点都是0的状况被我们再每个0位置时都考虑了一遍，所以最后还要减去C(cnt0, 2)

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

typedef pair<int,int> pii;
const int N=100010;
int a[N],cnt0=0,tot=0,n;
int ch[N*20][2],sum[N*20],sz=0,rt[N];
int L[N],R[N],Hash[N],hs=0;
stack<pii> st;

void prework()
{
    for(int i=1;i<=tot;i++)
    {
        while(!st.empty()&&st.top().FR<a[i]) st.pop();
        L[i]=st.empty()?1:st.top().SE+1;
        st.push(pii(a[i],i));
    }
    while(!st.empty()) st.pop();
    for(int i=tot;i>=1;i--)
    {
        while(!st.empty()&&st.top().FR<=a[i]) st.pop();
        R[i]=st.empty()?tot:st.top().SE-1;
        st.push(pii(a[i],i));
    }
}

void insert(int &o,int p,int l,int r,int k)
{
    sum[o=++sz]=sum[p]+1;
    if(l==r) return;
    ch[o][0]=ch[p][0];
    ch[o][1]=ch[p][1];
    int mid=(l+r)/2;
    if(k<=mid) insert(ch[o][0],ch[p][0],l,mid,k);
    else insert(ch[o][1],ch[p][1],mid+1,r,k);
}

int query(int L,int R,int l,int r,int x)
{
    if(l==r) return sum[R]-sum[L];
    int mid=(l+r)/2;
    if(x<=mid) return query(ch[L][0],ch[R][0],l,mid,x);
    else return sum[ch[R][0]]-sum[ch[L][0]]+query(ch[L][1],ch[R][1],mid+1,r,x);
}

void xi_jin_ping_is_good()
{
    long long ans=0;
    for(int i=1;i<=tot;i++) Hash[i]=a[i];
    sort(Hash+1,Hash+1+tot);
    hs=unique(Hash+1,Hash+1+tot)-(Hash+1);
    for(int i=1;i<=tot;i++)
        a[i]=lower_bound(Hash+1,Hash+1+hs,a[i])-Hash;
    for(int i=1;i<=tot;i++) insert(rt[i],rt[i-1],1,hs,a[i]);
    for(int i=1;i<=tot;i++)
    {
        if(i-L[i]<=R[i]-i)for(int j=L[i];j<=i;j++)
        {
            int d=Hash[a[i]]/Hash[a[j]];
            int x=lower_bound(Hash+1,Hash+1+hs,d)-Hash;
            if(Hash[x]>d) x--;
            if(x) ans+=query(rt[i-1],rt[R[i]],1,hs,x);
        }
        else for(int j=i;j<=R[i];j++)
        {
            int d=Hash[a[i]]/Hash[a[j]];
            int x=lower_bound(Hash+1,Hash+1+hs,d)-Hash;
            if(Hash[x]>d) x--;
            if(x) ans+=query(rt[L[i]-1],rt[i],1,hs,x);
        }
    }
    ans+=1ll*cnt0*n-1ll*cnt0*(cnt0-1)/2;
    printf("%lld\n",ans%998244353);
}

int main()
{
    n=read();
    for(int i=1;i<=n;i++)
    {
        int x=read();
        if(!x) cnt0++;
        else a[++tot]=x;
    }
    prework();
    xi_jin_ping_is_good();
    return 0;
}
```

