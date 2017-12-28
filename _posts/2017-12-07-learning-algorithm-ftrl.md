---
layout:     post
title:      FTRL算法
date:       2017-12-07
categories: Machine-Learning
---

<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

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

## 逻辑回归

逻辑回归的**目标函数**可归纳为：

$$
\hat{w} = \arg\min_{w}\sum_{i}L(y_i, {h}_{w}(x_i)) + \lambda\Psi({w})
$$

$$L$$为逻辑函数，$$\lambda$$是正则系数，$$\Psi$$是正则函数
<div id="sigmod" style="width: 600px; height: 400px;">
<!-- Plotly chart will be drawn inside this DIV -->
</div>
<script>
(function() {
const samples = 200
const step = 10

let n = [...Array(samples).keys()]
let w = 0.8
let x = n.map(_n => (_n - (samples/2)) / step)
let y = x.map(_x => 1 / (1 + Math.exp(-(_x * w))))

let data = [
  {
    x: x,
    y: y,
    mode: 'lines'
  }
];

let layout = {
  title:'Sigmod'
};

Plotly.newPlot('sigmod', data, layout, {displayModeBar: false});
})()
</script>

而逻辑回归的**优化方法**，传统的批量方法每次迭代都对**存量**训练数据集进行计算：

+ 梯度下降法
  + 批量梯度下降（BGD）
  + 随机梯度下降（SGD）
  + 小批量梯度下降（MBGD）
+ 还有很对基于梯度下降的新的优化方法。因为目前深度学习火热，产生了很多对梯度下降改进的优化方法
+ 牛顿法、拟牛顿法（L-BFGS）。因为用到了二阶导数，所以一般对于光滑的正则约束项（如L2）效果好点。

梯度下降法优点是精度和收敛效果不错，但是批量梯度下降（BGD）用全量数据进行计算，计算代价太大，没法应用于数据流做在线学习。随机梯度下降（SGD）和小批量梯度下降（MBGD）倒是可以只用部分数据进行计算。

## 在线学习算法

典型的**线性函数**随机梯度下降（SGD）参数更新的公式为：$${w}' = {w} - \eta(y_i - h_{w}(x_i))\cdot{x_i}$$。如果每次不是取随机值，而是用从数据流中获得的新数据$$x_t$$计算梯度，就能完成在线学习的过程。

$$
{w}_{t+1} = {w}_{t} - \eta\:(h_{w_{t}}(x_{t+1}) - y)\cdot{x_{t+1}}
$$

姑且称为**在线梯度下降**。
同样，**线性函数**小批量梯度下降（MBGD）参数更新的公式为：$${w}' = {w} - \eta\frac{1}{k}\sum_{i=1}^k(y_i - h_{w}(x_i))\cdot{x_i}$$。如果每次不是随机找小批量，而是将从数据流中获得的新数据$$x_t$$包含到小批量中计算梯度，也能完成在线学习的过程。

$$
{w}_{t+1}' = {w}_{t} - \eta\:\frac{1}{k}\sum_{i=1}^k(y_i - h_{w_{t}}(x_i))\cdot{x_i},\ k \ll m,\ t \in K
$$

可以称为**在线小批量梯度下降**。

牛顿法、拟牛顿法算法实现少，巨耗内存。目前主流稳定的只有 SciPy 基于 fortune 的实现和 SparkML 的实现，很难用到在线系统中。

## 在线梯度下降

梯度下降类的方法在进行在线学习是有两点不足：

1. 简单的在线梯度下降很难产生真正稀疏的解，稀疏性在机器学习中是很重要的事情，尤其做工程应用，稀疏的特征会大大减少 inference 时的内存和复杂度。
1. 对于不可微点的迭代会存在一些问题。猜想是由于流式的使用新的输入数据，而新数据的$${w}$$也许是一直变化的，会导致在边界点计算出现问题。

在线 **CTR** 预估时的特征一般非常之多，因此我们希望得到更加稀疏的模型，这不仅仅起到了特征选择的作用，也降低了预测计算的复杂度。在实际使用 LR 的时候都会使用 L1 或者 L2 正则，避免模型过拟合。
在 BGD 算法下，L1 正则化通常能得到更加稀疏的模型。可是在 SGD 算法下，特别是在线学习的过程中，参数并不是沿着全局梯度下降，而是沿着某个样本的梯度进行下降，这样即使是 L1 正则也不一定能得到稀疏解。
因此在后面的优化算法中，稀疏性是一个主要追求的目标。

## 算法优化

1. #### 添加 L1 正则项

    如之前提到的，在线学习的过程中参数并不是沿着全局梯度下降，而是沿着某个样本的梯度进行下降，L1 正则不一定会得到稀疏解。另外如果是自己实现的梯度下降算法，需要针对 float 数据类型做简单的阈值截断，牵涉到浮点数精度问题，否则很难能到 0 解。

    $$
    {w}_{t+1} = {w}_{t} - \eta\:\big(h_{w_{t}}(x_{t+1}) - y\big)\cdot{x_{t+1}} + \eta\:\lambda\:\psi({w_{t}})
    $$

1. #### 暴力截断

    上面已经提到为了避免浮点数精度的问题需要做一些截断，当然其目的是为了很小的浮点数相加为0，是基于计算的考虑。沿着这个思路，很容易就能想到基于模型的考虑，也能做截断，这是最直观没技术含量的思路，设定一个阈值，做截断来保证稀疏。每在线训练N个数据截断一次。

    $$
    {w}_{t+1} = T_0\bigg(\Big({w}_{t} - \eta\:\big(h_{w_{t}}(x_{t+1}) - y\big)\cdot{x_{t+1}}\Big),\ {d}\bigg)
    $$

    $$
    T_0(w'_{t+1}, {d}) =
      \begin{cases}
        0     & \quad \text{if } \left|w'_{t+1}\right| \leq {d}\\
        w'_{t+1}  & \quad \text{otherwise}
      \end{cases}
    $$

    <div id="truncated-gradient-0" style="width: 400px; height: 400px;">
    <!-- Plotly chart will be drawn inside this DIV -->
    </div>
    <script>
    (function() {
    const range = 10
    const d = 2
    var layout = {
      title: 'Truncated Gradient T0',
      xaxis: {
        title: 'w',
        range: [-range, range],
        showticklabels: false
      },
      yaxis: {
        title: 'T0',
        range: [-range, range],
        showticklabels: false
      },
      width: 400,
      height: 400,
      annotations: [
        { x: -d, y: 0, xref: 'x', yref: 'y', text: '-d', showarrow: true, arrowhead: 7, ax: 0, ay: -40 },
        { x:  d, y: 0, xref: 'x', yref: 'y', text: ' d', showarrow: true, arrowhead: 7, ax: 0, ay:  40 }
      ]
    };
    let trace = {
      type: 'scatter',
      x: [-range, -d, -d, d, d, range],
      y: [-range, -d,  0, 0, d, range],
      mode: 'lines',
      name: 'Truncated Gradient',
      line: {
        color: 'blue',
        width: 3
      },
      hoverinfo: 'none'
    };
    Plotly.plot('truncated-gradient-0', [trace], layout, {displayModeBar: false});
    })()
    </script>

1. #### 在 L1 正则的基础上做截断

    很明显，简单截断的方法可以增加结果的稀疏性，但是会影响结果的精度。权重小的特征，可能是确实是无用特征，但也可能是在训练刚开始的阶段初始值本来很小、或者训练数据中包含该特征的样本数本来就很少。做为改进，很容易想到可以在添加了在 L1 正则的基础上做截断。

    $$
    {w}_{t+1} = T_1\bigg(\Big({w}_{t} - \eta\:\big(h_{w_{t}}(x_{t+1}) - y\big)\cdot{x_{t+1}} + \eta\:\lambda\:\psi({w_{t}})\Big),\ {d}\bigg)
    $$

    $$
    T_1(w'_{t+1}, {a}, {d}) =
      \begin{cases}
        max(0, w'_{t+1} - {a})  & \quad \text{if } w'_{t+1} \in [0, {d}]\\
        min(0, w'_{t+1} - {a})  & \quad \text{if } w'_{t+1} \in [-{d}, 0]\\
        w'_{t+1}                & \quad \text{otherwise}
      \end{cases}
    $$

    <div id="truncated-gradient-1" style="width: 400px; height: 400px;">
    <!-- Plotly chart will be drawn inside this DIV -->
    </div>
    <script>
    (function() {
    const range = 10
    const d = 2
    const a = 1
    var layout = {
      title: 'Truncated Gradient T1',
      xaxis: {
        title: 'w',
        range: [-range, range],
        showticklabels: false
      },
      yaxis: {
        title: 'T1',
        range: [-range, range],
        showticklabels: false
      },
      width: 400,
      height: 400,
      annotations: [
        { x: -d, y: 0, xref: 'x', yref: 'y', text: '-d', showarrow: true, arrowhead: 7, ax: 0, ay: -40 },
        { x:  d, y: 0, xref: 'x', yref: 'y', text: ' d', showarrow: true, arrowhead: 7, ax: 0, ay:  40 },
        { x: -a, y: 0, xref: 'x', yref: 'y', text: '-a', showarrow: true, arrowhead: 7, ax: 0, ay:  20 },
        { x:  a, y: 0, xref: 'x', yref: 'y', text: ' a', showarrow: true, arrowhead: 7, ax: 0, ay:  -20 },
      ]
    };
    var trace = {
      type: 'scatter',
      x: [-range, -d, -d,    -a, a, d,     d, range],
      y: [-range, -d, -(d-a), 0, 0, (d-a), d, range],
      mode: 'lines',
      name: 'Truncated Gradient',
      line: {
        color: 'blue',
        width: 3
      },
      hoverinfo: 'none'
    };
    Plotly.plot('truncated-gradient-1', [trace], layout, {displayModeBar: false});
    })()
    </script>

1. #### FOBOS

    前向后向切分（FOBOS，Forward Backward Splitting）是 John Duchi 和 Yoran Singer 提出的。在该算法中，权重的更新分成两个步骤：

    $$
    {w}_{t+\frac{1}{2}} = {w}_{t} - \eta_{t}{g_{t}}\\
    {w}_{t+1} = \arg\min_{w}\{\frac{1}{2}\left\|{w} - {w}_{t+\frac{1}{2}}\right\|^2 + \eta_{t+\frac{1}{2}}\:\lambda\:\psi({w_{t}})\}
    $$

    第一个步骤实际上是一个标准的梯度下降（SGD），其中$${g_{t}}=\big(h_{w_{t}}(x_{t+1}) - y\big)\cdot{x_{t+1}}$$，为当前梯度，第二个步骤是对第一个步骤的结果进行局部调整。写成一个公式那就是：

    $$
    {w}_{t+1} = \arg\min_{w}\{\frac{1}{2}\left\|{w} - {w}_{t} + \eta_{t}{g_{t}}\right\|^2 + \eta_{t+\frac{1}{2}}\:\lambda\:\psi({w_{t}})\}
    $$

    求偏导，得到权重更新的公式：

    $$
    {w}_{t+1} = {w}_{t} - \eta_{t}{g_{t}} - \eta_{t+\frac{1}{2}}\:\lambda\:\partial\psi({w_{t+1}})
    $$

    从上面的公式可以看出，更新后的 $${w}_{t+1}$$ 不仅和 $${w}_{t}$$ 有关，还和 $$\psi({w_{t+1}})$$ 有关，这也就是“前向后向切分”这个名称的由来。

    在 L1 正则化下，FOBOS 的特征权重的各个维度的更新公式是：

    $$
    {w}_{t+1} = sgn({w}_{t} - \eta_{t}{g_{t}})\ max\{0, \left|{w}_{t} - \eta_{t}{g_{t}}\right| - \eta_{t+\frac{1}{2}}\:\lambda\}
    $$

    其中$${w}$$是特征权重$${W}$$的某一维度，可以对 $${W}$$ 的每一个维度进行单独求解。

    <p data-height="540" data-theme-id="0" data-slug-hash="GyoWaj" data-default-tab="result" data-user="Halo9Pan" data-embed-version="2" data-pen-title="FOBOS" class="codepen">See the Pen <a href="https://codepen.io/Halo9Pan/pen/GyoWaj/">FOBOS</a> by Halo Pan (<a href="https://codepen.io/Halo9Pan">@Halo9Pan</a>) on <a href="https://codepen.io">CodePen</a>.</p>

    上面是一个一维$${w}$$的图，可以看出随着迭代次数的增加，$${w}$$会趋于稳定，但同时也能发现，$$\lambda$$取值对$${w}$$的影响。直观理解，因为$$\lambda$$是正则化系数，如果值太小，正则惩罚的力度太小，迭代更趋向于去拟合$${w}$$值。此处简化$$\eta_{t}$$和$$\eta_{t+\frac{1}{2}}$$取值一样，并且为固定值，但实际上可以为基于$$t$$的单调函数。

1. #### RDA

    RDA（Regularized Dual Averaging Algorithm）叫做正则对偶平均算法，特征权重的更新策略是：

    $$
    {w}_{t+1} = \arg\min_{w}\{\bar{g_{t}}\cdot{w_t} + \lambda\:\psi({w_{t}}) + \frac{\beta_{t}}{t}\:\hbar({w_{t}})\}
    $$

    $$\bar{g_{t}}=\frac{1}{t}\sum_{r=1}^t[\eta_{t}\:\big(h_{w_{t}}(x_{t+1}) - y\big)\cdot{x_{t+1}}]\cdot{w}_{t}$$，包括了之前所有梯度的平均值；
    $$\psi({w_{t}})$$为正则项，$$\lambda$$为正则系数；
    $$\hbar({w_{t}})$$是一个严格凸函数，$$\beta_{t}$$是一个非负递增序列

    在 L1 正则化下，选取$$\beta^{t}=\gamma\sqrt{t} \text{ with } \gamma>0$$。那么 RDA 算法就改写为：

    $$
    {w}_{t+1} = \arg\min_{w}\{\bar{g_{t}}\cdot{w_t} + \lambda\:\left|{w_{t}}\right| + \frac{\gamma}{2\sqrt{t}}\:\left\|{w_{t}}\right\|\}
    $$

    求偏导，得到更新公式：

    $$
    {w_{t+1}} =
      \begin{cases}
        0                                                                              & \quad \text{if } \left|\bar{g_{t}}\right| < \lambda\\
        -\frac{\sqrt{t}}{\gamma}\big(\bar{g_{t}} - \lambda\cdot sgn(\bar{g_{t}})\big)  & \quad \text{otherwise}
      \end{cases}
    $$

    <p data-height="680" data-theme-id="0" data-slug-hash="OzNyMw" data-default-tab="result" data-user="Halo9Pan" data-embed-version="2" data-pen-title="RDA" class="codepen">See the Pen <a href="https://codepen.io/Halo9Pan/pen/OzNyMw/">RDA</a> by Halo Pan (<a href="https://codepen.io/Halo9Pan">@Halo9Pan</a>) on <a href="https://codepen.io">CodePen</a>.</p>

1. #### FTRL

    FTRL 算法综合考虑了 FOBOS 和 RDA 对于梯度和正则项的优势和不足，其特征权重的更新公式是：

    $$
    {w}_{t+1} = \arg\min_{w}\{ {z_{t}}\cdot{w_t} + \lambda_1\:\psi_1({w_{t}}) + \frac{1}{2}({\lambda_2} + \sum_{s=1}^t\sigma_s)\cdot{w_t}^2 \}
    $$

    其中$${z_{t}} = {z_{t-1}} + {g_{t} - \sigma\cdot{w_{t}}}$$。
    在 L1 正则化下，算法就改写为：

    $$
    {w}_{t+1} = \arg\min_{w}\{ {z_{t}}\cdot{w_t} + \lambda_1\:({w_{t}}| + \frac{1}{2}({\lambda_2} + \sum_{s=1}^t\sigma_s)\cdot{w_t}^2 \}
    $$

    求偏导，得到更新公式：

    $$
    {w_{t+1}} =
      \begin{cases}
        0                                                                              & \quad \text{if } \left|{z_{t}}\right| < \lambda_1\\
        -(\lambda_2+\sum_{s=1}^t\sigma_s)^{-1}\cdot\big({z_{t}}-\lambda_1\cdot sgn({z_{t}})\big)  & \quad \text{otherwise}
      \end{cases}
    $$

    在 SGD, FOBOS, RDA 的算法里面使用的是一个全局的学习率 $$\eta_{(t)}$$，意味着学习率是一个正数并且逐渐递减，对每一个维度都是一样的。
    而在 FTRL 算法里面，每个维度的学习率是不一样的。
    FTRL 考虑了训练样本本身在不同特征上分布的不均匀性，如果某一个维度特征的训练样本很少，每一个样本都很珍贵，那么该特征维度对应的训练速率可以独自保持比较大的值，每来一个包含该特征的样本，就可以在该样本的梯度上前进一大步，而不需要与其他特征维度的前进步调强行保持一致。在 FTRL 中，维度$$w^{i}$$的学习率是这样定义的：

    $$
    \eta_{t}^{i}=\frac{\alpha}{\beta+\sqrt{\sum_{s=1}^{t}(g_{s}^{i})^{2}}}
    $$

    定义$$\sigma_{(1:t)}=\frac{1}{\eta_{t}}$$, 所以

    $$
    \sum_{s=1}^{t}\sigma_{s}=\frac{1}{\eta_{t}^{i}}=\frac{\beta+\sqrt{\sum_{s=1}^{t}(g_{s}^{i})^{2}}}{\alpha}
    $$

    其中$$\alpha$$为学习率，$$\beta$$为学习率的平滑系数，$$\lambda_1$$为 L1 正则系数，$$\lambda_2$$为 L2 正则系数。

    <p data-height="800" data-theme-id="0" data-slug-hash="JMEaab" data-default-tab="result" data-user="Halo9Pan" data-embed-version="2" data-pen-title="FTRL" class="codepen">See the Pen <a href="https://codepen.io/Halo9Pan/pen/JMEaab/">FTRL</a> by Halo Pan (<a href="https://codepen.io/Halo9Pan">@Halo9Pan</a>) on <a href="https://codepen.io">CodePen</a>.</p>

