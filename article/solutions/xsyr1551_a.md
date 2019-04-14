# A. 选举

### 题面

![](http://www.ebola.pro/images/xsyr1551_a_1.png)

### 题解

大致构造方式：

```
1
1
2 1
2 3
3
3
4 3
4 5
5
5
6 5
6 7
...
```

最后一行再来个1，这样第一轮结束后因为1票多所以3号选民改变主意，第二轮结束后因为2少了一票所以4号选民改变主意，第三轮结束后因为3多了一票所以7号选民改变主意……这样就会依次进行n/2轮

```cpp
#include<bits/stdc++.h>
using namespace std;

int main()
{
    puts("999 500");
    for(int i=1;i<=499;i++)
        if(i&1) printf("1 %d\n1 %d\n",i,i);
        else printf("2 %d %d\n2 %d %d\n",i,i-1,i,i+1);
    puts("1 1");
    return 0;
}
```

