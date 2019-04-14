# 【AGC019D】Shift and Flip

[AGC019 D. Shift and Flip  -  AtCoder](https://agc019.contest.atcoder.jp/tasks/agc019_d)

看到这个n的范围就开心了，可以枚举最后A移动多少位与B对应

对于每个移动方案，统计移动后A与B不相同的位置，然后要搞这些位置。设位置数为x，移动步数为k，则这个移动方案的基准代价就是x+k

如果路上经过了B中含1的位置，那就没有额外代价了。否则就需要往反方向移动一些右移回来，或移动到位后继续移动几步再移回来。所以对于每个位置，分别算出它左边和右边第一个满足B\[i\]=1的位置i是多少，然后就可以计算出搞定每一位需要额外移动多少

对于每个位置，计算向左需要移动多少，以及向右需要移动多少，然后计算额外代价最小值即可

不必对左移和右移分别写一段代码，直接钦定只能右移做一遍，再把串反转过来再做一遍就行了，答案取最小值

还有不动的就看代码吧，比较简短

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N=2010;
int A[N<<2],B[N<<2];
int sa[N<<2],sb[N<<2];
int nxl[N<<2],nxr[N<<2];
char ss1[N],ss2[N];
int len,ans=INT_MAX;
int R[N<<2];

void xi_jin_ping_is_good()
{
    nxl[0]=-1;nxr[3*len-1]=3*len;
    for(int i=1;i<3*len;i++) nxl[i]=B[i]?i:nxl[i-1];
    for(int i=3*len-2;i>=0;i--) nxr[i]=B[i]?i:nxr[i+1];
    for(int k=0;k<len;k++)
    {
        int cnt=0,dafa=INT_MAX,mxr=0;
        memset(R,0,sizeof(R));
        for(int i=len;i<2*len;i++)
        {
            if(A[i]==B[i+k]) continue;
            cnt++;R[i-nxl[i]]=max(R[i-nxl[i]],nxr[i]-i-k);
        }
        if(cnt==0){ans=min(ans,k);continue;}
        for(int i=len-1;i>=0;i--)
            dafa=min(dafa,mxr+i),mxr=max(mxr,R[i]);
        ans=min(ans,cnt+k+dafa*2);
    }
}

int main()
{
    scanf("%s",ss1);len=strlen(ss1);scanf("%s",ss2);
    for(int i=0;i<len;i++) A[i]=A[i+len]=A[i+2*len]=ss1[i]-'0';
    for(int i=0;i<len;i++) B[i]=B[i+len]=B[i+2*len]=ss2[i]-'0';
    sa[0]=A[0];sb[0]=B[0];
    for(int i=1;i<3*len;i++) sa[i]=sa[i-1]+A[i],sb[i]=sb[i-1]+B[i];
    if(!sb[len-1]&&sa[len-1]) return puts("-1"),0;
    xi_jin_ping_is_good();
    reverse(A,A+3*len);
    reverse(B,B+3*len);
    xi_jin_ping_is_good();
    printf("%d\n",ans);
    return 0;
}
```

