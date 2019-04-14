# C. 最大团

### 题意

提答题。给定10张图，点数不超过1400，对每张图求最大团

### 题解

场上代码出了点小锅，del数组没有清空，导致单次判断的复杂度不断翻倍，所以跑的奇慢无比，只拿了63分

这题没什么好方法，前4个点暴力加上一大堆剪枝可以跑过去，后6个点就只能退火了。也许有人会去写神经网络？反正我不会

退火时，随机一个尚未选中的点，然后强制选这个点，把已有答案中会与它冲突的点删掉，得到下一个答案。然后下一个答案的size和当前答案的size按退火公式判断一下是否接受

跑了一晚上，前7个点已AC，后3个点与答案差1，目前得分92，答案包下载链接在文章底部。伸手党们可以随时关注这篇文章，等后3个点跑出来了我会更新答案包的

---

**12.09 update. 跑出了第8个点，目前得分95**

**12.11 update. 已经放弃了。神仙zh跑出了第9个点，非洲人只能吐血……**

---

### 代码

下面是退火的源码，欢迎拿闲置电脑一起来跑。~~反正电脑迟早烧坏~~

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=1410;
int e[N][N],n,m,cnt=0;
vector<int> ans,cur,tmp,del;
bool in[N];

int calc(int u)
{
    tmp.clear();del.clear();
    tmp.push_back(u);
    for(int x : cur)
        if(!e[x][u]) tmp.push_back(x);
        else del.push_back(x);
    if(tmp.size()>ans.size()) ans=tmp;
    return tmp.size();
}
double RAND(){return (double)rand()/RAND_MAX;}
bool accept(double dta,double tem){return dta>0||RAND()<exp(dta/tem);}
void anneal(double tem,double dta,double end)
{
    int res=0;cur.clear();
    memset(in,0,sizeof(in));
    while(tem>end)
    {
        int u=rand()%n+1;
        while(in[u]) u=rand()%n+1;
        int nxt=calc(u);
        if(accept(nxt-res,tem))
        {
            res=nxt;cur=tmp;in[u]=1;
            for(int x : del) in[x]=0;
        }
        tem*=dta;
    }
}

int main()
{
    srand(time(0));
    scanf("%d%d",&n,&m);
    for(int i=1,u,v;i<=m;i++)
    {
        scanf("%d%d",&u,&v);
        e[u][v]=e[v][u]=1;
    }
    for(int i=1;i<=n;i++)
        for(int j=1;j<=n;j++)
            e[i][j]^=1;
    while(true)
    {
        anneal(1e5,1-5*1e-6,1e-2);
        printf("%d\n",ans.size());
        for(int x : ans) printf("%d ",x);
        puts("");
    }
    return 0;
}
```

### 答案包

[点击下载](https://github.com/EbolaEmperor/OI-Learning/raw/master/Non-TraditionalProblems/SubmitAnswer/%5BXSY%231440C%5D%E6%9C%80%E5%A4%A7%E5%9B%A2(95pts)/ans.zip)
