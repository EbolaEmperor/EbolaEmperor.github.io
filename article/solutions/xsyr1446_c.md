# C. 树

### 题面

![](http://www.ebola.pro/images/xsyr1446_c_1.png)

### 题解

设左括号表示在当前点加一个左儿子，并走到新加节点；右括号表示往上找到第一个没有右儿子的点，给它加一个右儿子，并走到新加节点。于是一个满足要求的树，就对应一个合法的2n-2长度的括号序列，且括号序列的任意一个前缀不得存在m-1或更多个未匹配的左括号

将括号序列想象成二维平面上的一条折线，横坐标跨度为2n-2，一个左括号相当于走到右上方，右括号相当于走到右下方。如果存在一个经过直线y=-1的点，则它是不合法的，将第一个到达y=-1直线的点及其后面部分关于y=-1翻转，则终点必然位于直线y=-2，所以要选n+1个位置放右括号，方案数就是C(2n,n+1)

同理，经过直线y=m-1的折线也是不合法的，方案数计算方法类似

这样的话，经过一次y=-1，又经过一次y=m-1的方案就被减了两次，要把它加回来，方案数的计算方法还是和上面类似的

还有经过更多次y=-1和y=m-1的，把它们容斥一下即可

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int ha=998244353;
const int N=20000010;
int fac[N],ifac[N],n,m;
 
int Pow(int a,int b)
{
    int ans=1;
    for(;b;b>>=1,a=1ll*a*a%ha)
        if(b&1) ans=1ll*ans*a%ha;
    return ans;
}
 
void Init(int n)
{
    fac[0]=1;
    for(int i=1;i<=n;i++)
        fac[i]=1ll*fac[i-1]*i%ha;
    ifac[n]=Pow(fac[n],ha-2);
    for(int i=n-1;i>=0;i--)
        ifac[i]=1ll*ifac[i+1]*(i+1)%ha;
}
 
inline int C(int n,int m){return n<m?0:1ll*fac[n]*ifac[m]%ha*ifac[n-m]%ha;}
 
int main()
{
    cin>>n>>m;
    n--;Init(n<<1);
    long long ans=C(n<<1,n);
    for(int i=0;i<n;i+=m)
    {
        ans-=C(n<<1,n+i+1);
        ans-=C(n<<1,n+i+m-1);
        ans+=C(n<<1,n+i+m)<<1;
    }
    ans=(ans%ha+ha)%ha;
    cout<<ans<<endl;
    return 0;
}
```