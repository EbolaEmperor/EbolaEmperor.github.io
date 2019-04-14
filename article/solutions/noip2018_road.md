# 【NOIP2018】铺设道路

一年前写过这题，然而完全没有印象了，毕竟当时也没有写题解。考场上一脸懵逼，不过氪了50分钟总算还是做出来了

我们相当于是一层一层地填，对于每一层，有多少个连续段，我们就要填多少下，于是考虑维护每层的连续段数cnt

如果连续的若干层中，没有任何一个新增位置到了顶，那么这连续的若干层中，每层的答案都是一样的。只需找到满足上述性质的最顶层，然后求出答案后乘上跳过的层数即可

不难发现，对于某一层v，若某一段连续的位置都等于v，那cnt加一当且近当这一段连续位置的左右两边均大于v

于是我们可以对每层x开一个vector，存有哪些位置的层数等于x。这样就能保证复杂度为O(n)了

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=100010,D=10010;
int a[N],n;
vector<int> pos[D];

int main()
{
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
    {
        scanf("%d",a+i);
        pos[a[i]].push_back(i);
    }
    int cnt=1,pre=0;
    long long ans=0ll;
    for(int i=0;i<=10000;i++)
    {
        if(pos[i].empty()) continue;
        ans+=1ll*cnt*(i-pre);
        for(int j=0;j<pos[i].size();j++)
        {
            int p=pos[i][j];
            if(a[p-1]>a[p]) cnt++;
            if(j==0||p!=pos[i][j-1]+1) cnt--;
            if(a[p+1]>a[p]) cnt++;
        }
        pre=i;
    }
    cout<<ans<<endl;
    return 0;
}

```