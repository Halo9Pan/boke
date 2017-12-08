---
layout:     slide
title:      FTRL算法
date:       2017-12-07
categories: Machine-Learning
mathjax:    true
---

<section data-markdown data-separator="^\n---\n$" data-separator-vertical="^\n----\n$">
<script type="text/template">

# FTRL 算法

---

### 在线学习

----

<!-- .slide: style="text-align: left;"> -->
传统的离线批量算法无法有效地处理超大规模的数据集和在线数据流。
Online Learning不需要所有数据，可以以流式的方式处理任意数量的样本，是工业界做在线 **CTR** 预估时的常用算法。

----

<!-- .slide: style="text-align: left;"> -->
其实现方式大致可分为两种：

+ 在线凸优化（Online learning convex optimization）
  + FOBOS
  + RDA
  + FTRL
+ 在线贝叶斯（Online learning Bayesian）
  + AdPredictor
  + PBODL

----

<!-- .slide: style="text-align: left;"> -->
FTRL（Follow The Regularized Leader）算法就是 Online Learning 凸优化的一种。

---

### FTRL

<!-- .slide: style="text-align: left; font-size: medium;"> -->
Google [H. Brendan McMahan](https://research.google.com/pubs/author35837.html) 先后三年时间（2010年-2013年）从理论研究到实际工程化实现的 FTRL 算法：

+ 2010年： [Adaptive Bound Optimization for Online Convex Optimization](https://research.google.com/pubs/pub36483.html)  
理论性paper，但未显式地支持正则化项迭代
+ 2011年： [A Unified View of Regularized Dual Averaging and Mirror Descent with Implicit Updates](https://arxiv.org/abs/1009.3240)  
证明 regret bound 以及引入通用的正则化项
+ 2011年： [Follow-the-Regularized-Leader and Mirror Descent: Equivalence Theorems and L1 Regularization](https://research.google.com/pubs/pub37013.html)  
揭示 OGD, FOBOS, RDA 等算法与 FTRL 关系
+ 2013年： [Ad Click Prediction: a View from the Trenches](https://research.google.com/pubs/pub41159.html)  
给出了工程性实现，并且附带了详细的伪代码，开始被大规模应用

----

<!-- .slide: style="text-align: left;"> -->
$$
\hat{w} = \arg\min\_{w}\sum\_{i=1}^{n} L({w},z\_i) + g\lVert{w}\rVert\_1
$$

在处理诸如逻辑回归之类的带非光滑正则化项（例如1范数，做模型复杂度控制和稀疏化）的凸优化问题上性能非常出色，
</script>
</section>