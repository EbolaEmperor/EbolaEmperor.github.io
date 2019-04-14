# B. 电路

### 题面

![](http://www.ebola.pro/images/xsyr1551_b_1.png)

### 题解

不难发现，问题无解当且仅当m<n

有解时，从左往右依次安排，先安排长度大于等于3的，再安排长度为2的，最后安排长度为1的

具体见代码及注释。gao2和gao1就懒得打注释了，看懂了gao3的代码及注释，gao1和gao2一看代码就能秒懂

```cpp
#include<bits/stdc++.h>
using namespace std;
 
const int N=100010;
int n,m,len[N];
queue<int> s1,s2,s3;
int ans[3][N];
int idx,pos,rest; //pos表示当前正在考虑的列编号，idx表示上一根电线结尾在哪一行，rest表示当前列是否有上一根电线的尾巴
 
void gao3()  //安排长度大于等于3的电线
{
    int p=s3.front();s3.pop();
    if(!rest)
    {
        //没有尾巴，直接把这一列绕上
        ans[0][pos]=ans[1][pos]=ans[2][pos]=p;
        len[p]-=3;pos++;idx=0;
    }
    else
    {
        //有尾巴，把剩余两行绕上
        ans[2-idx][pos]=ans[1][pos]=p;
        len[p]-=2;pos++;idx=2-idx;
    }
    while(len[p])
    {
        if(!s2.empty())
        {
            //有长为2的电线。那当前电线尾巴往右伸一列，这一列剩下两行用长为2的电线绕上
            int q=s2.front();s2.pop();
            ans[2-idx][pos]=ans[1][pos]=q;
            ans[idx][pos]=p;
            len[q]-=2;len[p]--;
            pos++;rest=0;
        }
        else if(len[p]>1)
        {
            //没有长为2的电线且当前电线未安排的长度大于1。那当前电线的尾巴伸长2格，穿过竖线绕到另一根横线那儿，然后这一列剩下一根横线放一个长为1的电线
            int q=s1.front();s1.pop();
            ans[1][pos]=ans[2-idx][pos]=p;
            ans[idx][pos]=q;
            len[p]-=2;len[q]--;
            pos++;rest=0;
            idx=2-idx;
        }
        else ans[idx][pos]=p,len[p]--,rest=1;  //当前电线只剩1格未安排，直接尾巴往右伸一列，然后rest设为1
    }
}
 
void gao2()  //安排长度等于2的电线
{
    int p=s2.front();s2.pop();
    if(!rest)
    {
        ans[0][pos]=ans[1][pos]=p;
        len[p]-=2;
        if(!s2.empty())
        {
            int q=s2.front();s2.pop();
            ans[2][pos]=q;ans[2][++pos]=q;
            len[q]-=2;rest=1;idx=2;
        }
        else
        {
            int q=s1.front();s1.pop();
            ans[2][pos]=q;len[q]--;
            rest=0;pos++;
        }
    }
    else
    {
        ans[2-idx][pos]=ans[1][pos]=p;
        len[p]-=2;rest=0;pos++;
    }
}
 
void gao1()  //安排长度等于1的电线
{
    if(!rest)
    {
        int p=s1.front();s1.pop();
        ans[0][pos]=p;len[p]--;
        p=s1.front();s1.pop();
        ans[1][pos]=p;len[p]--;
        p=s1.front();s1.pop();
        ans[2][pos]=p;len[p]--;
        pos++;
    }
    else
    {
        int p=s1.front();s1.pop();
        ans[2-idx][pos]=p;len[p]--;
        p=s1.front();s1.pop();
        ans[1][pos]=p;len[p]--;
        pos++;rest=0;
    }
}
 
int main()
{
    scanf("%d%d",&n,&m);
    if(m<n) return puts("no"),0;
    puts("yes");
    for(int i=1;i<=m;i++)
    {
        scanf("%d",len+i);
        if(len[i]==1) s1.push(i);
        if(len[i]==2) s2.push(i);
        if(len[i]>=3) s3.push(i);
    }
    while(!s3.empty()) gao3();
    while(!s2.empty()) gao2();
    while(!s1.empty()) gao1();
    for(int i=0;i<3;i++,puts(""))
        for(int j=0;j<n;j++)
            printf("%d ",ans[i][j]);
    return 0;
}
```

