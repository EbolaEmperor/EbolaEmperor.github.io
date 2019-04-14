# 【WC2019】远古计算机

一道轻松愉快的小水题，只要会最短路就能做

别看题面上说了那么多指令，有用的其实就4个：read,write,add,jmp

接下来就分别说一下每个子任务的做法

### 任务一

评分参数：200,300,10000

注意到执行完最后一句后会回到第一句执行，因此直接read/write即可

答案如下：

```
node 1
read 0 a
write a 0
```

### 任务二

评分参数：4,5,50

显然需要一个O(1)的算法才能通过

直接打表，将第0到44个fib数打出来，然后读入k后直接jump到对应的行输出即可

以下是答案的生成代码：

```cpp
#include<bits/stdc++.h>
using namespace std;

int main()
{
    freopen("oldcomputer2.out","w",stdout);
    printf("node 1\n");
    printf("read 0 a\n");
    printf("add a 4\n");
    printf("jmp a\n");
    static int f[45];
    f[0]=0;f[1]=1;
    for(int i=2;i<=45;i++)
        f[i]=f[i-1]+f[i-2];
    for(int i=0;i<=45;i++)
        printf("write %d 0\n",f[i]);
}
```

### 任务三

评分参数：211,422,5000

实际上是要将一些信息从1传到n。注意到多路传输并不能减少轮数，因为最后肯定会卡在某个点，因此我们选择单路传输

单路传输的最快方案就是找一条1到n的最短路，因此直接跑单源最短路即可

以下是答案的生成代码：

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=110;
vector<int> g[N];
bool inq[N];
int dis[N],pre[N],n,m;

void spfa()
{
    memset(dis,0x3f,sizeof(dis));
    queue<int> q;
    q.push(1);dis[1]=0;
    while(!q.empty())
    {
        int u=q.front();
        for(int v : g[u])
            if(dis[u]+1<dis[v])
            {
                pre[v]=u;
                dis[v]=dis[u]+1;
                if(inq[v]) continue;
                inq[v]=1;q.push(v);
            }
        q.pop();inq[u]=0;
    }
    for(int i=n,nxt=0;i;i=pre[i])
    {
        printf("node %d\n",i);
        printf("read %d a\n",pre[i]);
        printf("write a %d\n",nxt);
        nxt=i;
    }
}

int main()
{
    freopen("oldcomputer3.in","r",stdin);
    freopen("oldcomputer3.out","w",stdout);
    int qwq;
    scanf("%d%d%d",&qwq,&n,&m);
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        g[u].push_back(v);
        g[v].push_back(u);
    }
    spfa();
}
```

### 任务四

评分参数：7,12,200

这个任务要求我们进行多路传输

注意到有1000个点，因此我们很容易能找到非常短的传输路径。我们可以跑一遍多源最短路（51到100号节点为源），然后将1到50号节点入队进行BFS。对于一个节点u，若存在一个节点v满足dis\[u\]=dis\[v\]+1，则将u的全部信息传递给v

因为一个节点可能需要处理多路信息，因此我们在BFS时对传递的信息加一个时间戳，一个点有多路信息时，按时间顺序处理这些信息

以下是答案生成代码：

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef pair<int,int> pii;
const int N=1010;
vector<int> g[N];
vector<pii> msg[N];
int dis[N],n,m;
bool inq[N];

void spfa()
{
    queue<int> q;
    memset(dis,0x3f,sizeof(dis));
    for(int i=51;i<=100;i++)
    {
        dis[i]=0;
        inq[i]=1;
        q.push(i);
    }
    while(!q.empty())
    {
        int u=q.front();
        for(int v : g[u])
            if(dis[u]+1<dis[v])
            {
                dis[v]=dis[u]+1;
                if(inq[v]) continue;
                inq[v]=1;q.push(v);
            }
        q.pop();inq[u]=0;
    }
}

int main()
{
    freopen("oldcomputer4.in","r",stdin);
    freopen("oldcomputer4.out","w",stdout);
    int qwq;
    scanf("%d%d%d",&qwq,&n,&m);
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        g[u].push_back(v);
        g[v].push_back(u);
    }
    spfa();
    queue<int> q;
    for(int i=1;i<=50;i++)
    {
        q.push(i);
        msg[i].push_back(pii(0,0));
    }
    while(!q.empty())
    {
        int u=q.front(),v=0;q.pop();
        for(int vv : g[u]) if(dis[u]==dis[vv]+1) v=vv;
        printf("node %d\n",u);
        sort(msg[u].begin(),msg[u].end());
        for(pii pr : msg[u])
        {
            printf("read %d a\n",pr.second);
            printf("write a %d\n",v);
            msg[v].push_back(pii(pr.first+1,u));
        }
        if(v) q.push(v);
    }
    return 0;
}
```

### 任务五

评分参数：21,22,31

还是多路信息传递，不过这次我们有了明确的信息传递目标

我们可以对每个节点，记录一下占用情况。具体地，当bad\[u\]\[t\]=1时，说明节点u在时间点t时被占用

这样，在我们跑最短路时，若当前节点是u，需要用他的距离去更新v。w可以用来更新dis\[v\]，当且仅当w和w+1时刻，v节点均为空闲状态。因此我们可以设w初值为dis\[u\]+1，然后不断自增直到满足上述条件，再来更新dis\[v\]

跑完一遍最短路后，要记得更新s到t最短路径上的所有点的占用状态

按上述方法跑10遍单源最短路即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=210;
struct MSG{int from,to,time;};
vector<int> g[N];
vector<MSG> msg[N];
bool bad[N][N],inq[N];
int dis[N],pre[N],n,m,ans=0;

bool operator < (const MSG &a,const MSG &b){return a.time<b.time;}

void spfa(int s,int t)
{
    memset(pre,0,sizeof(pre));
    memset(dis,0x3f,sizeof(dis));
    queue<int> q;q.push(s);
    dis[s]=0;inq[s]=1;
    while(!q.empty())
    {
        int u=q.front();
        for(int v : g[u])
        {
            int w=dis[u]+1;
            while(bad[v][w]||bad[v][w+1]) w++;
            if(w<dis[v])
            {
                dis[v]=w;pre[v]=u;
                if(inq[v]) continue;
                inq[v]=1;q.push(v);
            }
        }
        q.pop();inq[u]=0;
    }
    for(int i=t,nxt=0;i;i=pre[i])
    {
        bad[i][dis[i]]=bad[i][dis[i]+1]=1;
        msg[i].push_back((MSG){pre[i],nxt,dis[i]});
        nxt=i;
    }
    ans=max(ans,dis[t]);
}

int main()
{
    freopen("oldcomputer5.in","r",stdin);
    freopen("oldcomputer5.out","w",stdout);
    int qwq;
    scanf("%d%d%d",&qwq,&n,&m);
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        g[u].push_back(v);
        g[v].push_back(u);
    }
    for(int i=1;i<=10;i++) bad[i][0]=bad[i][1]=1;
    for(int i=1;i<=10;i++) spfa(i,101-i);
    for(int i=1;i<=n;i++)
    {
        if(msg[i].empty()) continue;
        sort(msg[i].begin(),msg[i].end());
        printf("node %d\n",i);
        for(MSG pp : msg[i])
        {
            printf("read %d a\n",pp.from);
            printf("write a %d\n",pp.to);
        }
    }
    cerr<<ans+2<<endl;
    return 0;
}
```
