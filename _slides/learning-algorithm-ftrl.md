---
layout:     slide
title:      FTRL算法
date:       2017-12-07
categories: Machine-Learning
mathjax:    true
plotly:     true
---

<section>
  <h1>FTRL 算法介绍</h1>
</section>

<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
<section data-markdown data-separator="^\n---\n$" data-separator-vertical="^\n----\n$">
<script type="text/template">

# 在线学习

----

<!-- .slide: style="text-align: left;"> -->
传统的离线批量算法无法有效地处理超大规模的数据集和在线数据流。
Online Learning不需要所有数据，可以以流式的方式处理任意数量的样本，是工业界做在线 **CTR** 预估时的常用算法。

----

<!-- .slide: style="text-align: left;"> -->
+ 在线凸优化（Online learning convex optimization）
  + FOBOS
  + RDA
  + FTRL
+ 在线贝叶斯（Online learning Bayesian）
  + AdPredictor
  + PBODL

---

# FTRL

----

<!-- .slide: style="text-align: left;"> -->
FTRL（Follow The Regularized Leader）算法就是 Online Learning 凸优化的一种。
Google [H. Brendan McMahan](https://research.google.com/pubs/author35837.html) 先后三年时间（2010年-2013年）从理论研究到实际工程化实现的 FTRL 算法：

----

+ 2010： [Adaptive Bound Optimization for Online Convex Optimization](https://research.google.com/pubs/pub36483.html)  
+ 2011： [A Unified View of Regularized Dual Averaging and Mirror Descent with Implicit Updates](https://arxiv.org/abs/1009.3240)  
+ 2011： [Follow-the-Regularized-Leader and Mirror Descent: Equivalence Theorems and L1 Regularization](https://research.google.com/pubs/pub37013.html)  
+ 2013： [Ad Click Prediction: a View from the Trenches](https://research.google.com/pubs/pub41159.html)  

----

<!-- .slide: style="text-align: left;"> -->
FTRL 在处理诸如逻辑回归之类的带非光滑正则化项的凸优化问题上性能非常出色

---

# 逻辑回归

----

<!-- .slide: style="text-align: left;"> -->
逻辑回归的**目标函数**可归纳为：

$$
\hat{w} = \arg\min\_{w}\sum\_{i}L(y\_i, {h}\_{w}(x\_i)) + \lambda\Psi({w})
$$

$L$为逻辑函数，$\lambda$是正则系数，$\Psi$是正则函数

----

<!-- .slide: style="text-align: left;"> -->
而逻辑回归的**优化方法**，传统的批量方法每次迭代都对**存量**训练数据集进行计算：

----

+ 梯度下降法
  + 批量梯度下降（BGD）
  + 随机梯度下降（SGD）
  + 小批量梯度下降（MBGD）

----

+ 还有很对基于梯度下降的新的优化方法。因为目前深度学习火热，产生了很多对梯度下降改进的优化方法

----

+ 牛顿法、拟牛顿法（L-BFGS）。因为用到了二阶导数，所以一般对于光滑的正则约束项（如L2）效果好点。

----

<!-- .slide: style="text-align: left;"> -->
梯度下降法优点是精度和收敛效果不错，但是批量梯度下降（BGD）用全量数据进行计算，计算代价太大，没法应用于数据流做在线学习。  
随机梯度下降（SGD）和小批量梯度下降（MBGD）倒是可以只用部分数据进行计算。

---

# 在线学习算法

----

<!-- .slide: style="text-align: left;"> -->
典型的**线性函数**随机梯度下降（SGD）参数更新的公式为：${w}' = {w} - \eta(y\_i - h\_{w}(x\_i))\cdot{x\_i}$  
如果每次不是取随机值，而是用从数据流中获得的新数据$x\_t$计算梯度，就能完成在线学习的过程

$$
{w}\_{t+1} = {w}\_{t} - \eta\:(h\_{w\_{t}}(x\_{t+1}) - y)\cdot{x\_{t+1}}
$$

----

<!-- .slide: style="text-align: left;"> -->
**线性函数**小批量梯度下降（MBGD）参数更新的公式为：${w}' = {w} - \eta\frac{1}{k}\sum\_{i=1}^k(y\_i - h\_{w}(x\_i))\cdot{x\_i}$  
如果每次不是随机找小批量，而是将从数据流中获得的新数据$x_t$包含到小批量中计算梯度，也能完成在线学习的过程

$$
{w}\_{t+1}' = {w}\_{t} - \eta\:\frac{1}{k}\sum\_{i=1}^k(y\_i - h\_{w\_{t}}(x\_i))\cdot{x\_i}\\\\
k \ll m,\ t \in K
$$

---

# 在线梯度下降

----

梯度下降类的方法在进行在线学习是有两点不足：

1. 简单的在线梯度下降很难产生真正稀疏的解。
1. 对于不可微点的迭代会存在一些问题。

---

# 算法优化

----

### 添加 L1 正则项

----

$$
{w}\_{t+1} = {w}\_{t} - \eta\:\big(h\_{w\_{t}}(x\_{t+1}) - y\big)\cdot{x\_{t+1}} + \eta\:\lambda\:\psi({w\_{t}})
$$

----

### 暴力截断

----

$$
{w}\_{t+1} = T\_0\bigg(\Big({w}\_{t} - \eta\:\big(h\_{w\_{t}}(x\_{t+1}) - y\big)\cdot{x\_{t+1}}\Big),\ {d}\bigg)
$$

$$
T\_0(w'\_{t+1}, {d}) =
  \begin{cases}
    0     & \quad \text{if } \left|w'\_{t+1}\right| \leq {d}\\\\
    w'\_{t+1}  & \quad \text{otherwise}
  \end{cases}
$$

----

### 在 L1 正则的基础上做截断

----

$$
{w}\_{t+1} = T\_1\bigg(\Big({w}\_{t} - \eta\:\big(h\_{w\_{t}}(x\_{t+1}) - y\big)\cdot{x\_{t+1}} + \eta\:\lambda\:\psi({w\_{t}})\Big),\ {d}\bigg)
$$

$$
T\_1(w'\_{t+1}, {a}, {d}) =
  \begin{cases}
    max(0, w'\_{t+1} - {a})  & \quad \text{if } w'\_{t+1} \in [0, {d}]\\\\
    min(0, w'\_{t+1} - {a})  & \quad \text{if } w'\_{t+1} \in [-{d}, 0]\\\\
    w'\_{t+1}                & \quad \text{otherwise}
  \end{cases}
$$

----

### FOBOS

----

<!-- .slide: style="text-align: left;"> -->
前向后向切分（FOBOS，Forward Backward Splitting）是 John Duchi 和 Yoran Singer 提出的。在该算法中，权重的更新分成两个步骤：

----

$$
{w}\_{t+\frac{1}{2}} = {w}\_{t} - \eta\_{t}{g\_{t}}\\\\
{w}\_{t+1} = \arg\min\_{w}\\{\frac{1}{2}\left\|{w} - {w}\_{t+\frac{1}{2}}\right\|^2 + \eta\_{t+\frac{1}{2}}\:\lambda\:\psi({w\_{t}})\\}
$$ <!-- .element: class="fragment" data-fragment-index="1" -->

其中 ${g\_{t}}=\big(h\_{w\_{t}}(x\_{t+1}) - y\big)\cdot{x\_{t+1}}$
<!-- .element: class="fragment" data-fragment-index="2" -->

----

<!-- .slide: style="text-align: left;"> -->
合并公式：
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w}\_{t+1} = \arg\min\_{w}\\{\frac{1}{2}\left\|{w} - {w}\_{t} + \eta\_{t}{g\_{t}}\right\|^2 + \eta\_{t+\frac{1}{2}}\:\lambda\:\psi({w\_{t}})\\}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

----

<!-- .slide: style="text-align: left;"> -->
求偏导，权重更新公式:
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w}\_{t+1} = {w}\_{t} - \eta\_{t}{g\_{t}} - \eta\_{t+\frac{1}{2}}\:\lambda\:\partial\psi({w\_{t+1}})
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

更新后的 ${w}\_{t+1}$ 不仅和 ${w}\_{t}$ 有关，还和 $\psi({w\_{t+1}})$ 有关。
<!-- .element: class="fragment" data-fragment-index="3" -->

----

<!-- .slide: style="text-align: left;"> -->
在 L1 正则化下，FOBOS 的特征权重的各个维度的更新公式：
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w}\_{t+1} = sgn({w}\_{t} - \eta\_{t}{g\_{t}})\\\\max\\{0, \left|{w}\_{t} - \eta\_{t}{g\_{t}}\right| - \eta\_{t+\frac{1}{2}}\:\lambda\\}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

其中${w}$是特征权重${W}$的某一维度，可以对 ${W}$ 的每一个维度进行单独求解
<!-- .element: class="fragment" data-fragment-index="3" -->

----

<p data-height="540" data-theme-id="0" data-slug-hash="GyoWaj" data-default-tab="result" data-user="Halo9Pan" data-embed-version="2" data-pen-title="FOBOS" class="codepen">See the Pen <a href="https://codepen.io/Halo9Pan/pen/GyoWaj/">FOBOS</a> by Halo Pan (<a href="https://codepen.io/Halo9Pan">@Halo9Pan</a>) on <a href="https://codepen.io">CodePen</a>.</p>

---

### RDA

----

<!-- .slide: style="text-align: left;"> -->
RDA（Regularized Dual Averaging Algorithm）叫做正则对偶平均算法

----

<!-- .slide: style="text-align: left;"> -->
特征权重的更新策略
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w}\_{t+1} = \arg\min\_{w}\\{\bar{g\_{t}}\cdot{w\_t} + \lambda\:\psi({w\_{t}}) + \frac{\beta\_{t}}{t}\:\hbar({w\_{t}})\\}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

$\bar{g\_{t}}=\frac{1}{t}\sum\_{r=1}^t[\eta\_{t}\:\big(h\_{w\_{t}}(x\_{t+1}) - y\big)\cdot{x\_{t+1}}]\cdot{w}\_{t}$，包括了之前所有梯度的平均值；
$\psi({w\_{t}})$ 为正则项，$\lambda$ 为正则系数；
$\hbar({w\_{t}})$ 是一个严格凸函数，$\beta\_{t}$ 是一个非负递增序列
<!-- .element: class="fragment" data-fragment-index="3" -->

----

<!-- .slide: style="text-align: left;"> -->
在 L1 正则化下，选取$\beta\_{t}=\gamma\sqrt{t} \text{ with } \gamma>0$
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w}\_{t+1} = \arg\min\_{w}\\{\bar{g\_{t}}\cdot{w\_t} + \lambda\:\left|{w\_{t}}\right| + \frac{\gamma}{2\sqrt{t}}\:\left\|{w\_{t}}\right\|\\}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

----

<!-- .slide: style="text-align: left;"> -->
求偏导，得到更新公式：
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w\_{t+1}} =
  \begin{cases}
    0                                                                              & \quad \text{if } \left|\bar{g\_{t}}\right| < \lambda\\\\
    -\frac{\sqrt{t}}{\gamma}\big(\bar{g\_{t}} - \lambda\cdot sgn(\bar{g\_{t}})\big)  & \quad \text{otherwise}
  \end{cases}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

----

<p data-height="680" data-theme-id="0" data-slug-hash="OzNyMw" data-default-tab="result" data-user="Halo9Pan" data-embed-version="2" data-pen-title="RDA" class="codepen">See the Pen <a href="https://codepen.io/Halo9Pan/pen/OzNyMw/">RDA</a> by Halo Pan (<a href="https://codepen.io/Halo9Pan">@Halo9Pan</a>) on <a href="https://codepen.io">CodePen</a>.</p>

---

## FTRL

----

<!-- .slide: style="text-align: left;"> -->
FTRL 算法综合考虑了 FOBOS 和 RDA 对于梯度和正则项的优势和不足

----

<!-- .slide: style="text-align: left;"> -->
其特征权重的更新公式
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w}\_{t+1} = \arg\min\_{w}\\\\
\\{ {z\_{t}}\cdot{w\_t} + \lambda\_1\:\psi\_1({w\_{t}}) + \frac{1}{2}({\lambda\_2} + \sum\_{s=1}^t\sigma\_s)\cdot{w\_t}^2 \\}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->


其中 ${z\_{t}} = {z\_{t-1}} + {g\_{t} - \sigma\cdot{w\_{t}}}$
<!-- .element: class="fragment" data-fragment-index="3" -->

----

<!-- .slide: style="text-align: left;"> -->
在 L1 正则化下：
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w}\_{t+1} = \arg\min\_{w}\\\\
\\{ {z\_{t}}\cdot{w\_t} + \lambda\_1\:({w\_{t}}| + \frac{1}{2}({\lambda\_2} + \sum\_{s=1}^t\sigma\_s)\cdot{w\_t}^2 \\}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

----

<!-- .slide: style="text-align: left;"> -->
求偏导，更新公式：
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
{w\_{t+1}} =
  \begin{cases}
    0                                                                              & \quad \text{if } \left|{z\_{t}}\right| < \lambda\_1\\\\
    -(\lambda\_2+\sum\_{s=1}^t\sigma\_s)^{-1}\cdot\big({z\_{t}}-\lambda\_1\cdot sgn({z\_{t}})\big)  & \quad \text{otherwise}
  \end{cases}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

----

<!-- .slide: style="text-align: left;"> -->
- 在 SGD, FOBOS, RDA 的算法里面使用的是一个全局的学习率 $\eta\_{(t)}$，意味着学习率是一个正数并且逐渐递减，对每一个维度都是一样的 <!-- .element: class="fragment" data-fragment-index="1" -->
- 在 FTRL 算法里面，每个维度的学习率是不一样的 <!-- .element: class="fragment" data-fragment-index="2" -->
- FTRL 考虑了训练样本本身在不同特征上分布的不均匀性 <!-- .element: class="fragment" data-fragment-index="3" -->

----

<!-- .slide: style="text-align: left;"> -->
在 FTRL 中，维度$w^{i}$的学习率定义：
<!-- .element: class="fragment" data-fragment-index="1" -->

$$
\eta\_{t}^{i}=\frac{\alpha}{\beta+\sqrt{\sum\_{s=1}^{t}(g\_{s}^{i})^{2}}}
$$ <!-- .element: class="fragment" data-fragment-index="2" -->

定义 $\sigma\_{(1:t)}=\frac{1}{\eta\_{t}}$
<!-- .element: class="fragment" data-fragment-index="3" -->

$$
\sum\_{s=1}^{t}\sigma\_{s}=\frac{1}{\eta\_{t}^{i}}=\frac{\beta+\sqrt{\sum\_{s=1}^{t}(g\_{s}^{i})^{2}}}{\alpha}
$$ <!-- .element: class="fragment" data-fragment-index="4" -->

其中$\alpha$为学习率，$\beta$为学习率的平滑系数，$\lambda\_1$为 L1 正则系数，$\lambda\_2$为 L2 正则系数。
<!-- .element: class="fragment" data-fragment-index="5" -->

----

<p data-height="800" data-theme-id="0" data-slug-hash="JMEaab" data-default-tab="result" data-user="Halo9Pan" data-embed-version="2" data-pen-title="FTRL" class="codepen">See the Pen <a href="https://codepen.io/Halo9Pan/pen/JMEaab/">FTRL</a> by Halo Pan (<a href="https://codepen.io/Halo9Pan">@Halo9Pan</a>) on <a href="https://codepen.io">CodePen</a>.</p>

---

## 工程实现

----

<!-- .slide: style="text-align: left;"> -->
### Predict
因为添加了 L1 正则，训练结果 $w$ 很稀疏

----

### Training

----

<!-- .slide: style="text-align: left;"> -->
### 在线丢弃训练数据中很少出现的特征
1. Poisson Inclusion: 对某一维度特征所来的训练样本，以 $p$ 的概率接受并更新模型
1. Bloom Filter Inclusion: 用 bloom filter 从概率上做某一特征出现 k 次才更新

----

<!-- .slide: style="text-align: left;"> -->
### 浮点数重新编码

1. 特征权重不需要用32bit或64bit的浮点数存储，存储浪费空间
2. 16bit encoding，但是要注意处理 rounding 技术对 regret 带来的影响

----

<!-- .slide: style="text-align: left;"> -->
### 训练若干相似模型

1. 对同一份训练数据序列，同时训练多个相似的模型
1. 这些模型有各自独享的一些特征，也有一些共享的特征
1. 有的特征维度可以是各个模型独享的，可以用同样的数据训练

----

<!-- .slide: style="text-align: left;"> -->
### 模型共享

1. 多个模型公用一个特征存储（例如放到 redis 中），各个模型都更新这个共有的特征结构
1. 对于某一个模型，对于其所训练的特征向量的某一维，直接计算一个迭代结果并与旧值做一个平均

----

<!-- .slide: style="text-align: left;"> -->
### 使用正负样本的**数目**来计算梯度的和

----

<!-- .slide: style="text-align: left;"> -->
### 二次抽样

在实际中，CTR远小于50%，所以正样本更加有价值。通过对训练数据集进行二次抽样，可以大大减小训练数据集的大小。

---

## 行业情况

----

<!-- .slide: style="text-align: left;"> -->
- LR + FE <!-- .element: class="fragment" data-fragment-index="1" -->
- LR + GBDT <!-- .element: class="fragment" data-fragment-index="2" -->
- FTRL <!-- .element: class="fragment" data-fragment-index="3" -->
- FM/FFM <!-- .element: class="fragment" data-fragment-index="4" -->
- MLR <!-- .element: class="fragment" data-fragment-index="5" -->
- PBODL <!-- .element: class="fragment" data-fragment-index="6" -->
- DNN <!-- .element: class="fragment" data-fragment-index="7" -->


</script>
</section>

<section>
  <h1>END</h1>
</section>
