# C. 六

### 题面

![](http://www.ebola.pro/images/xsyr1533_c_1.png)

### 题解

因为质因数很少，所以可以状压dp

状态是一个8进制数，第x位为0表示质因数x尚未在任何数中出现，1-6表示在哪个数里出现，7表示已经在两个数中出现。当然状态还需要第二维，它的意义是：对于已使用的每个质因数，求出它在几个数中出现，然后把求出的结果加起来

于是我们的dp顺序应该以第二维为第一关键字、第一维为第二关键字从小到大排序进行。所以我们可以维护一个set，每次取最小的状态，然后将能扩展到的状态塞进set里面

如果已经决定新加入的数字要出现质因数p，还有考虑这个数包含多少个p，所以要乘上n中包含的p数量

代码好写好调而且好看，不明白的直接看代码即可

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int ha=1e9+7;
struct node
{
    int x,y;
    node(int _x=0,int _y=0):x(_x),y(_y){}
    bool operator < (const node &a) const{return y==a.y?x<a.x:y<a.y;}
};
int p[9],cnt[9],cp=0;
int g[100],f[300000];
set<node> s;
int ans=0,a[9];
LL n;

node calc(int x)
{
    int used=0,tot=0;
    static int b[9],p[9];
    memset(b,0,sizeof(b));
    memset(p,0,sizeof(p));
    for(int i=1;i<=cp;i++)
    {
        b[i]=a[i];
        if(x&(1<<i-1))
        {
            if(!b[i]){b[i]=6;continue;}
            if(!used&&b[i]) used=b[i];
            if(b[i]==7) return node(-1,0);
            if(used&&b[i]&&used!=b[i]) return node(-1,0);
            b[i]=7;
        }
    }
    int stu=0,len=0;p[7]=7;
    for(int i=1;i<=cp;i++) if(b[i]&&!p[b[i]]) p[b[i]]=++tot;
    for(int i=1;i<=cp;i++) b[i]=p[b[i]];
    for(int i=cp;i>=1;i--) stu=stu<<3|b[i];
    for(int i=1;i<=cp;i++) if(b[i]) len+=1+(b[i]==7);
    return node(stu,len);
}

void gao()
{
    s.insert(node(0,0));
    while(!s.empty())
    {
        int x=s.begin()->x,v=x;s.erase(s.begin());
        for(int i=1;i<=cp;i++) a[i]=v&7,v>>=3;
        ans=(ans+f[x])%ha;
        for(int t=1;t<(1<<cp);t++)
        {
            node nxt=calc(t);
            if(nxt.x==-1) continue;
            if(!f[nxt.x]) s.insert(nxt);
            f[nxt.x]=(f[nxt.x]+1ll*f[x]*g[t])%ha;
        }
    }
}

int main()
{
    cin>>n;
    for(LL i=2;i*i<=n;i++)
    {
        if(n%i) continue;
        p[++cp]=i;
        while(n%i==0) cnt[cp]++,n/=i;
    }
    if(n>1) p[++cp]=n,cnt[cp]=1;
    for(int s=1;s<(1<<cp);s++)
    {
        g[s]=1;
        for(int i=1;i<=cp;i++)
            if(s&(1<<i-1)) g[s]=1ll*g[s]*cnt[i]%ha;
    }
    f[0]=1;gao();
    printf("%d\n",ans-1);
    return 0;
}
```

