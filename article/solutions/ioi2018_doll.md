# 【IOI2018】机械娃娃

如果n是2的幂次减1的话，不难构造出一棵满二叉树，其中叶子是触发器（当然有一个特殊的叶子需要指回到起点），非叶节点是开关。触发器出口统一连到根节点开关，起点出口也连到根节点开关。模拟一下开关切换过程，不难得到叶子的遍历顺序。具体地，我们可以得到形如下图的构造方式：

![](http://www.ebola.pro/images/ioi2018_doll_1.png)

如果n不是2的幂次减1，那么我们可以人为地去挖掉一些子树。例如n=4时，我们可以沿用上图的构造方式，挖去一些子树后得到下图构造：

![](http://www.ebola.pro/images/ioi2018_doll_2.png)

具体实现方式请参考代码

```cpp
#include<bits/stdc++.h>
#include "doll.h"
using namespace std;

const int N=210010;

void create_circuit(int m,vector<int> A)
{
    static bool stu[N];
    static int deep[N];
    vector<int> C,X,Y;
    C.resize(m+1);C[0]=-1;A.push_back(0);
    int n=A.size(),dep=ceil(log2(n)),tot=1<<dep,S=1;
    X.push_back(0);Y.push_back(0);deep[0]=dep;
    for(int x : A)
    {
        int u=0;
        while(true)
        {
            int d=deep[u];
            stu[u]^=1;
            int &v=stu[u]?X[u]:Y[u];
            if(!v)
            {
                if((1<<d-1)<=tot-n) tot-=1<<d-1,v=-1; //挖子树
                else if(d==1){v=x;C[x]=-1;break;} //连接触发器
                else deep[S]=d-1,v=-S-1,S++,X.push_back(0),Y.push_back(0); //新建开关
            }
            u=-(stu[u]?X[u]:Y[u])-1;
        }
    }
    answer(C,X,Y);
}
```