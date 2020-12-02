---
title: 机器学习笔记(七)
author: Kzero Coder
date: 2020-11-20 18:00:00 +0800
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


# PCA

考虑一个数据集$\{x_1,\dots,x_N\},x_n\in R^D$，目标是投影数据到$M(M<D)$维空间内。

## 最大方差推导

假设降维到一维，即$M=1$。定义$M$维空间的方向由$u_1$决定，且$u_1^Tu_1=1$

**目标**：最大化投影方差。

$$
\frac{1}{N}\sum_{i=1}^{N}(\mu_1^Tx_i-\mu_1^T\overline{x})^T(\mu_1^Tx_i-\mu_1^T\overline{x})=\frac{1}{N}\sum_{i=1}^{N}(\mu_1^Tx_i-\mu_1^T\overline{x})(\mu_1^Tx_i-\mu_1^T\overline{x})^T\\
=\frac{1}{N}\sum_{i=1}^{N}\mu_1^T(x_i-\overline{x})^T(x_i-\overline{x})\mu_1=\mu_1^TS\mu_1\\
where.\ \overline{x}=\frac{1}{N}\sum_{i=1}^Nx_i,\ S=\frac{1}{N}\sum_{i=1}^{N}(x-\overline{x})(x-\overline{x})^T
$$

很显然$S$是原数据的方差。

为了求解$\mu_1$使得方差最大，使用拉格朗日乘子法：

$$
\frac{\partial(\mu_1^TS\mu_1+\lambda(1-\mu_1^T\mu_1))}{\partial\mu_1}=2S\mu_1-2\lambda\mu_1
$$

使其为0，可以得到：

$$
S\mu_1=\lambda\mu_1
$$

其中$\lambda$为常数，所以显然$\mu_1$为S的特征向量。

要最大化$\mu_1^TS\mu_1$，即：

$$
\mu_1^TS\mu_1=\mu_1^T\lambda\mu_1=\lambda
$$

所以$\mu_1$对应特征值最大的特征向量。



## 最小误差推导

引入一组$D$维单位正交基集合：$\{\mu_1,\dots,\mu_D\},\mu_i\mu_j=\delta_{ij}(\delta_{ij}=1,if\ i=j;else\ 0)$

可以把原数据点改写为$x_n=\sum_{i=1}^{D}\alpha_{ni}\mu_i,\alpha_{ni}=x_n^T\mu_i$。相当于旋转旧坐标系到由$\mu_i$决定的新坐标系。

目标是通过投影到$M$维子空间类表示这些点。

通过前$M$个基向量来表示$M$维线性子空间：

$$
\tilde{x}_n=\sum_{i=1}^{M}z_{ni}\mu_i+\sum_{i=M+1}^{D}b_i\mu_i
$$

其中$z_{ni}$依赖于特定样本点，$b_i$为常数。

**目标**：通过$\mu_i,z_{ni},b_i$最小化失真

$$
J=\frac{1}{N}\sum_{n=1}^{N}||x_n-\tilde{x}_n||^2=\frac{1}{N}\sum_{n=1}^{N}(x_n-\tilde{x}_n)^T(x_n-\tilde{x}_n)\\
=\frac{1}{N}\sum_{n=1}^{N}(x_n-(\sum_{i=1}^{M}z_{ni}\mu_i+\sum_{i=M+1}^{D}b_i\mu_i))^T(x_n-(\sum_{i=1}^{M}z_{ni}\mu_i+\sum_{i=M+1}^{D}b_i\mu_i))\\
=\frac{1}{N}\sum_{n=1}^{N}(x_n^Tx_n-2x_n^T(\sum_{i=1}^{M}z_{ni}\mu_i+\sum_{i=M+1}^{D}b_i\mu_i)+(\sum_{i=1}^{M}z_{ni}\mu_i+\sum_{i=M+1}^{D}b_i\mu_i)^T(\sum_{i=1}^{M}z_{ni}\mu_i+\sum_{i=M+1}^{D}b_i\mu_i))\\
\ \\
\ \\

\frac{\partial{J}}{\partial{z_{ni}}}\propto2x_n^T\mu_i-2z_{ni}\mu_i^T\mu_i(i=1,\dots,M),\ z_{ni}=x_n^T\mu_i\\
\frac{\partial{J}}{\partial{b_{i}}}\propto\frac{1}{N}\sum_{n=1}^{N}2x_n^T\mu_i-2b_i\mu_i^T\mu_i(i=M+1,\dots,D),\ b_i=\frac{1}{N}\sum_{n=1}^{N}x_n^T\mu_i=\overline{x}_n^T\mu_i
$$

因此

$$
J=\frac{1}{N}\sum_{n=1}^{N}(x_n-\tilde{x}_n)^T(x_n-\tilde{x}_n)=\\
\frac{1}{N}\sum_{n=1}^N(x_n-\sum_{i=1}^Mx_n^T\mu_i\mu_i-\sum_{i=M+1}^{D}\overline{x}_n^T\mu_i\mu_i)^T(x_n-\sum_{i=1}^Mx_n^T\mu_i\mu_i-\sum_{i=M+1}^{D}\overline{x}_n^T\mu_i\mu_i)\\
=\frac{1}{N}\sum_{n=1}^N((\sum_{i=M+1}^{D}(x_n-\overline{x}_n)^T\mu_i\mu_i)^T(\sum_{i=M+1}^{D}(x_n-\overline{x}_n)^T\mu_i\mu_i))\\
=\frac{1}{N}\sum_{n=1}^N\sum_{i=M+1}^{D}(x_n^T\mu_i-\overline{x}_n^T\mu_i)^2=\sum_{i=M+1}^{D}\mu_i^TS\mu_i
$$

与最大方差推导同结果。



## 对高维数据的PCA

寻找$D$维数据方差的特征向量，规模为$O(D^3)$，在维度较高的情况下，计算时间较长。

**解决方法**：令$X$为$N\times D$中心化数据矩阵，相关特征向量为

$$
\frac{1}{N}X^TX\mu_i=\lambda_i\mu_i
$$

两边乘$X$得：

$$
\frac{1}{N}XX^T(X\mu_i)=\lambda_i(X\mu_i)
$$

令$X\mu_i=v_i$：

$$
\frac{1}{N}XX^Tv_i=\lambda_iv_i
$$

此时$XX^T$为$N\times N$矩阵，其有$N-1$个特征值和原始元素协方差矩阵相同。

为了确定特征向量，两边同乘$X^T$：

$$
\frac{1}{N}X^TX(X^Tv_i)=\lambda_i(X^Tv_i)
$$

得到的$X^Tv_i$即是$S$特征值为$\lambda_i$的特征向量（不一定归一化）。

