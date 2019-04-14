# 【九省联考2018】秘密袭击

**题目链接：[秘密袭击 - 洛谷](https://www.luogu.org/problemnew/show/P4365)**

正解太难了，我们考虑暴力撵标算 ~~（请一本正经地读这句话）~~

枚举每个节点，钦点他就是第k大的数，然后比它大的数设为1，其它设为0，然后跑树形背包dp求出大小为k-1的背包有多少种方案

单次树形背包dp是O(n²k)的，n次就是O(n²k²)，显然会gg

然后就有一种新方法。钦点每个节点继承父节点及位于它到根的路径左侧的所有子节点的dp数组，最后再贡献回父节点去。这样单次就是O(nk)的了

复杂度O(n²k)，还是会TLE。前k大的数显然没必要跑dp，只要从第k大的开始跑就行了，此时复杂度O((n-k)nk)。再优化一下取模，以及memset不要对整个数组清空，而是只清空大小为n\*k的一块，然后就可以卡过去了。如果还过不去，那就别用for循环继承数组，memcpy更快

```cpp
#include<bits/stdc++.h>
using namespace std;

const int ha=64123;
const int N=1680;
vector<int> G[N];
int n,k,w,d[N],idx[N],f[N][N];

void add(int &x,const int &y){x=x+y>=ha?x+y-ha:x+y;}
void dfs(int u,int fa,int rt)
{
    if(fa) memcpy((d[u]>d[rt]||d[u]==d[rt]&&u<rt)?(f[u]+1):f[u],f[fa],sizeof(int)*(k+1));
    for(int v : G[u]) if(v!=fa) dfs(v,u,rt);
    for(int i=0;i<k;i++) add(f[fa][i],f[u][i]);
}

int main()
{
    scanf("%d%d%d",&n,&k,&w);
    for(int i=1;i<=n;i++) scanf("%d",d+i);
    for(int i=1,u,v;i<n;i++)
    {
        scanf("%d%d",&u,&v);
        G[u].push_back(v);
        G[v].push_back(u);
    }
    for(int i=1;i<=n;i++) idx[i]=i;
    stable_sort(idx+1,idx+1+n,[](int x,int y){return d[x]>d[y];});
    int ans=0;
    for(int i=k;i<=n;i++)
    {
        int u=idx[i];
        for(int i=1;i<=n;i++)
            memset(f[i],0,sizeof(int)*(k+1));
        f[u][0]=1;dfs(u,0,u);
        ans=(ans+1ll*d[u]*f[u][k-1])%ha;
    }
    printf("%d\n",ans);
    return 0;
}
```