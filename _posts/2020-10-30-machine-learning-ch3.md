---
title: 机器学习笔记(三)
author: Kzero Coder
date: 2020-10-30 18:00:00 +0800
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


# 分类理论和非参分类器

## 贝叶斯决策论

贝叶斯决策论对分类任务来说，在相关概率都已知的理想情况下，贝叶斯决策考虑如何基于这些概率和误判损失来选择最优的类别标识。

<img src="/assets/img/machineLearning/machineLearningCh3/image-20201115213247059.png" alt="image-20201115213247059" style="zoom:50%;" />

当二分类的类别为正态分布时，可以得到一个decision boundary (决策面)。

假设$\Sigma_1=\Sigma_2=\Sigma, P(Y=1)=\theta$：

$$
P(Y=1|X)=\frac{p(X|Y=1)P(Y=1)}{p(X)}=\frac{\theta N(\mu_1,\Sigma_1)}{p(X)}\\
P(Y=2|X)=\frac{p(X|Y=2)P(Y=2)}{p(X)}=\frac{(1-\theta) N(\mu_2,\Sigma_2)}{p(X)}\\
\textbf{令P(Y=1|X)/P(Y=2|X)=1有决策面}\\
1=\frac{\theta}{1-\theta}\sqrt{\frac{\Sigma_2}{\Sigma_1}}exp\left\{\frac{1}{2}[(x-\mu_2)^T\Sigma_2^{-1}(x-\mu_2)-(x-\mu_1)^T\Sigma_1^{-1}(x-\mu_1)]\right\}
$$

两边取对数得到：

$$
0=ln\frac{\theta}{1-\theta}+ln\sqrt{\frac{\Sigma_2}{\Sigma_1}}+\frac{1}{2}[(x-\mu_2)^T\Sigma_2^{-1}(x-\mu_2)-(x-\mu_1)^T\Sigma_1^{-1}(x-\mu_1)]\\
=ln\frac{\theta}{1-\theta}+ln\sqrt{\frac{\Sigma_2}{\Sigma_1}}+\frac{1}{2}[\mu_1^T\Sigma^{-1}\mu_1+\mu_2^T\Sigma^{-1}\mu_2-2x^T\Sigma^{-1}\mu_1-2x^T\Sigma^{-1}\mu_2]
$$


## Bayes Error

样本后验概率：

$$
P(Y=i|X)=\frac{p(X|Y=i)P(Y=i)}{p(X)}=\frac{\pi_ip_i(X)}{\sum_i\pi_ip_i(X)}=q_i(X)
$$

计算一个样本被分到错误类别的概率称为误差概率，在给定数据集$X$的情况下，风险函数：

$$
r(x)=min[q_1(X),q_2(X)]
$$

那么错误的期望为：

$$
e=E(r(X))=\int r(x)p(x)\mathrm{d}x=\int min[\pi_1p_1(x),\pi_2p_2(x)]\mathrm{d}x\\
=\pi_1\int_{L_1}p_1(x)\mathrm{d}x+\pi_2\int_{L_2}p_2(x)\mathrm{d}x=\pi_1e_1+\pi_2e_2\\
sometimes,\ e_1=\int_{ln(\pi_1/\pi_2)}^{+\infty}p_1(x)\mathrm{d}x,\ e_1=\int^{ln(\pi_1/\pi_2)}_{-\infty}p_2(x)\mathrm{d}x
$$

Bayes error时分类错误概率的下界。

![image-20201115220452278](/assets/img/machineLearning/machineLearningCh3/image-20201115220452278.png)

贝叶斯分类器是理论上最好分类器，其最小化了分类错误概率。

但是因为需要积分密度函数，所以计算Bayes error一般是一个非常困难的问题。

## 学习分类器

<left><img src="/assets/img/machineLearning/machineLearningCh3/image-20201115221738397.png" alt="image-20201115221738397" style="zoom:50%;" /></left>

分类问题可分为两个阶段：

- 推断(inference)：使用模型习得$P(C_k\|X)$。
- 决策(decision)：利用后验进行分类取最优。

有三种方法：

- 生成式模型(generative model)：对每个类别$C_k$，独立地确定类条件密度$p(x\|c_k)$和推断先验概率$P(C_k)$，使用贝叶斯公式求出$P(C_k\|x)$。
- 判别式模型(discriminative model)：首先解决后验类密度$p(C_k\|x)$，使用决策论来对新的x进行分类，直接对后验概率建模。
- 找到一个判别函数$f(x)$，直接将输入x映射为标签。在这种情况下，概率不起作用。



## kNN分类器

根据与样本距离最近的k个样本的类别"投票"得到该样本的标签。

其特征为：

- 不具有显式的学习过程，直接预测(惰性学习的代表)
- 非参数学习算法
- 具有很高的容量，在训练样本较大时有较高精度
- 计算成本高，训练集较小的时候泛化能力差，易过拟合

非参数化方法估计的**基本形式**$p(x)=\frac{k}{NV}$，k为落在区域R的数据点数量，V为区域R的体积，N为样本总数。

- Parzen density estimate：固定V，从数据中确定k。区域R取以x为中心的小超立方体。如果数据点$x_n$位于以x为中心的边长为h的超立方体内，则$k(\frac{x-x_k}{h})=1$，则点x处的概率密度估计为

  $$
  p(x)=\frac{1}{NV}\sum_{n=1}^Nk(\frac{x-x_k}{h})
  $$

- kNN方法：固定k，从数据中确定V。考虑以x为中心的小球体，估计概率密度$p(x)$。允许球体的体积自由增加到恰好包含k个不同的数据点，则$p(x)=\frac{k}{NV}$，V为球体的最终体积。

**基于kNN的贝叶斯分类器**：

<img src="/assets/img/machineLearning/machineLearningCh3/image-20201115223604428.png" alt="image-20201115223604428" style="zoom:67%;" />

其中$k_1+k_2=k,V_1=V_2$

### kNN的三要素

1. k值的选择
   - k较小，学习偏差小，学习方差偏大。易发生过拟合。
   - k较大，模型变简单，学习波动较小，增大学习偏差。
2. 距离度量
   - Lp距离：$Lp(x,x')=(\sum_{l=1}^{n}\|x_{l}-x'_{l}\|^p)^{1/p}$
     - p = 2时为欧氏距离
     - p = 1时为曼哈顿距离
     - p为无穷的时候为差值最大值
   - 马氏距离
     - $D(x,x')=\sqrt{(x-x')^T\Sigma^{-1}(x-x')}$
   - 内积，角度，相关系数，海明距离
3. 决策规则

## 总结

<img src="/assets/img/machineLearning/machineLearningCh3/image-20201115224809219.png" alt="image-20201115224809219" style="zoom:67%;" />