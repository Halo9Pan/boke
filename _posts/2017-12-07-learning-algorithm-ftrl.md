---
layout:     post
title:      FTRL算法
date:       2017-12-07
categories: Machine-Learning
---

## 在线学习

传统的离线批量算法无法有效地处理超大规模的数据集和在线数据流。
Online Learning不需要所有数据，可以以流式的方式处理任意数量的样本，是工业界做在线 **CTR** 预估时的常用算法。

其实现方式大致可分为两种：

+ 在线凸优化（Online learning convex optimization）
  + FOBOS
  + RDA
  + FTRL
+ 在线贝叶斯（Online learning Bayesian）
  + AdPredictor
  + PBODL

## FTRL

FTRL（Follow The Regularized Leader）算法就是 Online Learning 凸优化的一种。
Google [H. Brendan McMahan](https://research.google.com/pubs/author35837.html) 先后三年时间（2010年-2013年）从理论研究到实际工程化实现的 FTRL 算法：

+ 2010年： [Adaptive Bound Optimization for Online Convex Optimization](https://research.google.com/pubs/pub36483.html)  
理论性paper，但未显式地支持正则化项迭代
+ 2011年： [A Unified View of Regularized Dual Averaging and Mirror Descent with Implicit Updates](https://arxiv.org/abs/1009.3240)  
证明 regret bound 以及引入通用的正则化项
+ 2011年： [Follow-the-Regularized-Leader and Mirror Descent: Equivalence Theorems and L1 Regularization](https://research.google.com/pubs/pub37013.html)  
揭示 OGD, FOBOS, RDA 等算法与 FTRL 关系
+ 2013年： [Ad Click Prediction: a View from the Trenches](https://research.google.com/pubs/pub41159.html)  
给出了工程性实现，并且附带了详细的伪代码，开始被大规模应用

FTRL 在处理诸如逻辑回归之类的带非光滑正则化项（例如1范数，做模型复杂度控制和稀疏化）的凸优化问题上性能非常出色，

## 逻辑回归 LR

逻辑回归的**目标函数**可归纳为：

$$
\hat{w} = \arg\min_{w}\sum_{i}L(y_i, {f}_{w}(x_i)) + \lambda{C}({w})
$$

$$L$$为逻辑函数，$$\lambda$$是正则系数，$$\Omega$$是正则函数

而逻辑回归的**优化方法**，传统的批量方法每次迭代都对**存量**训练数据集进行计算：

+ 梯度下降法
  + 批量梯度下降（BGD）
  + 随机梯度下降（SGD）
  + 小批量梯度下降（MBGD）
+ 还有很对基于梯度下降的新的优化方法。因为目前深度学习火热，产生了很多对梯度下降改进的优化方法
+ 牛顿法、拟牛顿法（L-BFGS）。因为用到了二阶导数，所以一般对于光滑的正则约束项（如L2）效果好点。

梯度下降法优点是精度和收敛效果不错，但是批量梯度下降（BGD）用全量数据进行计算，计算代价太大，没法应用于数据流做在线学习。随机梯度下降（SGD）和小批量梯度下降（MBGD）倒是可以只用部分数据进行计算。

## 在线算法

典型的**线性函数**随机梯度下降（SGD）参数更新的公式为：$${w}' = {w} + \eta(y_i - f_{w}(x_i))x_i$$。如果每次不是取随机值，而是用从数据流中获得的新数据$$x_t$$计算梯度，就能完成在线学习的过程。

$$
{w}_t = {w}_{t-1} + \eta(y - f_{w_{t-1}}(x_t))x_t
$$

同样，**线性函数**小批量梯度下降（MBGD）参数更新的公式为：$${w}' = {w} + \eta\frac{1}{k}\sum_{i=1}^k(y_i - f_{w}(x_i))x_i$$。如果每次不是随机找小批量，而是将从数据流中获得的新数据$$x_t$$包含到小批量中计算梯度，也能完成在线学习的过程。

$$
{w}_t' = {w}_{t-1} + \eta\frac{1}{k}\sum_{i=1}^k(y_i - f_{w}(x_i))x_i, k << m, t \in K
$$

牛顿法、拟牛顿法算法实现少，巨耗内存。目前主流稳定的只有 SciPy 基于 fortune 的实现和 SparkML 的实现，很难用到在线系统中。

梯度下降类的方法在进行在线学习是有两点不足：

1. 简单的在线梯度下降很难产生真正稀疏的解，稀疏性在机器学习中是很重要的事情，尤其做工程应用，稀疏的特征会大大减少 predict 时的内存和复杂度。
1. 对于不可微点的迭代会存在一些问题。猜想是由于流式的使用新的输入数据，而新数据的$${w}$$也许是一直变化的，会导致在边界点计算出现问题。

