[返回首页](https://EbolaEmperor.github.io)
# 【JSOI2009】有趣的游戏 题解

[我是题目链接](https://www.lydsy.com/JudgeOnline/problem.php?id=1444)

### 题解

·好像正解是高斯消元解方程什么的

·不管它，我们可以用玄学方法A掉这题（谁让他精度要求只到小数点后两位来着）

·初中老师曾经用PPT产生了大量随机数，以此证明“事件发生的次数越多，频率越趋近于概率”，我们可以用到这种思想。所以我们可以用某种神奇方法构造出状态转移方案，然后转移足够多次，得出的就是近似解了

·造出AC自动机后发现结点数量奇少，只有100个，所以我们可以用矩阵乘法来转移

·矩阵的构造方法如下：

	对于词尾结点p，有a[p][p]=1

	对于AC自动机上的一条边i->j，有a[i][j]=j产生的概率

·把这个矩阵自己乘自己，乘上它几十遍，就是近似解了，保留两位就可以A了

```cpp
#include<bits/stdc++.h>
using namespace std;

int ch[110][15],fail[110],sz=0;
bool tag[110];
int n,l,m;
double psb[15];
char s[15];
int word[15];
struct Matrix{double a[110][110];} a;

void insert(char *s,int k)
{
	int p=0,len=strlen(s);
	for(int i=0;i<len;i++)
	{
		int j=s[i]-'A';
		if(!ch[p][j]) ch[p][j]=++sz;
		p=ch[p][j];
	}
	tag[p]=1;word[k]=p;
}

void getfail()
{
	queue<int> q;
	for(int i=0;i<m;i++)
		if(ch[0][i]) q.push(ch[0][i]);
	while(!q.empty())
	{
		int u=q.front();
		tag[u]|=tag[fail[u]];
		for(int i=0;i<m;i++)
			if(ch[u][i])
			{
				int v=ch[u][i];
				fail[v]=ch[fail[u]][i];
				q.push(v);
			}
			else ch[u][i]=ch[fail[u]][i];
		q.pop();
	}
}

Matrix operator * (Matrix &a,Matrix &b)
{
	Matrix c;
	for(int i=0;i<=sz;i++)
		for(int j=0;j<=sz;j++)
		{
			c.a[i][j]=0;
			for(int k=0;k<=sz;k++)
				c.a[i][j]+=a.a[i][k]*b.a[k][j];
		}
	return c;
}

int main()
{
	double p1,p2;
	memset(tag,0,sizeof(tag));
	cin>>n>>l>>m;
	for(int i=0;i<m;i++)
		scanf("%lf%lf",&p1,&p2),psb[i]=p1/p2;
	for(int i=1;i<=n;i++)
		scanf("%s",s),insert(s,i);
	getfail();
	for(int i=0;i<=sz;i++)
		if(tag[i]) a.a[i][i]=1;
		else for(int j=0;j<m;j++) a.a[i][ch[i][j]]+=psb[j];
	for(int i=1;i<=50;i++) a=a*a;
	for(int i=1;i<=n;i++)
		printf("%.2lf\n",a.a[0][word[i]]);
	return 0;
}
```