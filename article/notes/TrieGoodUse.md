[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Trie)

# 字典树的妙用——求最大异或值

### 提出问题

给定长度为n的序列A，其中A[i]≤10^9，求A[i] xor A[j] (i≠j)的最大值，n≤10^5

[提交地址](https://www.luogu.org/problemnew/show/U26197)

### 暴力思路

枚举i，j，更新答案，时间复杂度O(n²)

```cpp
#include<bits/stdc++.h>
using namespace std;

int a[100010],n;

int main()
{
  cin>>n;
  for(int i=1;i<=n;i++) scanf("%d",a+i);
  int ans=0;
  for(int i=1;i<n;i++)
    for(int j=i+1;j<=n;j++)
      ans=max(ans,a[i]^a[j]);
  cout<<ans<<endl;
  return 0;
}
```

### 用字典树求解

字典树可以把时间复杂度降低到O(n log max{A})，max{A}表示序列A中元素的最大值

我们举一个例子：给定序列为3 , 5 , 6 , 4

将它们转化为二进制，保留3位，高位补0，得：011 , 101 , 110 , 100

将这些二进制数看成串，建字典树如下

![](http://ebola.blogwo.com/wp-content/uploads/sites/3855/2018/05/%E5%9B%BE%E7%89%871-300x193.png)

显然，建出来的字典树是一棵二叉树

于是我们只需枚举i，而j可以在字典树上走出来，比如a[i]的最高位是1，那查找a[j]的时候，指针在最高位就往0走，如果子树0为空，那就往1走。

举一个例子，我们现在枚举a[i]=4，它的二进制码是100，那我查找时，指针就顺次走0->1->1

指针这样走，得到的xor值一定是最大的，因为我们相当于从高位到低位保证xor的结果中每一位都尽量为1

```cpp
#include<bits/stdc++.h>
using namespace std;

struct Node{Node* ch[2];Node(){ch[0]=ch[1]=NULL;}};
Node* root;
int a[100010],n;

void insert(int x)
{
  Node* p=root;
  for(int i=30;i>=0;i--)
  {
    int j=(x>>i)&1;
    if(p->ch[j]==NULL) p->ch[j]=new Node;
    p=p->ch[j];
  }
}

int query(int x)
{
  int ans=0;
  Node* p=root;
  for(int i=30;i>=0;i--)
  {
    int j=((x>>i)&1)^1;
    if(p->ch[j]) ans|=(1<<i);
    else j^=1;
    p=p->ch[j];
  }
  return ans;
}

int main()
{
  cin>>n;
  root=new Node;
  for(int i=1;i<n;i++) scanf("%d",a+i);
  insert(a[1]);
  int ans=0;
  for(int i=2;i<=n;i++)
  {
    ans=max(ans,query(a[i]));
    insert(a[i]);
  }
  cout<<ans<<endl;
  return 0;
}
```
