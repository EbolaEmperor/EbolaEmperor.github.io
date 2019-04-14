[返回首页](https://EbolaEmperor.github.io)
[返回专题](https://EbolaEmperor.github.io/special/Strling)

# 斯特林数 学习笔记

## 第一类斯特林数

### 基本概念

第一类斯特林数表示的是将n个不同元素排列成k个不同的环的方案数（可以通过旋转得到的两个环视为相同的环），记作：![](http://latex.codecogs.com/svg.latex?\begin{bmatrix}n\\&amp;space;k\end{bmatrix})

读作：n轮换k

定义 ![](http://latex.codecogs.com/svg.latex?\begin{bmatrix}&amp;space;0\\&amp;space;0&amp;space;\end{bmatrix}&amp;space;=1,&amp;space;\begin{bmatrix}&amp;space;n\\&amp;space;0&amp;space;\end{bmatrix}&amp;space;=0(n&gt;0))

它不像组合数一样有直接计算公式，而是只能通过递推得到。其递推公式为：

![](http://latex.codecogs.com/svg.latex?\begin{bmatrix}&amp;space;n\\&amp;space;k&amp;space;\end{bmatrix}&amp;space;=&amp;space;\begin{bmatrix}&amp;space;n-1\\&amp;space;k-1&amp;space;\end{bmatrix}&amp;space;+(n-1)&amp;space;\begin{bmatrix}&amp;space;n-1\\&amp;space;k&amp;space;\end{bmatrix})

### 特殊性质

1. ![](http://latex.codecogs.com/svg.latex?\begin{bmatrix}&amp;space;n\\&amp;space;1&amp;space;\end{bmatrix}&amp;space;=(n-1)!)

2. ![](http://latex.codecogs.com/svg.latex?\begin{bmatrix}&amp;space;n\\&amp;space;n-1&amp;space;\end{bmatrix}&amp;space;=C_n^2)

3. ![](http://latex.codecogs.com/svg.latex?\sum_{k=0}^n\begin{bmatrix}&amp;space;n\\&amp;space;k&amp;space;\end{bmatrix}&amp;space;=n!)

4. 不知道它跟斯特林数有什么关系，但做到了这个题，题解说是什么斯特林公式：![](http://latex.codecogs.com/svg.latex?n!\approx&amp;space;\sqrt{2\pi&amp;space;n}\left&amp;space;(&amp;space;\frac{n}{e}&amp;space;\right&amp;space;)^n)

## 第二类斯特林数

### 基本概念

第二类斯特林数表示的是将n个不同元素划分成k个子集的方案数，记作：![](http://latex.codecogs.com/svg.latex?\begin{Bmatrix}&amp;space;n\\&amp;space;k&amp;space;\end{Bmatrix})

读作：n子集k

定义 ![](http://latex.codecogs.com/svg.latex?\begin{Bmatrix}&amp;space;0\\&amp;space;0&amp;space;\end{Bmatrix}&amp;space;=1,&amp;space;\begin{Bmatrix}&amp;space;n\\&amp;space;0&amp;space;\end{Bmatrix}&amp;space;=0(n&gt;0))

同样的，它只能通过递推得到。其递推公式为：

![](http://latex.codecogs.com/svg.latex?\begin{Bmatrix}&amp;space;n\\&amp;space;k&amp;space;\end{Bmatrix}&amp;space;=&amp;space;\begin{Bmatrix}&amp;space;n-1\\&amp;space;k-1&amp;space;\end{Bmatrix}&amp;space;+k&amp;space;\begin{Bmatrix}&amp;space;n-1\\&amp;space;k&amp;space;\end{Bmatrix})

### 特殊性质

1. ![](http://latex.codecogs.com/svg.latex?\begin{Bmatrix}&amp;space;n\\&amp;space;n&amp;space;\end{Bmatrix}&amp;space;=1)

2. ![](http://latex.codecogs.com/svg.latex?\begin{Bmatrix}&amp;space;n\\&amp;space;2&amp;space;\end{Bmatrix}&amp;space;=2^{n-1}+1)

3. ![](http://latex.codecogs.com/svg.latex?\begin{Bmatrix}&amp;space;n\\&amp;space;n-1&amp;space;\end{Bmatrix}&amp;space;=&amp;space;\begin{Bmatrix}&amp;space;n\\&amp;space;n-1&amp;space;\end{Bmatrix}&amp;space;+\begin{Bmatrix}n\\n-1\end{Bmatrix}=C_n^2)

4. ![](http://latex.codecogs.com/svg.latex?x^k=\sum_{i=1}^k\begin{Bmatrix}&amp;space;k\\&amp;space;i&amp;space;\end{Bmatrix}&amp;space;i!C_x^i)

5. 设 ![](http://latex.codecogs.com/svg.latex?z=n-\left&amp;space;\lceil&amp;space;\frac{k+1}{2}&amp;space;\right&amp;space;\rceil,&amp;space;w=\left&amp;space;\lfloor&amp;space;\frac{k-1}{2}&amp;space;\right&amp;space;\rfloor)，则有 ![](http://latex.codecogs.com/svg.latex?\begin{Bmatrix}&amp;space;n\\&amp;space;k&amp;space;\end{Bmatrix}&amp;space;=C_z^w(mod&amp;space;2))

就这样吧。这个公式啊，真的是糟糕透顶
