---
title: 机器学习笔记(五)
author: Kzero Coder
date: 2020-11-10 18:00:00 +0800
categories: [Blogging, Machine Learning]
tags: [writing]
math: true
---
<head>
    <script type="text/x-mathjax-config"> 
   		MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } }); 
   	</script>
    <script type="text/x-mathjax-config">
    	MathJax.Hub.Config({tex2jax: {
             inlineMath: [ ['$','$'], ["\\(","\\)"] ],
             processEscapes: true
           }
         });
    </script>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" 	type="text/javascript">
	</script>
</head>


# 支持向量机SVM

SVM是定义在特征空间上的，间隔最大的线性分类器，是一种二分类模型。

SVM支持处理线性可分数据集和非线性可分数据集：

- 训练数据集线性可分时，通过硬间隔最大化，学习一种线性分类器
- 训练数据集近似线性可分时，通过软间隔最大化，学习一种线性分类器
- 训练数据集不可分时，通过使用核技巧和软间隔最大化，学习一种非线性分类器

## 硬间隔最大化

<img src="/assets/img/machineLearning/machineLearningCh5/image-20201117103137099.png" alt="image-20201117103137099" style="zoom:50%;" />

Margin(记作m)为$d^-+d^+$

$$
w^Tx_i+b>+c,\ for\ all\ x_i\in class2\\
w^Tx_i+b<-c,\ for\ all\ x_i\in class1\\

or\ (w^Tx_i+b)y_i>c
$$

很明显，最小可行Margin为：

$$
m=\frac{w^T}{||w||}(x_i-x_j)=\frac{2c}{||w||}
$$

目标是最大化Margin，问题变化为：

$$
max_w\ \frac{2c}{||w||}\\
s.t\ y_i(w^Tx_i+b)\ge c,\ \forall i
$$

此时问题的解$w^\*,b^\*,c^\*$太过松散。但是如果我们成比例将$w$和$b$变为$\lambda w$和$\lambda b$，此时间隔变为$2\lambda c$，但是对求解最优化问题的不等式约束没有影响，所以可以把问题简化为：

$$
max_w\ \frac{1}{||w||}\\
s.t\ y_i(w^Tx_i+b)\ge 1,\ \forall i
$$

### 对偶问题

假设原问题为：

$$
min_w\ f(w)\\
s.t\ (g_i(w)\le 0,i=1,\dots,k)\wedge(h_i(w)=0,i=1,\dots,l)
$$

则用拉格朗日乘子法可以表示为：

$$
L(w,\alpha,\beta)=f(w)+\sum_{i=1}^{k}\alpha_ig_i(w)+\sum_{i=1}^{l}\beta_ih_i(w)
$$

重写原问题为：

$$
min_wmax_{\alpha,\beta,\alpha_i\ge0}L(w,\alpha,\beta)
$$

其对偶问题为
：
$$
max_{\alpha,\beta,\alpha_i\ge0}min_wL(w,\alpha,\beta)
$$

设原问题最优解为$p^*$，对偶问题最优解为$d^*$，那么可以得到：

- 弱对偶性：

  $$
  d^*=max_{\alpha,\beta,\alpha_i\ge0}min_wL(w,\alpha,\beta)\le min_wmax_{\alpha,\beta,\alpha_i\ge0}L(w,\alpha,\beta)=p^*
  $$
  
  证明过程：
  
  $$
  obviously,\ min_wL\le L\le max_{\alpha,\beta,\alpha_i\ge0}L,\ then:\\
  min_wL\le max_{\alpha,\beta,\alpha_i\ge0}L\Rightarrow max_{\alpha,\beta,\alpha_i\ge0}min_wL\le min_wmax_{\alpha,\beta,\alpha_i\ge0}L
  $$
  
- 强对偶性：

  $$
  L(w,\alpha,\beta)\ has\ a\ saddle\ point\Longleftrightarrow d^*=p^*
  $$
  

### KKT条件(Karush-Kuhn-Tucker)

如果L中有鞍点，那么鞍点满足KTT条件：

$$
\frac{\partial}{\partial w_i}L(w,\alpha,\beta)=0,i=1,\dots,k\\
\frac{\partial}{\partial \beta_i}L(w,\alpha,\beta)=0,i=1,\dots,l\\
\alpha_ig_i(w)=0,i=1,\dots,m\\
g_i(w)\le 0,i=1,\dots,m\\
\alpha_i\ge0,i=1,\dots,m
$$

**定理**：如果$w^*,\alpha^*,\beta^*$满足KTT条件，那么由对偶问题就能获得著问题的最优下界。

### 求解margin分类器最优问题

原问题等价于：

$$
min_{w,b}\ \frac{1}{2}w^Tw\\
s.t\ 1-y_i(w^Tx_i+b)\le0,\ \forall i
$$

写成拉格朗日乘子法：

$$
L(w,b,\alpha)=\frac{1}{2}w^Tw+\sum_{i=1}^{m}\alpha_i[1-y_i(w^Tx_i+b)]
$$

接下来求解对偶问题：

$$
max_{\alpha_i\ge0}min_{w,b}L(w,b,\alpha)
$$

先最小化w和b有：

$$
\nabla_wL(w,b,\alpha)=w-\sum_{i=1}^{m}\alpha_iy_ix_i=0\\
\nabla_{b}L(w,b,\alpha)=\sum_{i=1}^{m}\alpha_iy_i=0\\
\Longrightarrow w=\sum_{i=1}^{m}\alpha_iy_ix_i
$$

将L中的w替换：

$$
L(w,b,\alpha)=\sum_{i=1}^{m}\alpha_i-\frac{1}{2}\sum_{i,j=1}^{m}\alpha_i\alpha_jy_iy_j(x_i^Tx_j)
$$

因此对偶问题变为：

$$
max_\alpha\ J(\alpha)=\sum_{i=1}^{m}\alpha_i-\frac{1}{2}\sum_{i,j=1}^{m}\alpha_i\alpha_jy_iy_j(x_i^Tx_j)\\
s.t\ (\alpha_i\ge0,i=1,\dots,k)\wedge(\sum_{i=1}^{m}\alpha_iy_i=0)
$$

这是一个二次方程规划问题(quadratic programming)。

注意两件事：

- $w=\sum_{i=1}^{m}\alpha_iy_ix_i$
- 核kernel：$x_i^Tx_j$



训练样本中对应$\alpha_i>0$的数据样本称为支持向量(support vectors)

要得到b，只需要将某个SV代入即可：$b=y_j-\sum_{i=1}^{m}\alpha_iy_i(x_i^Tx_j)$

$w=\sum_{i\in SV}\alpha_iy_ix_i$



对于新数据集$z$。计算$w^Tz+b=\sum_{i\in SV}\alpha_iy_i(x_i^Tz)+b$。如果为正，分到class1；否则class2。$w$不需要显式还原。

$$
y^*=sign(\sum_{i\in SV}\alpha_iy_i(x_i^Tz)+b)
$$

## 软间隔最大化

最优化问题有一点不一样：

$$
min_{w,b}\ \frac{1}{2}w^Tw+C\sum_{i=1}^m\xi_i\\
s.t\  (y_i(w^Tx_i+b)\ge1-\xi_i,\ \forall i)\wedge(\xi_i\ge0,\ \forall i)
$$

其中：

- $\xi_i$是松弛变量(slack variables)
- 如果$x_i$没有错误，则$\xi_i=0$
- $\xi_i$是错误的上界
- C是错误核间隔之间的权衡系数(tradeoff parameter)



对偶问题为：

$$
max_\alpha\ J(\alpha)=\sum_{i=1}^{m}\alpha_i-\frac{1}{2}\sum_{i,j=1}^{m}\alpha_i\alpha_jy_iy_j(x_i^Tx_j)\\
s.t\ (0\le\alpha_i\le C,i=1,\dots,k)\wedge(\sum_{i=1}^{m}\alpha_iy_i=0)
$$

## 非线性分类器

**核心思想：**将$x_i$转为更高空间使其线性可分。

<left><img src="/assets/img/machineLearning/machineLearningCh5/image-20201117121845236.png" alt="image-20201117121845236" style="zoom: 67%;" /></left>

### 核技巧(The kernel Trick)

由于软间隔最大化SVM中，样本点只在$J(\alpha)$最后以内积形式出现，所以仅需要在特征空间上计算内积，就不需要显性升维。

定义核函数(kernel function)K为$K(x_i,x_j)=\phi(x_i)^T\phi(x_j)$

**核函数例子：**

- 线性核(Linear kernel)：
- 
  $$
  K(x,x')=x^Tx'
  $$

- 多项式核(Polynomial kernel)/P阶多项式核：
- 
  $$
  K(x,x')=(1+x^Tx')^P
  $$

- 径向基核(Radis basis kernel)：

$$
K(x,x')=exp(-\frac{1}{2}||x-x'||^2)
$$

选择核函数时，相同效率选择特征空间较低的。

### 核的重要性

#### 待翻译

<img src="/assets/img/machineLearning/machineLearningCh5/image-20201117134213776.png" alt="image-20201117134213776" style="zoom:50%;" />