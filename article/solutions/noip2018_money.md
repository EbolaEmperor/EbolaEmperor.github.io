# 【NOIP2018】货币系统

观察样例，猜测我们需要选择的集合一定是给定集合的一个子集。这个结论似乎无法证明，但考场上也没有太多选择了，只好假装它是对的

那么要选尽可能少，肯定就要选尽可能小的数，如果某个较大的数可以通过某几个较小的数按照规则得到，那么那个较大的数就是没用的

那就不难想到去枚举。从小到大枚举给定集合中的数p，然后对于一个当前可以表达出的数x，将所有的x+kp标记为可以表达出来。一开始默认只有0能表达出来。枚举p的时候，如果p已经可以表达出来了，就不选这个数，也就进行上述操作

为了保证复杂度的正确性，我们需要添加done标记。进行上述操作时，先将所有done标记清空，然后每将一个数标记为可表达，就顺便将这个数的done标记设为1。然后我们在枚举x之后对x+kp进行标记时，如果x+kp的done标记为1，那可以直接break出去，因为后面的数一定已经标记完了

总的复杂度就是O(T\*ans\*maxv)

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010,D=25010;
int a[N],n,T;
bool cann[D],done[D];

void gao(int x)
{
    cann[x]=1;
    for(int k=0;k<=a[n];k++)
    {
        if(!cann[k]) continue;
        if(done[k]) continue;
        for(int i=1;k+x*i<=a[n];i++)
        {
            if(done[k+x*i]) break;
            cann[k+x*i]=1;
            done[k+x*i]=1;
        }
    }
}

int main()
{
    cin>>T;
    while(T--)
    {
        cin>>n;
        for(int i=1;i<=n;i++)
            scanf("%d",a+i);
        int tot=0;
        memset(cann,0,sizeof(cann));
        sort(a+1,a+1+n);
        for(int i=1;i<=n;i++)
        {
            if(cann[a[i]]) continue;
            memset(done,0,sizeof(done));
            gao(a[i]);tot++;
        }
        cout<<tot<<endl;
    }
    return 0;
}

```