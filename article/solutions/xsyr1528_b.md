# B. Dexterity

### 题意简述

两人玩石头剪刀布游戏。A知道了B的出拳序列，但这个序列是一个循环序列，也就是说A不知道B出拳顺序的真实起点位置。A还知道了每局胜或平能获得的分数。假定A是一个非洲人，求A采取最优策略的情况下，能获得多少分

### 题解

首先把串复制一遍，然后建立后缀树（即反串后缀自动机的fail树）

对于一个节点，他到某个子节点的边是压缩的，只要到了这个节点并确定下一个字母，就可以知道这个压缩的串是什么。所以这个压缩的串，除了第一个字符不能保证拿分，其它的字符对应的局是稳赢的

所以直接dp。对于后缀树上的每个节点，记录它在SAM中对应的right集合中最右边的那个点，然后就可以知道自己是第几局了。出拳是可以由自己决定的，但第一个字符是天注定的（然后A又是个非洲人）。所以可以枚举出拳，在三个子节点中取dp最小值，再在所有出拳中取dp最大值。注意加上第一个字符对应的局获胜或平的得分

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=400010;
int len[N],prt[N],rgt[N];
int ch[N][3],lst=1,tot=1;
int s[N],n,w[N],d[N];
LL sumw[N];
char ss[N];

void insert(int c)
{
    int p=lst,np=++tot;rgt[np]=len[np]=len[p]+1;
    while(p&&!ch[p][c]) ch[p][c]=np,p=prt[p];
    if(!p) prt[np]=1;
    else
    {
        int q=ch[p][c];
        if(len[p]+1==len[q]) prt[np]=q;
        else
        {
            int nq=++tot;len[nq]=len[p]+1;rgt[nq]=rgt[np];
            memcpy(ch[nq],ch[q],sizeof(ch[nq]));
            prt[nq]=prt[q];prt[q]=prt[np]=nq;
            while(ch[p][c]==q) ch[p][c]=nq,p=prt[p];
        }
    }
    lst=np;
}

LL dp(int u)
{
    if(!u) return 1ll<<60;
    int now=len[prt[u]];LL f[3],res=0;
    if(len[u]>=n) return sumw[n]-sumw[now+1];
    for(int k=0;k<3;k++) f[k]=dp(ch[u][k]);
    for(int k=0;k<3;k++)
    {
        LL resp=1ll<<60;
        for(int i=0;i<3;i++)
        {
            int tmp=0;
            if(k==i) tmp=d[len[u]+1];
            if((k+1)%3==i) tmp=w[len[u]+1];
            resp=min(resp,f[i]+tmp);
        }
        res=max(res,resp);
    }
    if(len[u]>now) res+=sumw[len[u]]-sumw[now+1];
    return res;
}

LL gao()
{
    for(int i=1;i<=n;i++)
        sumw[i]=sumw[i-1]+w[i];
    memset(ch,0,sizeof(ch));
    for(int i=1;i<=tot;i++)
        ch[prt[i]][s[rgt[i]-len[prt[i]]]]=i;
    return dp(1);
}

int main()
{
    scanf("%d%s",&n,ss+1);
    for(int i=1;i<=n;i++)
        scanf("%d%d",w+i,d+i);
    for(int i=1;i<=n;i++)
    {
        if(ss[i]=='r') s[i]=0;
        if(ss[i]=='s') s[i]=1;
        if(ss[i]=='p') s[i]=2;
        s[i+n]=s[i];
    }
    reverse(s+1,s+2*n+1);
    for(int i=1;i<=2*n;i++)
        insert(s[i]);
    printf("%lld\n",gao());
    return 0;
}
```
