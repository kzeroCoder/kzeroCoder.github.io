---
title: 机器学习笔记(一)
author: Kzero Coder
date: 2020-10-10 15:29:00 +0800
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



# 简介和决策树

## 关于机器学习

机器学习是**寻找一种对自然/人工主题、现象或活动可预测且/或可执行的及其理解方法。**

<img src="/assets/img/machineLearning/machineLearningCh1/1.png" alt="image-20201013160916540" />



研究算法需要满足**PTE**

​    P：提高它的性能

​    T：在某项任务中

​    E：利用一些经验  

  

### **机器学习的一般泛型**



**监督学习**

$$
Given \space D=\left\{X_i,Y_i\right\},learn \space F(\cdot;\theta),s.t:Y_i=F(X_i) ,\space D^{new}=\left\{X_j\right\}\Longrightarrow \left\{Y_j\right\}
$$

**非监督学习**

$$
Given \space D=\left\{X_i\right\},learn \space F(\cdot;\theta),s.t:Y_i=F(X_i) ,\space D^{new}=\left\{X_j\right\}\Longrightarrow \left\{Y_j\right\}
$$

**强化学习**

$$
Given \space D=\left\{env, actions, rewards, simulator/trace/real \space game\right\}\\
learn \space policy:e,r\rightarrow a, utility:a, e \rightarrow r, \space s.t \left\{env, new\space real\space game\right\}\Longrightarrow a_1,a_2,a_3\dots
$$


## 决策树

决策树可以表示输入属性的任何函数

如果对训练数据中的**每个样例都建立一条从根到叶的路径**（除非f对于输入x是不确定的）就得到一个一致的决策树，但其可能没有**泛化能力**

s.t ​**希望找一个更加紧凑（小规模）的决策树**

### **Top-Down的决策树归纳算法**

Main loop:

1. A <==下一个结点node的最好属性
2. 把A作为决策属性赋给结点node
3. 把A的每一个取值，创建一个新的儿子结点node
4. 把相应训练样本分到叶结点
5. 如果训练样本被很好分类，则停止，否则在新的叶结点上重复上述过程

### **贪心策略**

基于一个可以最优化某项准则的属性来切分示例集合

#### **确定待测条件**

##### **依赖于属性类型**

- 名词性/离散

- 有序的（不可将不相邻属性放在一个集合中）

- 连续（**离散化**构造有序类属性，或者**二分决策**）
  - *其中离散化分为**静态——在起始位置一次离散化**和**动态——范围可以通过等区间或等频率确定**或者是聚类*

##### **依赖于切分的分支数**

- 多路切分（一个属性值对应一路切分）

- 两路切分（离散属性值被切分为两个子集，**需要寻找最优切分**）

#### **确定最好的切分**

- 最理想情况是每个子集“皆为正例”或“皆为反例”，需要对结点混乱度(impurity)进行测量

<img src="/assets/img/machineLearning/machineLearningCh1/2.png" alt="image-20201013160916540" style="zoom:150%"/>

##### **熵(Entropy)**

随机变量X的熵H(X)是对从X随机采样值在最短编码情况下的每个值的平均长度（以2为底就是0、1编码）


$$
H(X)=-\sum_{i=1}^N P(x=i)log_2P(x=i)
$$

###### 条件熵

- 在X给定Y=v的**特定条件熵**$H(X\|Y=v)$：

$$
H(X|y=j)=-\sum_{i=1}^N P(x=i|y=j)log_2P(x=i|y=j)
$$

- X在给定Y的**条件熵**$H(X\|Y)$:

$$
H(X|Y)=\sum_{j=Val(y)}P(y=j)H(X|y=j)
$$

- X和Y的**互信息**：

$$
I(X;Y)=H(X)-H(X|Y)=H(Y)-H(Y|X)=H(X)+H(Y)-H(X,Y)
$$

###### 样本熵

- S是训练集例子的样本集

- $P_+$是S中的正例比例

- $P_-$是S中的反例比例

- 用熵类测量S的混杂度
  
  
  $$
  H(S)\equiv -p_+log_2p_++-p_-log_2p_-
  $$
  
  
  <img src="/assets/img/machineLearning/machineLearningCh1/3.png" alt="image-20201013160916540" style="zoom:150%"/>

##### **信息增益**


$$
GAIN_{split}=Entropy(p)-(\sum_{i=1}^k\frac{n_i}{n}Entropy(i))
$$



父结点P被切分成k部分；$n_i$是每一切分的样本数

**即目标类变量与属性A变量在S上的互信息**

用于测量由于切分带来的熵减少量，选择具有最大减少量的切分(即最大增益)

**缺点**：倾向于选择具有分支多的属性，因为每份样本可以很少，但很纯



##### 补充

- **交叉熵**

$$
H(p,q)=-\sum_{i=1}^{n}p(x_i)\log q(x_i)
$$

- **相对熵/KL散度**
  $$
  KL(p||q)=-H(q)+H(p,q)=\sum_{i=1}^{n}p(x_i)\log p(x_i)-\sum_{i=1}^{n}p(x_i)\log q(x_i)=\sum_{i=1}^{n}p(x_i)\log (p(x_i)/q(x_i))
  $$



### 树归纳的停止准则

- 当一个结点上所有样本属于同一个类别，停止扩展
- 当一个结点上所有样本具有相似属性值，停止扩展
- *提早结束*

### 基于决策树分类的优点

- 够检过程计算资源开销小
- 分类未知样本速度快
- 对于小规模的树比较容易理解
- 在许多小的简单数据集合上性能与其他方法相近

### 过拟合

过拟合导致**决策树比需要的更加复杂**（类似龙格现象）

通过**奥卡姆剃刀原理**选择最简树

#### **最小描述长度准则(Minimum Description Length)**

<img src="/assets/img/machineLearning/machineLearningCh1/4.png" alt="image-20201013160916540"/>

Cost(Model)为模型花费，Cost(Data\|Model)为不符合模型、需要另发的数据所需花费。

#### 解决过拟合的方法

- 前剪枝
  - 常规结点停止扩张情况：
    - 所有样本属于同一个类别
    - 所有样本具有相同属性值
  - 新增样本停止扩张情况：
    - 实例数量小于用户设定阈值
    - 实例的类分布和可用特征无关(通过卡方分布检测)
    - 扩展当前结点不会降低结点混乱度
- 后剪枝
  - 将树扩张到完全
  - 如果某结点的下一级为叶结点，且去掉叶结点使得树的误差减小，将叶结点收缩到上一级结点

### 对于确实属性值的处理

<img src="/assets/img/machineLearning/machineLearningCh1/5.png" alt="image-20201013160916540" style="zoom:80%"/><img src="/assets/img/machineLearning/machineLearningCh1/6.png" alt="image-20201013160916540"/>

通过其他数据属性的概率设置该项概率。

**对于后续加入的数据：**

<img src="/assets/img/machineLearning/machineLearningCh1/7.png" alt="image-20201013160916540" style="zoom:150%"/>

## 曲线拟合

### 问题目标

- 利用训练集合，对每一个$\overline{x}$，预测目标值$\overline{t}$

- 实质是学习真实函数(此处用sin函数)
  
  
  $$
  y(x,w)=w_0+w_1x+\dots+w_mx^n\\
  y(x,w)是x的多项式函数，w的线性函数
  $$



### 如何确定参数w(假定给定m)

- 建立误差函数，测量每个样本点目标值t与预测y之间的误差。**(实质是最小二乘问题)**误差函数如下：
  
  
  $$
  E(w)=\frac{1}{2}\left\{y(x_n,w)-t_n\right\}^2
  $$



- **拟合优度评价**
$$
E_{RMS}=\sqrt{2E(w^*)/N}
$$


<img src="/assets/img/machineLearning/machineLearningCh1/8.png" alt="image-20201013160916540" style="zoom:150%"/>

- **阶数高明显过拟合**

### 加入惩罚项抑制过拟合

由于参数多的时候，w^*^往往具有很大的绝对值，故可以在优化目标函数E(w)中加入对w的惩罚(正则)
$$
\widetilde{E}(w)=\frac{1}{2}\sum_{n=1}^N\left\{y(x_n,w)-t_n\right\}^2+\frac{\lambda}{2}||w||^2
$$

惩罚项比重大时降低模型复杂度，比重小时退化成原型。

<img src="/assets/img/machineLearning/machineLearningCh1/9.png" alt="image-20201013160916540" style="zoom:150%"/>

