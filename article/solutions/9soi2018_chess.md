# 【九省联考2018】一双木棋

**题目链接：[一双木棋 - 洛谷](https://www.luogu.org/problemnew/show/P4363)**

先手希望a-b尽可能大，后手希望a-b尽可能小。所以考虑对抗搜索。根据当前是谁的回合，决定当前答案是取max还是取min

记一个pos数组表示每行填到哪个位置了（每行一定是从左往右按顺序填），然后将这个pos数组哈希成一个long long，用map存储就可以记忆化了

状态数显然不会太多，因此跑的很快。unordered_map更是快到飞起

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=15;
int a[N][N],b[N][N];
int pos[N],n,m;
unordered_map<LL,int> f;

int decode(LL stu)
{
    int cnt=0;
    for(int i=n;i>=1;i--)
        pos[i]=stu&15,stu>>=4,cnt+=pos[i];
    return cnt&1;
}

LL compress()
{
    LL stu=0;
    for(int i=1;i<=n;i++)
        stu=stu<<4|pos[i];
    return stu;
}

int dfs(LL stu)
{
    if(f.count(stu)) return f[stu];
    int op=decode(stu);
    int ans=op?INT_MAX:-INT_MAX;
    for(int i=1;i<=n;i++)
    {
        if(pos[i]>=pos[i-1]) continue;
        pos[i]++;
        LL nxt=compress();
        if(op) ans=min(ans,dfs(nxt)-b[i][pos[i]]);
        else ans=max(ans,dfs(nxt)+a[i][pos[i]]);
        pos[i]--;
    }
    return f[stu]=ans;
}

int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++)
        for(int j=1;j<=m;j++)
            scanf("%d",a[i]+j);
    for(int i=1;i<=n;i++)
        for(int j=1;j<=m;j++)
            scanf("%d",b[i]+j);
    LL mx=0;
    for(int i=1;i<=n;i++)
        mx=mx<<4|m;
    f[mx]=0;pos[0]=m;
    int ans=dfs(0);
    printf("%d\n",ans);
    return 0;
}
```