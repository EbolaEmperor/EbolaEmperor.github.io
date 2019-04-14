# 【清华集训2014】矩阵变换

卖萌表情？“orz”算不算卖萌（大雾

好的其实这题不需要我们卖萌，因为一定有解啊（不然spj还要智能判断表情萌不萌？

之前闲来无事看《数学一本通》，发现了稳定婚姻这个有趣的问题，然后发现这题似乎就是裸题？

有N行，又有N个数字，我们把行看作男生，把数字看作女生

男生喜欢他那一行出现得更靠左的数字（女生）。女生在每一行出现的位置不一样，她喜欢她出现位置更靠后的行（男生）

然后直接跑Gale-Shapley算法就好了

```cpp
#include<bits/stdc++.h>
using namespace std;

const int S=(1<<20)+5;
char buf[S],*H,*T;
inline char Get()
{
    if(H==T) T=(H=buf)+fread(buf,1,S,stdin);
    if(H==T) return -1;return *H++;
}
inline int read()
{
    int x=0;char c=Get();
    while(!isdigit(c)) c=Get();
    while(isdigit(c)) x=x*10+c-'0',c=Get();
    return x;
}

const int N=210,M=410;
int MLF[N][N],FLM[N][N];
int husband[N],cnt[N];
int wife[N],n,m;

void GaleShapley()
{
    memset(cnt,0,sizeof(cnt));
    memset(wife,0,sizeof(wife));
    memset(husband,0,sizeof(husband));
    queue<int> single_dog;  //这个队列的命名就是为了自我警醒，嗯……
    for(int i=1;i<=n;i++) single_dog.push(i);
    while(!single_dog.empty())
    {
        int boy=single_dog.front();
        int girl=MLF[boy][++cnt[boy]];
        int pre=FLM[girl][husband[girl]];
        int now=FLM[girl][boy];
        if(now>pre)
        {
            pre=husband[girl];
            wife[pre]=0;
            husband[girl]=boy;
            wife[boy]=girl;
            single_dog.pop();
            if(!pre) continue;
            single_dog.push(pre);
        }
    }
}

int main()
{
    for(int T=read();T;T--)
    {
        memset(cnt,0,sizeof(cnt));
        n=read(),m=read();
        for(int i=1;i<=n;i++)
            for(int j=1;j<=m;j++)
            {
                int x=read();
                if(!x) continue;
                MLF[i][++cnt[i]]=x;
                FLM[x][i]=j;
            }
        GaleShapley();
        for(int i=1;i<=n;i++)
            printf("%d ",wife[i]);
        putchar('\n');
    }
    return 0;
}
```

