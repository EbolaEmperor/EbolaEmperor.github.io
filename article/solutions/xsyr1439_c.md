# C. 决战

### 题意简述

给定矩阵第一行的所有元素，你需要构造出剩下的所有元素，使得：

![](http://latex.codecogs.com/svg.latex?\sum_{p}(-1)^{\phi(p)}\prod_{i=1}^n M(i,p_i)=1)

其中p是1到n的一个排列，![](http://latex.codecogs.com/svg.latex?\phi(p))表示p的逆序对数量

保证给定的第一行中存在两数互质。你构造时只能使用绝对值不超过2000的整数

### 题解

题中所给的式子就是行列式的另一种特殊定义。所以我们就是要让矩阵的行列式为1

先考虑一个简单的情况，互质的两数恰好是M(1,1)和M(1,2)

考虑高斯消元求行列式的方法。将矩阵消成上三角矩阵后将对角线元素相乘就是矩阵的行列式。所以一个简单的想法就是，令第2行与第1行消去后M(2,2)=1/M(1,1)，第3到n行除主对角线元素为1外其余元素均为0，这样矩阵的行列式就为1了

设![](http://latex.codecogs.com/svg.latex?a=M(1,1),\;\;b=M(1,2),\;\;x=M(2,1),\;\;y=M(2,2))

现在我们需要使得![](http://latex.codecogs.com/svg.latex?y-\frac{x}{a}b=\frac{1}{a})，两边同乘a，得到：![](http://latex.codecogs.com/svg.latex?ay-bx=1)。这是不定方程的基本形式，用扩欧解决即可。因为a与b互质，所以该方程必然有解

上面的做法基于互质两数恰为M(1,1)与M(1,2)。假如互质的数不是这两个呢？

根据行列式的基本性质，交换矩阵的某两行或某两列，行列式的值添一个符号。所以我们可以把这两个互质的数换到M(1,1)和M(1,2)来，根据交换次数看我们要使得新矩阵的行列式值为1还是-1，用上述做法构造出新矩阵，然后交换回去得到原矩阵即可

**正负号的判定细节繁多，务必小心！（我调正负号调了2小时）**

```cpp
#include<bits/stdc++.h>
using namespace std;

int n,h[7],ans[7][7],fg=1;

int exgcd(int a,int b,int &x,int &y)
{
    if(b==0){x=1;y=0;return a;}
    int g=exgcd(b,a%b,x,y);
    int t=x;x=y;y=t-(a/b)*y;
    return g;
}

void swapc(int a,int b)
{
    for(int i=2;i<=n;i++)
        swap(ans[i][a],ans[i][b]);
}

int main()
{
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
        scanf("%d",h+i);
    int a,b,x,y;
    for(int i=1;i<=n;i++)
        for(int j=i+1;j<=n;j++)
            if(__gcd(h[i],h[j])==1) a=i,b=j;
    if(a==2) swap(a,b);
    if(a!=1) fg=-fg;
    if(b!=2) fg=-fg;
    int g=exgcd(h[a],-h[b],y,x);
    if(fg!=g) x=-x,y=-y,g=-g;
    ans[2][1]=x;ans[2][2]=y;
    ans[3][3]=ans[4][4]=ans[5][5]=1;
    swapc(1,a);swapc(2,b);
    for(int i=1;i<=n;i++) printf("%d ",h[i]);puts("");
    for(int i=2;i<=n;i++,puts(""))
        for(int j=1;j<=n;j++)
            printf("%d ",ans[i][j]);
    return 0;
}
```
