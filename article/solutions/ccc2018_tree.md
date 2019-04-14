# 【CCC2018】完美平衡树

题目其实就是要你求：

![](http://latex.codecogs.com/svg.latex?f(w)=\sum\limits_{k=2}^wf(\left\lfloor\frac{w}{k}\right\rfloor))

这似乎是一个裸题？直接整除分块，然后递归下去再记忆化一下就完事了？

嗯，实测只能拿到59分。[这是提交记录](https://www.luogu.org/recordnew/show/16436720)

办法总是有的。我们借鉴杜教筛的思想，先预处理一小部分的![](http://latex.codecogs.com/svg.latex?f)值。因为求![](http://latex.codecogs.com/svg.latex?f(w))的复杂度是![](http://latex.codecogs.com/svg.latex?O(\sqrt{w}))，所以预处理的复杂度是![](http://latex.codecogs.com/svg.latex?O(n\sqrt{n}))

然后小心地调一波参，发现![](http://latex.codecogs.com/svg.latex?n=50000)的时候就过了

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N=50000;
unordered_map<int,LL> f;
LL _f[N];

LL F(int w)
{
    if(w<N) return _f[w];
    if(f.count(w)) return f[w];
    LL res=0;
    for(int k=2,dv;k<=w;k=dv+1)
    {
        dv=w/(w/k);
        res+=F(w/k)*(dv-k+1);
    }
    return f[w]=res;
}

int main()
{
    _f[1]=1;
    for(int w=2;w<N;w++)
        for(int k=2,dv;k<=w;k=dv+1)
        {
            dv=w/(w/k);
            _f[w]+=_f[w/k]*(dv-k+1);
        }
    int n;cin>>n;
    cout<<F(n)<<endl;
    return 0;
}
```