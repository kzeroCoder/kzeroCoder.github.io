---
title: 机器学习笔记(二)
author: Kzero Coder
date: 2020-10-16 18:00:00 +0800
categorise: [Blogging, Machine Learning]
tage: [writing]
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


# 概率论

**噪声**：因为传感器测量的变动性，形成的部分可见性和不正确标签。

## 一些简单定理和定义

<left><img src="/assets/img/machineLearning/machineLearningCh2/image-20201115110216175.png" alt="image-20201115110216175" style="zoom:50%;" /></left>

随机变量的值在实验重复的情况下会发生变化。

概率分布分为离散概率分布(如Bernoulli分布)和连续概率分布(如Gaussian分布，由概率密度函数下的面积确定)

**独立**：$P(A\cap B)=P(A)*P(B)$

**在连续概率分布下的p(x)的直观意义**：

$p(x_1)=a，p(x_2)=b$表示观测X”接近“$x_1$的可能是观测X"接近”$x_2$可能的a/b倍

$$
\lim_{h\rightarrow 0}\frac{P(x_1-h<X<x_1+h)}{P(x_2-h<X<x_2+h)}=\lim_{h\rightarrow0}\frac{\int_{x_1-h}^{x_1+h}p(x)\mathrm{d}x}{\int_{x_2-h}^{x_2+h}p(x)\mathrm{d}x}\approx \frac{a}{b}
$$

## 连续概率分布

**均匀概率分布**：

$$
p(x)=1/(b-a), a\le x\le b
$$

**一维Gaussian分布**：

$$
p(x)=\frac{1}{\sqrt{2\pi}\sigma}e^{-(x-\mu)^2/2\sigma^2}
$$

**多维Gaussian分布**：

$$
p(x\|\mu,\Sigma)=\frac{1}{(2\pi)^{D/2}}\frac{1}{\|\Sigma\|^{1/2}}exp\left\{-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)\right\}
$$

**指数分布**：

$$
p(x)=\frac{1}{\mu}e^{-x/\mu}
$$

## 数值特征

<left><img src="/assets/img/machineLearning/machineLearningCh2/image-20201115112129378.png" alt="image-20201115112129378" style="zoom:50%;" /></left>

## 中心极限定理

设$(X_1,X_2,\dots,X_n)$独立同分布，当n很大的时候，$p(\overline{x})$近似等于$N(E[X_i],Var[X_i]/N)$

## 条件独立

A和B在给定C的情况下条件独立，如果：

$$
P(A\cap B|C)=P(A|C)*P(B|C)\equiv p(A|B,C)=P(A|C)
$$

即在C确定下，B对A无影响。

## 先验分布和后验分布

**先验分布**：在试验观察之前，得到的经验估计，称为先验分布，如$P(\theta)=a$

**后验分布**：后验分布是一个条件分布，即对样本x的条件下，$\theta$的条件分布$P(\theta\|x)$，一般通过Bayes法则进行计算。

**贝叶斯分布**：

$$
P(A|B)=\frac{P(B|A)P(A)}{P(B)}=\frac{P(B|A)P(A)}{P(B|A)P(A)+P(B|\neg A)P(\neg A)}
$$

## 联合分布和边缘分布

**联合分布**：即A, B, C等同时出现的概率$P(A,B,C,\dots)$

**边缘分布**：通过联合概率分布，减少随机变量，如$P(A,B)=P(A,B,C)+P(A,B,\neg C)$，在连续概率分布中表现为对被减少维度的积分。

## 最大似然估计(MLE)

在独立同分布的完全观测的假设下：

$$
L(\theta)=P(D;\theta)=P(x_1, x_2,\dots,x_N;\theta)\\
=P(X_1;\theta)\dots P(X_N;\theta)=\prod_i^NP(X_i;\theta)
$$

由于计算机对于较小数的多次乘法有较大舍入误差，一般使用对数似然估计：

$$
\log L(\theta)=\sum_i^N \log P(X_i;\theta)
$$

对参数$\theta$估计得到：

$$
\hat\theta=\mathop{\arg\min}\limits_{\theta}L(\theta)/\mathop{\arg\min}\limits_{\theta}\log L(\theta)
$$

## 对过拟合的缓和(smoothing)

对于Bernoulli分布，有$\hat\theta_{ML}^{head}=\frac{n^{head}}{n^{head}+n_{tail}}$。如果$n^{head}$为0，那么显然参数为0。

加入$n'$作为假想计数，得到$\hat\theta_{ML}^{head}=\frac{n^{head}+n'}{n^{head}+n_{tail}+n'}$可以缓和过拟合。

## 贝叶斯学习

![image-20201115120411358](/assets/img/machineLearning/machineLearningCh2/image-20201115120411358.png)

那么最大后验估计MAP estimate为$\hat\theta_{MAP}=\mathop{\arg\min}\limits_{\theta}P(\theta\|D)$。

显然，如果先验是均匀分布，那么MLE=MAP

## Beta分布(Bernoulli的贝叶斯估计)

$$
Beta(\mu|a,b)=\frac{\Gamma(a+b)}{\Gamma(a)\Gamma(b)}\mu^{a-1}(1-\mu)^{b-1}\\
where\space \Gamma(x)\equiv\int_0^\infty u^{x-1}e^{-u}du,\  \int_0^1Beta(\mu|a,b)d\mu=1\\
E|\mu|=\frac{a}{a+b}, var|\mu|=\frac{ab}{(a+b)^2(a+b+1)}
$$

其中参数a, b控制参数$\mu$的分布，被称为hyperparameter(超参)

*注：a, b不一定为整数*

将beta分布代入先验概率$P(\mu)$，把二项分布的似然函数代入$P(data\|\mu)$，得到：

$$
P(\mu|data)=P(\mu|m,l,a,b)\propto P(\mu)P(data|\mu)\propto \mu^m(1-\mu)^l\mu^{a-1}(1-\mu)^{b-a}\propto \mu^{m+a-1}\mu^{l+b-a}\\
(l=N-m)
$$

可以看到$P(\mu\|data)$满足beta分布的形式，所以beta分布是二项分布似然函数的共轭先验(也是Probability Density Function)；

$$
P(\mu|m,l,a,b)=\frac{\Gamma(m+a+l+b)}{\Gamma(m+a)\Gamma(l+b)}\mu^{m+a-1}(1-\mu)^{l+b-1}\\
E[\mu|data]=\frac{m+a}{m+a+l+b}
$$

如果要尽可能好地预测下一次的输出，在给定观测数据集D的情况下，x的预测分布为：

$$
p(x=1|D)=\int_0^1p(x=1|\mu)p(\mu|D)d\mu=\int_0^1\mu p(\mu|D)d\mu=E[\mu|D]=\frac{m+a}{m+a+l+b}
$$

随着观测数量的增加，后验概率表示的不一定性将会持续降低。通过贝叶斯推断问题，参数为$\theta$，观测数据集D，由联合概率分布$p(\theta,D)$描述：

$$
E_\theta[\theta]=E_D[E_\theta[\theta|D]]\\
where\space E_\theta[\theta]=\int p(\theta)\theta d\theta, E_D[E_\theta[\theta|D]]=\int\left\{\int\theta_p(\theta|D)d\theta\right\}p(D)dD\\
Proof\space var_\theta[\theta]=E_D[var_\theta[\theta|D]]+var_D[E_\theta[\theta|D]]
$$

var公式左侧项是$\theta$的先验方差，右侧第一项为$\theta$的平均后验方差，第二项是$\theta$的后验均值的方差，平均来看，$\theta$的后验方差小于先验方差。

**MAP**

$\theta$的后验分布：

$$
P(\theta|x_1,\dots,x_N)=\frac{p(x_1,\dots,x_N|\theta)p(\theta)}{p(x_1,\dots,x_N)}\propto\theta^{m}(1-\theta)^{l}\times \theta^{\alpha-1}(1-\theta)^{\beta-1}=\theta^{m+\alpha-1}(1-\theta)^{l+\beta-1}
$$

MAP过程：

$$
\hat\theta_{MAP}=\mathop{\arg\min}\limits_{\theta}logP(\theta|x_1,\dots,x_n)=\frac{m+a}{N+\alpha+\beta}
$$

## Dirichlet分布

Dirichlet分布基本可以说是多维beta分布：

$$
Dir(\mu|\alpha)=\frac{\Gamma(\alpha_0)}{\Gamma(\alpha_1)\dots\Gamma(\alpha_K)}\prod_{k=1}^K \mu^{\alpha_k-1}\\
\alpha_0=\sum_{k=1}^K \alpha_k
$$

## MLE vs MAP

**Frequentist/MLE approach**：$\theta$是一个未知常量，从数据中估计

**Bayesian/MAP approach**：$\theta$是一个随机变量，表示一个概率分布

**缺点**：

- MLE：当数据集太小的时候，会过拟合
- MAP：利用不同先验的人，将终止于不同的估计