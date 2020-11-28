---
title: 机器学习笔记(四)
author: Kzero Coder
date: 2020-11-04 18:00:00 +0800
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


# 贝叶斯分类器，条件独立和朴素贝叶斯

## 条件独立

**定义**：X与Y在给定Z的情况下条件独立，当X的概率分布在给定Z的情况下与Y值独立，数学表示为：

$$
(\forall i,j,k)P(X=x_i|Y=y_j,Z=z_k)=P(X=x_i|Z=z_k)
$$

一般写作：

$$
P(X|Y,Z)=P(X|Z)
$$

## 朴素贝叶斯

朴素贝叶斯假设在给定$Y$的情况下$X_i$相互条件独立，那么就有：

$$
P(X_1,X_2|Y)=P(X_1|X_2,Y)P(X_2|Y)=P(X_1|Y)P(X_2|Y)
$$

一般形式为：

$$
P(X_1,\dots,X_n|Y)=\prod_{i=1}^{n}P(X_i|Y)
$$

在这个条件下，贝叶斯法则可以写为：

$$
P(Y=y_k|X_1,\dots,X_n)=\frac{P(Y=y_k)P(X_1,\dots,X_n|Y=y_k)}{\sum_{i=1}^{K}P(Y=y_i)P(X_1,\dots,X_n|Y=y_i)}\\
=\frac{P(Y=y_k)\prod_{i=1}^{n}P(X_i|Y=y_k)}{\sum_{j=1}^{K}P(Y=y_j)\prod_{i=1}^{n}P(X_i|Y=y_j)}
$$

自然分类决策标准为：

$$
Y^{new}\leftarrow\mathop{\arg\max}\limits_{y_k}P(Y=y_k)\prod_{i=1}^{n}P(X_i|Y=y_k)
$$

### 离散数据的朴素贝叶斯算法

- 训练朴素贝叶斯

  <img src="/assets/img/machineLearning/machineLearningCh4/image-20201116172409488.png" alt="image-20201116172409488" style="zoom:50%;" />

- 分类

  <img src="D:/学习&素材/机器学习/机器学习复习笔记.assets/image-20201116172444865.png" alt="image-20201116172444865" style="zoom:50%;" />

  

**MLE**：

$$
\hat{\pi}_k=\hat{P}(Y=y_k)=\frac{\#D\{Y=y_k\}}{|D|}\\
\hat{\theta}_{ijk}=\hat{P}(X_i=x_{ij}|Y=y_k)=\frac{\#D\{X_i=x_{ij}\wedge Y=y_k\}}{\#D\{Y=y_k\}}\\
\hat{\theta}=\mathop{\arg\max}\limits_{\theta}P(D|\theta)
$$

使用MLE估计受数据影响大：如果有某个$P(X_i\|Y)$为0，则整个$P(Y\|X)$为0。



**MAP**

$$
\hat{\pi}_k=\hat{P}(Y=y_k)=\frac{\#D\{Y=y_k\}+\alpha_k}{|D|+\sum_{m}\alpha_m}\\
\hat{\theta}_{ijk}=\hat{P}(X_i=x_{ij}|Y=y_k)=\frac{\#D\{X_i=x_{ij}\wedge Y=y_k\}+\alpha_k'}{\#D\{Y=y_k\}+\sum_m\alpha_m'}\\
\hat{\theta}=\mathop{\arg\max}\limits_{\theta}P(\theta|D)=\mathop{\arg\max}\limits_{\theta}\frac{P(D|\theta)P(\theta)}{P(D)}
$$

和MLE相比，只是在分子分母上下同加了一个假想值，可以减少数据的影响。



朴素贝叶斯使用了很强的条件独立性假设，但通常情况下$X_i$并不是条件独立的，但是其效果依然很好。

<img src="/assets/img/machineLearning/machineLearningCh4/image-20201116202711235.png" alt="image-20201116202711235" style="zoom:80%;" />

(借用学长的特殊例子过程)

### 连续数据的朴素贝叶斯算法

以Gaussian Naive Bayes(GNB)为例子：

$$
P(X_i=x|Y=y_k)=\frac{1}{\sigma_{ik}\sqrt{2\pi}}\exp(-\frac{(x-\mu_{ik})^2}{2\sigma_{ik}^2})
$$

(有时候对方差会有更强的假设，如：

- 方差与类别Y独立
- 方差与维度X独立
- 上面两个都有

）

- 训练贝叶斯模型：

  <img src="/assets/img/machineLearning/machineLearningCh4/image-20201116204324115.png" alt="image-20201116204324115" style="zoom: 50%;" />

  利用MLE进行估计

  $$
  \hat{\mu}_{ik}=\frac{1}{\sum_j\delta(Y^j=y_k)}\sum_jX_i^j\delta(Y^j=y_k)\\
  \hat{\delta}_{ik}^2=\frac{1}{\sum_j\delta(Y^j=y_k)}\sum_j(X_i^j-\hat{\mu}_{jk})^2\delta(Y^j=y_k)\\
  \delta(z)=1\ if\ z\ true,\ else\ 0
  $$

- 分类

  <img src="/assets/img/machineLearning/machineLearningCh4/image-20201116204403332.png" alt="image-20201116204403332" style="zoom:50%;" />

# 逻辑回归

Naive Bayes是通过$P(Y),P(X\|Y)$计算$P(Y\|X)$，逻辑回归则是直接学习$P(Y\|X)$。

- 考虑学习函数$f:X\rightarrow Y$，当

  - X是一个实向量$<X_1\dots X_n>$
  - Y是布尔变量
  - 假设当给定$Y$的时候所有的$X_i$相互条件独立
  - 模型$P(X_i\|Y=y_k)$是高斯模型$N(\mu_{ik},\sigma_i)$
  - 模型$P(Y)$是伯努利模型$Bernoulli(\pi)$

- 接下来计算$P(Y=1\|X)$：

  $$
  P(Y=1|X)=\frac{P(Y=1)P(X|Y=1)}{P(Y=1)P(X| Y=1)+P(Y=0)P(X|Y=0)}\\
  =(1+\frac{P(Y=0)P(X| Y=0)}{P(Y=1)P(X|Y=1)})^{-1}=(1+exp(ln\frac{P(Y=0)P(X| Y=0)}{P(Y=1)P(X|Y=1)}))^{-1}
  $$


  因为$P(Y)\sim B(\pi)$，所以$\pi=P(Y=1)$：

$$
P(Y=1|X)=(1+exp(ln\frac{\pi}{1-\pi}+\sum_{i}{ln{P(X_i|Y=0)}{P(X_i|Y=1)}}))^{-1}
$$

  又因为$P(X_i\|Y=y_i)\sim N(\mu_{ik},\sigma_i)$：

$$
P(X_i|Y=y_i)=\frac{1}{\sigma\sqrt{2\pi}}\exp(-\frac{(x-\mu_{ik})^2}{2\sigma_i^2})
$$

  所以有：
$$
P(Y=1|X)=(1+exp(ln\frac{1-\pi}{\pi}+\sum_{i}\frac{\mu_{i0}-\mu_{i1}}{\sigma_i^2}X_i+\frac{\mu_{i1}^2-\mu_{i0}^2}{2\sigma_i^2}))^{-1}
$$

  化为向量形式：
$$
  P(Y=1|X)=(1+exp(W^TX))^{-1}
$$



*注：对于多类逻辑回归，只需要学习R-1个参数，第R个直接减。*



## 训练逻辑回归：MCLE

假设我们有$L$个训练样本$\{<X^1,Y^1>,\dots,<X^L,Y^L>\}$

对$W$做最大似然估计：

$$
W_{MCLE}=\mathop{\arg\max}\limits_{W}\prod P(Y^l|W,X^l)
$$

损失函数为：

$$
l(W)=\sum_llnP(Y^l|W,X^l)=\sum_lY^llnP(Y^l=1|X^l,W)+(1-Y^l)lnP(Y^l=0|X^l,W)\\
=\sum_lY^lln\frac{P(Y^l=1|X^l,W)}{P(Y^l=0|X^l,W)}+lnP(Y^l=0|X^l,W)\\
=\sum_lY^l(W^TX)-ln(1+exp(W^TX))
$$

其中$l(W)$是凹函数，但是解析解，一般用梯度下降、牛顿法、共轭梯度等进行迭代下降逼近。

### 梯度下降法

MLE：

$$
\frac{\partial l(W)}{\partial w_i}=\sum_lX_i^l(Y^l-\hat{P}(Y^l=1|X^l,W))\\
w_{i+1}\leftarrow w_i+\eta\sum_lX_i^l(Y^l-\hat{P}(Y^l=1|X^l,W))
$$

若假设W的先验为高斯分布$N(0,\sigma)$，以MAP估计：

$$
w_{i+1}\leftarrow w_i-\eta\lambda w_i+\eta\sum_lX_i^l(Y^l-\hat{P}(Y^l=1|X^l,W))
$$

其中$\eta\lambda w_i$为正则项。

### 牛顿法

$$
\frac{\partial^2l(W)}{\partial W}=-\sum_lx_i^lx_i^lP(Y=0|X^l,W)P(Y=1|X^l,W)\\
w_{i+1}\leftarrow w_i-\frac{\partial l(W)}{\partial w_i}(\frac{\partial^2l(W)}{\partial W})^{-1}
$$

## 针对离散的X

假设$X_i$为布尔变量时，假设$P(X_i\|Y=y_k)\sim B(\theta_{ik})$