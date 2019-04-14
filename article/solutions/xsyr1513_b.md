# B. 斗地主

### 题目大意

给你一幅牌，按斗地主的规则出牌，但不能出单牌。请问最后没法继续出牌时，手上的次小牌最大是多少。如果存在一种出牌方式，能使得最后手上没有次小牌（即全部出完或只剩一张），则输出-1

牌的数量不超过20张

### 题解

很多出牌方式是没用的

有用的只有五种：顺子、三带一、四带两单、对子、三张

至于其它的出法，不难发现，全都可以用上面的若干种方式组合得到。当然王炸比较特殊，我们将大小王看作同一张牌，当成对子就好了。然后最后如果次小值是王，那说明你手上一开始只有一张王，记下是哪张就可以输出了

然后直接暴搜即可

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=25;
char poker[N];
int cnt[N];
int n,ans=-1;
bool big=0;

void Init()
{
    for(int i=1;i<=n;i++)
    {
        char c=poker[i];
        if(isdigit(c)&&c>'2') cnt[c-'0']++;
        else if(c=='T') cnt[10]++;
        else if(c=='J') cnt[11]++;
        else if(c=='Q') cnt[12]++;
        else if(c=='K') cnt[13]++;
        else if(c=='A') cnt[14]++;
        else if(c=='2') cnt[15]++;
        else cnt[16]++;
        if(c=='W') big=1;
    }
}

void print(int c)
{
    if(c==100) puts("-1");
    else if(c<10) printf("%d\n",c);
    else if(c==10) puts("T");
    else if(c==11) puts("J");
    else if(c==12) puts("Q");
    else if(c==13) puts("K");
    else if(c==14) puts("A");
    else if(c==15) puts("2");
    else puts(big?"W":"w");
}

void dfs(int rest)
{
    // 顺子
    for(int i=3;i<=10;i++)
        for(int j=5;i+j-1<=14;j++)
        {
            bool flag=0;
            for(int k=i;k<i+j;k++)
                if(!cnt[k]) flag=1;
            if(flag) break;
            for(int k=i;k<i+j;k++) cnt[k]--;
            dfs(rest-j);
            for(int k=i;k<i+j;k++) cnt[k]++;
        }

    // 三带一 或 三张
    for(int i=3,j;i<=16;i++)
        if(cnt[i]>=3)
        {
            cnt[i]-=3;
            for(j=3;j<=16;j++)
                if(cnt[j]==1){cnt[j]--;break;}
            dfs(j<=16?rest-4:rest-3);
            cnt[i]+=3;
            if(j<=16) cnt[j]++;
        }
    
    // 四带二
    for(int i=3,j;i<=16;i++)
        if(cnt[i]==4)
        {
            cnt[i]-=4;
            int flag=0;
            int fk[3],tt=0;
            for(int j=3;j<=16;j++)
                if(cnt[j]==1)
                {
                    cnt[j]--;
                    fk[++tt]=j;
                    if(++flag>=2)break;
                }
            if(flag<2)
                for(int j=3;j<=16;j++)
                    if(cnt[j])
                    {
                        cnt[j]--;
                        fk[++tt]=j;
                        break;
                    }
            dfs(rest-6);
            for(int j=1;j<=tt;j++) cnt[fk[j]]++;
            cnt[i]+=4;
        }
    
    // 对子
    for(int i=3;i<=16;i++)
        if(cnt[i]>=2)
        {
            cnt[i]-=2;
            dfs(rest-2);
            cnt[i]+=2;
        }

    // 全都是单牌了
    bool finished=1;int mn=0,mns=0;
    for(int i=3;i<=16;i++)
    {
        if(cnt[i]>1) finished=0;
        if(cnt[i]==1&&!mn) mn=i;
        else if(cnt[i]==1&&!mns) mns=i;
    }
    if(finished){ans=max(ans,mns?mns:100);return;}
}

int main()
{
    scanf("%d%s",&n,poker+1);
    Init();dfs(n);
    print(ans);
    return 0;
}
```

