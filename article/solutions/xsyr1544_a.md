# A. Dreamweaver

### 题意简述

这是一道通信题

你需要实现两个程序，第一个程序会得到一个长为8的int序列，然后它需要向第二个程序传递信息，信息由一个长度至少为1000的int序列构成。第二个程序需要利用信息还原出第一个程序得到的序列

第一个程序给第二个程序传信息时，会出现一个“中间人”。第二个程序不能得到信息，而只能向这个中间人获取信息，每次只能获取信息序列的1个int值，而且不知道获取的这个int对应的序列下标是什么，最多获取17次信息

### 题解

假如第一个程序得到的8个int都小于65537，那么，我们可以将这8个数视做一个7次多项式的各项系数，然后将x=0到1000代入，并求得模65537意义下的y值

一个int有32位，用10位来存x，22位来存y，这样一个点值(x,y)就能压缩到1个int里面

这样，第二个程序只要随便知道8个int，然后还原出8个点值，再用待定系数法求出原多项式，就可以知道第一个程序得到的序列了

当序列的值大于65537时，我们可以将一个值拆成两个，第一个是原数/65537，第二个是原数%65537。然后得到一个长为16的新序列，按同样的方法去做，最后合并出原序列即可

注意输入含有负数，为了方便模意义下的运算，统一加上INT_MAX后处理，最后答案减去INT_MAX

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int ha=65537;

static int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=(LL)a*a%ha)
        if(b&1) ans=(LL)ans*a%ha;
    return ans;
}

vector<int> encode(vector<int>arr,int lim)
{
    static int b[16];
    for(int i=0;i<8;i++)
    {
        LL t=arr[i]+(LL)INT_MAX;
        b[i<<1]=t%ha;
        b[i<<1|1]=t/ha;
    }
    vector<int> res;
    for(int x=0;x<1000;x++)
    {
        int y=0;
        for(int j=0,tx=1;j<16;j++)
        {
            y=(y+(LL)tx*b[j])%ha;
            tx=(LL)tx*x%ha;
        }
        res.push_back(y<<10|x);
    }
    return res;
}

vector<int> decode(int(*const arr)(int),int n,int lim)
{
    static int g[16][17];
    static LL ans[16];
    for(int i=0;i<16;i++)
    {
        int a=arr(i),x=a&1023,y=a>>10;
        for(int j=0,tx=1;j<16;j++)
            g[i][j]=tx,tx=(LL)tx*x%ha;
        g[i][16]=y;
    }
    for(int i=0;i<16;i++)
    {
        int p=i;
        while(!g[p][i]) p++;
        for(int j=0;j<17;j++) swap(g[i][j],g[p][j]);
        int inv=Pow(g[i][i],ha-2);
        for(int j=i+1;j<16;j++)
        {
            int t=1ll*g[j][i]*inv%ha;
            for(int k=0;k<17;k++)
                g[j][k]=(g[j][k]-(LL)t*g[i][k]%ha+ha)%ha;
        }
    }
    for(int i=15;i>=0;i--)
    {
        LL c=g[i][16];
        for(int j=i+1;j<16;j++)
            c=(c-ans[j]*g[i][j]%ha+ha)%ha;
        ans[i]=c*Pow(g[i][i],ha-2)%ha;
    }
    vector<int> res;
    for(int i=0;i<8;i++)
        res.push_back(ans[i<<1]+ans[i<<1|1]*ha-INT_MAX);
    return res;
}
```

