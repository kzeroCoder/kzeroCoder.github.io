---
title: 机器学习笔记(六)
author: Kzero Coder
date: 2020-11-15 18:00:00 +0800
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


# 聚类

## 无监督学习

目标：通过对无标记训练样本的学习来揭露数据的内在性质以及规律。

- 数据的分类与聚类问题
- 密度估计问题
- 降维
  - 线性：主成分分析(PCA)
  - 非线性：流形学习(Manifold learning)

## 聚类

**定义**：将一组对象分成相似对象类的过程。

- 类内高相似度
- 类间低相似度

**距离度量的属性**：

- $D(A,B)=D(B,A)$
- $D(A,A)=0$
- $D(A,B)=0\ iif\ A=B$
- $D(A,B)\le D(A,C)+D(B,C)$

**常用的距离度量**：

- Minkowski Metric：

  $d(x,y)=(\sum_{i=1}^{p}\|x_i-y_i\|^r)^{1/r}$

- Hamming distance：

  当曼哈顿距离衡量的特征全是二进制(二元)时变为汉明距离

- Correlation Coefficient

  ​	Pearson correlation coefficient：
  
  $$
  s(x,y)=\frac{\sum_{i=1}^{p}(x_i-\overline{x})(y_i-\overline{y})}{\sqrt{\sum_{i=1}^{p}(x_i-\overline{x})^2\times {\sum_{i=1}^{p}(y_i-\overline{y})^2}}}\\
  where\ \overline{x}=\frac{1}{p}\sum_{i=1}^{p}x_i\ and\ \overline{y}=\frac{1}{p}\sum_{i=1}^{p}y_i
  $$
  
- Edit distance

**聚类算法**：

- 划分算法
  - 通常开始于随机划分，迭代优化。
  - 如K means聚类，基于混合模型的聚类。
- 分层算法
  - 自底向上聚合
  - 自顶向下分割

## 层次聚类

**目的**：从一组文件中建立起一个树形层次分类(树状图)

<img src="/assets/img/machineLearning/machineLearningCh6/image-20201117172210434.png" alt="image-20201117172210434" style="zoom:50%;" />

通过在期望层切割树状图，每个相连的组件形成一个聚类。

<img src="/assets/img/machineLearning/machineLearningCh6/image-20201117172429560.png" alt="image-20201117172429560" style="zoom:67%;" />

### 自底向上聚合聚类

- 开始于每个物体单独作为一个聚类
- 然后重复把最近的聚类聚合
- 直到只有一个聚类

### 自顶向下分割聚类

- 开始于所有数据在一个聚类中
- 考虑每一种可能的方式把聚类一分为二，选择最好的分割。
- 然后在两边重复进行2过程

### 最近的聚类对测量

- Single-Link：通过两个聚类最近点的距离描述聚类间距离

  <img src="/assets/img/machineLearning/machineLearningCh6/image-20201117192743353.png" alt="image-20201117192743353" style="zoom:80%;" />

- Complete-Link：通过两个聚类最远点的距离描述聚类间距离

  <img src="/assets/img/machineLearning/machineLearningCh6/image-20201117192820748.png" alt="image-20201117192820748" style="zoom:80%;" />

- Centroid：利用质心距离来描述聚类间距离

  <img src="/assets/img/machineLearning/machineLearningCh6/image-20201117192850537.png" alt="image-20201117192850537" style="zoom:80%;" />

- Average-Link：利用两个聚类间每一对点的距离平均描述聚类间距离

### 计算复杂度

- 所有层次聚类方法需要计算n个样例中每一对的相似度：$O(n^2)$
- 每一次迭代，需要排序选出最大的：$O(n^2\log n)$，然后需要更新合并聚类的相似度
- 如果问题简单处理，则需要$O(n^2\log n)$或$O(n^3)$



## 划分聚类

**划分方法**：取一种划分方式将n个对象划分进一个K聚类的集合中。

**准则**：

- 全局最优：枚举所有划分方式
- 有效的启发式方法：K-means和K-medoids算法

### K-means

**算法**：

- 输入：训练样本集$D$以及聚类的个数$k$
- 输出：簇划分$C=\{C_1,\dots,C_k\}$
- 初始化：随机初始化$k$个簇中心$\mu=\{\mu_1,\dots,\mu_k\}$
- 迭代：
  - 根据当前点距离最近的簇中心，将当前点归于该簇中心所在簇
  - 重新计算簇中心$\mu'=\frac{1}{C_k}\sum_{i \in C_k}x_i$
- 终止条件：如果最后一次迭代所有的成员所在簇没有变动。

**计算复杂度**：

- 每次迭代：
  - 计算每个对象到质心的距离$O(kn)$
  - 计算质心：每个对象加到质心计算$O(n)$
- 假设进行$l$次迭代$O(lkn)$



# EM算法

## EM算法看K-means

**K-means算法在优化什么？**

- 有关中心$\mu$和簇划分$C$的潜在函数：$F(\mu,C)=\sum_{j=1}^{m}\|\|\mu_C(j)-x_j\|\|^2$
- K-means在优化$min_{\mu}min_{C}F(\mu,C)$

**重新看K-means算法**：

- 固定$\mu$，优化$C$(贴标签/分类过程)：
- 
  $$
  \mathop{\min}\limits_{C(1),\dots,C(m)}\sum_{j=1}^{m}||\mu_{C(j)}-x_j||^2=\sum_{j=1}^{m}\mathop{\min}\limits_{C(j)}||\mu_{C(j)}-x_j||^2
  $$
  
- 固定$C$，优化$\mu$(重定中心过程)：
- 
  $$
  \mathop{\min}\limits_{\mu_1,\dots,\mu_k}\sum_{i=1}^{k}\sum_{j:C(j)=i}||\mu_i-x_j||^2=\sum_{i=1}^{k}\mathop{\min}\limits_{\mu_i}\sum_{j:C(j)=i}||\mu_i-x_j||^2
  $$
  

可以看到K-means算法被分为Expectation step和Maximization step，也即EM算法。

### 生成式模型(Generative Model)看K-means

假设数据来自K个相同方差的高斯分布的混合，不妨设第$i$个分布的均值为$\mu_i$，且每个分布的协方差矩阵均为$\sigma^2I$

每个训练样本产生步骤：

- 以$P(y=i)$的概率选择第$i$个高斯分布

- 以第$i$个高斯分布产生样本$x\sim N(\mu_i,\sigma^2I)$
- 
  $$
  p(x|y=i)\sim N(\mu_i,\sigma^2I)\\
  p(x)=\sum_ip(x|y=i)P(y=i)
  $$
  
  $p(x)$前一部分为混合成份(Mixture component)，后一部分为混合比例(Mixture proportion)

**通过高斯贝叶斯分类器进行推导：**

$$
\log\frac{P(Y=i|X)}{P(Y=j|X)}=\log\frac{p(x|Y=i)P(Y=i)}{p(x|Y=j)P(Y=j)}\\
=\log\frac{P(Y=i)}{P(Y=j)}+\log\exp(\frac{1}{2}(x-\mu_j)^T(\sigma^2I)^{-1}(x-\mu_j)-\frac{1}{2}(x-\mu_i)^T(\sigma^2I)^{-1}(x-\mu_i))\\
=\log\frac{P(Y=i)}{P(Y=j)}+\frac{1}{2\sigma^2}(\mu_j^T\mu_j-2\mu_j^Tx-\mu_i^T\mu_i+2\mu_i^Tx)\\
=\log\frac{P(Y=i)}{P(Y=j)}+\frac{1}{\sigma^2}(\mu_i^T-\mu_j^T)x+\frac{1}{2\sigma^2}(\mu_j^T\mu_j-\mu_i^T\mu_i)
$$

令$w=\frac{1}{\sigma^2}(\mu_i^-\mu_j),b=\frac{1}{2\sigma^2}(\mu_j^T\mu_j-\mu_i^T\mu_i)+\log\frac{P(Y=i)}{P(Y=j)}$：

$$
\log\frac{P(Y=i|X)}{P(Y=j|X)}=w^Tx+b
$$

**MLE**：

由于不知道$y_i$，使用最大边缘密度似然：

$$
\arg\max\prod_jP(x_j)=\arg\max\prod_j\sum_{i=1}^{k}P(y_j=i,x_j)=\arg\max\prod_j\sum_{i=1}^{k}P(y_j=i)p(x_j|y_j=i)
$$

其中$p(y_j=i,x_j)\propto P(y_j=i)\exp(-\frac{1}{2\sigma^2}\|\|x_j-\mu_i\|\|^2)$，且$y=\{y_1,\dots,y_k\}$为one-of-k expression

所以有：

$$
\prod_j\sum_{i=1}^{k}P(y_j=i,x_j)\propto\prod_{j=1}^{m}\exp(-\frac{1}{2\sigma^2}||x_j-\mu_{C(j)}||^2)\\
\arg\max\prod_j\sum_{i=1}^{k}P(y_j=i)p(x_j|y_j=i)=\arg\max\prod_{j=1}^{m}\exp(-\frac{1}{2\sigma^2}||x_j-\mu_{C(j)}||^2)\\
=\arg\max\log\prod_{j=1}^{m}\exp(-\frac{1}{2\sigma^2}||x_j-\mu_{C(j)}||^2)=\arg\max\sum_{j=1}^{m}-\frac{1}{2\sigma^2}||x_j-\mu_{C(j)}||^2\\
=\arg\min\sum_{j=1}^{m}||x_j-\mu_{C(j)}||^2
$$

## K-means的分类效果不佳情况

- 聚类可能不线性可分
- 聚类可能交叠
- 一些聚类可能比其他"wider"

<img src="/assets/img/machineLearning/machineLearningCh6/image-20201117210159669.png" alt="image-20201117210159669" style="zoom: 50%;" />

## K-means的优缺点

**优点**

- 计算复杂度低，思想简单，容易实现

**缺点**

- 需要确定聚类数量k
- 分类结果依赖于簇中心的初始化
- 记过不一定是全局最优，只能保证局部最优
- 对噪声敏感，不能解决不规则形状的聚类

**性质**

- 假设数据呈现球形分布
- 假设各个簇的先验相同，但是簇数量可能不均匀
- 迭代过程实际等价于EM算法



# GMM

高斯混合分布可以被写作高斯分布的线性叠加：

$$
p(x)=\sum_{k=1}^{K}\pi_kN(x|\mu_k,\Sigma_k)
$$

引入K维二进制随机变量z，且z满足1-of-k representation：

$$
z_k\in\{0,1\},\sum_{k}z_k=1\\
p(z_k=1)=\pi_k,0\le\pi_k\le1,\sum_k\pi_k=1
$$

因为z使用1-of-K encoding，有：

$$
p(z)=\prod_{k=1}^{K}\pi_k^{z_k}
$$

那么可以把条件分布写为：

$$
p(x|z)=\prod_{k=1}^KN(x|\mu_k,\Sigma_k)^{z_k}
$$

x的边缘分布为：

$$
p(x)=\sum_zp(z)p(x|z)=\sum_{k=1}^K\pi_kN(x|\mu_k,\Sigma_k)
$$

根据贝叶斯公式：

$$
\gamma(z_k)=p(z_k=1|x)=\frac{p(z_k=1)p(x|z_k=1)}{\sum_{j=1}^Kp(z_j=1)p(x|z_j=1)}=\frac{\pi_kN(x|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x|\mu_j,\Sigma_j)}
$$


## MLE

假设我们有一个$k$维数据集$X=\{x_1,\dots,x_N\}$，则log-likelihood为：

$$
\ln p(X|\pi,\mu,\Sigma)=\sum_{n=1}^{N}\ln \sum_{k=1}^K\pi_kN(x_n|\mu_k,\Sigma_k)
$$

对$\mu_k$求导，可以得到：

$$
\frac{\partial\ln p}{\partial\mu_k}\propto\sum_{n=1}^{N}\frac{\pi_kN(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x_n|\mu_j,\Sigma_j)}\Sigma_k^{-1}(x_n-\mu_k)
$$

令其为0，容易得到：

$$
\mu_k=\frac{1}{N_k}\sum_{n=1}^{N}\gamma(z_{nk})x_n\\
where,\ \gamma(z_{nk})=\frac{\pi_kN(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x_n|\mu_j,\Sigma_j)},N_k=\sum_{n=1}^{N}\gamma(z_{nk})
$$

其中$N_k$可以看作分配到聚类$k$数据点数量。 



对$\Sigma_k$求导，得到（求导不会QAQ，直接写答案了）：

$$
\Sigma_k=\frac{1}{N_k}\sum_{n=1}^{N}\gamma(z_{nk})(x_n-\mu_k)(x_n-\mu_k)^T
$$



利用拉格朗日乘子法，重构原函数，得到：

$$
\ln p+\lambda(1-\sum_{k=1}^K\pi_k)
$$



对$\pi_k$求导，得到：

$$
\frac{\partial(\ln p+\lambda(1-\sum_{k=1}^K\pi_k))}{\partial\pi_k}\propto\sum_{n=1}^{N}\frac{N(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x_n|\mu_j,\Sigma_j)}-\lambda
$$

令其为0：

$$
0=\sum_{n=1}^{N}\frac{N(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x_n|\mu_j,\Sigma_j)}-\lambda
$$

两边同乘$\pi_k$：

$$
0=\sum_{n=1}^{N}\frac{\pi_kN(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x_n|\mu_j,\Sigma_j)}-\lambda\pi_k
$$

进行$k=\{1,\dots,K\}$求和：

$$
0=\sum_{k=1}^{K}\sum_{n=1}^{N}\frac{\pi_kN(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x_n|\mu_j,\Sigma_j)}-\lambda\sum_{k=1}^{K}\pi_k
$$

可以求解：

$$
\lambda=\sum_{k=1}^{K}\sum_{n=1}^{N}\gamma(z_{nk})=N
$$

那么：

$$
\pi_k=\frac{N_k}{N}
$$



此处对于$\mu_k,\Sigma_k,\pi_k$并未给出一个解析解，并且$\gamma(z_{nk})$还依赖于这些参数。但是，这些结果确实给出了一个简单的迭代方法寻找问题的最大似然解。

- 初始化均值$\mu_k$，方差$\Sigma_k$和混合比例$\pi_k$(一般$\mu_k$基于点，$\Sigma_k,\pi_k$初始为相同值；或者通过K-means算法先进行估计)

- E-step：使用当前参数值，估计$\gamma(z_{nk})$：
- 
  $$
  \gamma(z_{nk})=\frac{\pi_kN(x_n|\mu_k,\Sigma_k)}{\sum_{j=1}^K\pi_jN(x_n|\mu_j,\Sigma_j)}
  $$
  
- M-step：使用$\gamma(z_{nk})$重新估计模型参数：
- 
  $$
  \mu_k^{new}=\frac{1}{N_k}\sum_{n=1}^{N}\gamma(z_{nk})x_n,\ N_k=\sum_{n=1}^{N}\gamma(z_{nk})\\
  \Sigma_k=\frac{1}{N_k}\sum_{n=1}^{N}\gamma(z_{nk})(x_n-\mu_k^{new})(x_n-\mu_k^{new})^T\\
  \pi_k=\frac{N_k}{N}
  $$
  

## 与K-means对比

1. GMM-EM收敛前，迭代了更多次，每次迭代都需要更多计算量，通常通过K-means找到GMM的一个合适初始值，接下来使用GMM-EM调节。
2. K-means算法对数据点的聚类进行了硬分配，而GMM-EM基于后验概率进行软分配。

