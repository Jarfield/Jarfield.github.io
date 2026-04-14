---
title: softmax 与 cross entropy 求导入门
cover: /images/Covers/sky.jpg
date: 2026-04-14 19:40:00
tags:
  - 数学基础
  - 求导
  - softmax
  - 交叉熵
  - 反向传播
categories:
  - 学习笔记
mathjax: true
---

**金培晟** *Jarfield*

这篇笔记只讲一个在深度学习里极其高频的结论：

$$
\nabla_z L = p-y
$$

其中 $z$ 是 logits，$p=\mathrm{softmax}(z)$，$y$ 是标签分布。  
很多人会记这个结果，但不清楚它为什么成立，也不清楚它在 batch、one-hot、soft label 以及数值稳定写法下分别意味着什么。

<!-- more -->

## 0. 问题设置

设一个 $C$ 分类问题的输出 logits 为

$$
z = (z_1,\dots,z_C)^\top \in \mathbb R^C.
$$

经过 softmax 后得到预测概率

$$
p_i = \frac{e^{z_i}}{\sum_{k=1}^C e^{z_k}}, \qquad i=1,\dots,C.
$$

标签记作

$$
y = (y_1,\dots,y_C)^\top \in \mathbb R^C.
$$

通常有两种情形：

1. **one-hot 标签**：某一类为 1，其余为 0
2. **soft label**：$y_i \ge 0$ 且 $\sum_i y_i = 1$

交叉熵损失定义为

$$
L = -\sum_{i=1}^C y_i \log p_i.
$$

我们的目标是求

$$
\nabla_z L.
$$

## 1. 先看 softmax 本身的导数

### 1.1 softmax 不是逐元素函数

这一点要先强调。  
sigmoid 是逐元素的，但 softmax 不是，因为每个 $p_i$ 都依赖所有 $z_j$：

$$
p_i = \frac{e^{z_i}}{\sum_k e^{z_k}}.
$$

因此 softmax 的导数不是一个简单的逐元素乘法，而是一个 Jacobian 矩阵。

### 1.2 求 $\partial p_i / \partial z_j$

分两种情况。

#### 情况 1：$i=j$

$$
\frac{\partial p_i}{\partial z_i}
=
\frac{e^{z_i}\sum_k e^{z_k} - e^{z_i}e^{z_i}}{\left(\sum_k e^{z_k}\right)^2}
=
p_i(1-p_i).
$$

#### 情况 2：$i\neq j$

$$
\frac{\partial p_i}{\partial z_j}
=
\frac{0\cdot \sum_k e^{z_k} - e^{z_i}e^{z_j}}{\left(\sum_k e^{z_k}\right)^2}
=
-p_ip_j.
$$

把这两种情况合并：

$$
\frac{\partial p_i}{\partial z_j}
=
p_i(\delta_{ij}-p_j),
$$

其中 $\delta_{ij}$ 是 Kronecker delta。

### 1.3 Jacobian 形式

因此 softmax 的 Jacobian 为

$$
J_{\mathrm{softmax}}(z)
=
\frac{\partial p}{\partial z}
=
\mathrm{diag}(p)-pp^\top.
$$

这个式子本身就很重要，因为它说明：

- 对角项是各分量自己的增长率
- 非对角项是类别之间的竞争耦合项

> softmax 的核心不是“归一化一下”，而是把所有类别绑在同一个概率单纯形上，因此一个 logit 上升时，别的类会被一起压下去。

## 2. 再看 cross entropy 对概率的导数

交叉熵写成

$$
L(p) = -\sum_{i=1}^C y_i \log p_i.
$$

对 $p_i$ 求偏导：

$$
\frac{\partial L}{\partial p_i}
=
-\frac{y_i}{p_i}.
$$

于是

$$
\nabla_p L
=
\left(-\frac{y_1}{p_1},\dots,-\frac{y_C}{p_C}\right)^\top.
$$

这一步本身还不简洁。  
真正漂亮的是把它继续通过 softmax 往 logits 回传。

## 3. 关键推导：为什么会变成 $p-y$

我们要求的是对 logits 的梯度：

$$
\frac{\partial L}{\partial z_j}
=
\sum_{i=1}^C \frac{\partial L}{\partial p_i}\frac{\partial p_i}{\partial z_j}.
$$

代入前两节结果：

$$
\frac{\partial L}{\partial z_j}
=
\sum_{i=1}^C \left(-\frac{y_i}{p_i}\right)p_i(\delta_{ij}-p_j).
$$

约掉 $p_i$：

$$
\frac{\partial L}{\partial z_j}
=
\sum_{i=1}^C -y_i(\delta_{ij}-p_j).
$$

展开：

$$
\frac{\partial L}{\partial z_j}
=
\sum_{i=1}^C (-y_i\delta_{ij}) + \sum_{i=1}^C y_i p_j.
$$

第一项是

$$
-y_j.
$$

第二项因为 $p_j$ 与求和变量 $i$ 无关，所以

$$
\sum_{i=1}^C y_i p_j = p_j \sum_{i=1}^C y_i.
$$

如果 $y$ 是概率分布，则

$$
\sum_{i=1}^C y_i = 1,
$$

因此

$$
\frac{\partial L}{\partial z_j} = p_j - y_j.
$$

向量形式就是

$$
\nabla_z L = p-y.
$$

这就是最经典的结果。

## 4. 为什么这个结果这么漂亮

如果只看链式法则，你可能会觉得：

- softmax 的 Jacobian 很复杂
- cross entropy 的导数也不简洁

但两者一组合，很多项直接抵消了。  
这不是巧合，而是因为：

1. softmax 把 logits 映到概率分布
2. cross entropy 正好是概率分布上的自然损失

所以这两个东西是高度匹配的一对。

> 我的理解是：`softmax + cross entropy` 的组合之所以统治多分类输出层，不只是因为“常用”，而是因为它在数学上把反传简化到了最干净的形态。

## 5. one-hot 标签下怎么理解

如果真实类别是第 $t$ 类，则 one-hot 标签满足

$$
y_t = 1,\qquad y_i=0\ (i\neq t).
$$

此时损失退化为

$$
L = -\log p_t.
$$

梯度为

$$
\nabla_z L = p-y.
$$

具体到分量：

$$
\frac{\partial L}{\partial z_t} = p_t - 1,
$$

$$
\frac{\partial L}{\partial z_j} = p_j,\qquad j\neq t.
$$

这表示：

- 正类 logit 希望被继续推高，因为当前梯度是负的（若 $p_t<1$）
- 负类 logit 希望被压低，因为梯度是正的

这和直觉完全一致。

## 6. soft label 下会发生什么

如果标签不是 one-hot，而是某种软分布，例如 label smoothing 或 distillation 里的教师分布，那么公式仍然不变：

$$
\nabla_z L = p-y.
$$

只是此时 $y$ 不再是某一维为 1，而是一个更平滑的目标分布。

这意味着：

- one-hot 训练是在逼近“硬目标”
- soft label 训练是在逼近“分布目标”

从梯度上看，二者完全统一，只是目标分布不同。

## 7. 批量形式

假设一个 batch 有 $B$ 个样本。  
把 logits、softmax 输出、标签分别按行堆叠成矩阵：

$$
Z \in \mathbb R^{B\times C},\qquad
P = \mathrm{softmax}(Z),\qquad
Y \in \mathbb R^{B\times C}.
$$

若损失取 batch 平均：

$$
L = -\frac{1}{B}\sum_{b=1}^B \sum_{i=1}^C Y_{bi}\log P_{bi},
$$

则对 logits 的梯度是

$$
\nabla_Z L = \frac{1}{B}(P-Y).
$$

若损失取 batch 求和，则没有 $\frac1B$：

$$
\nabla_Z L = P-Y.
$$

这一点在手推和代码对齐时非常重要，因为很多框架默认对 batch 做平均。

## 8. 接上线性层：输出层权重怎么求导

设最后一层线性变换为

$$
z = Wh + b,
$$

其中

$$
W \in \mathbb R^{C\times d},\quad
h \in \mathbb R^d,\quad
b \in \mathbb R^C.
$$

若定义

$$
\delta \equiv \nabla_z L = p-y,
$$

则由线性层反传公式直接得

$$
\nabla_W L = \delta h^\top,
$$

$$
\nabla_b L = \delta,
$$

$$
\nabla_h L = W^\top \delta.
$$

也就是说，输出层最关键的工作其实就是先得到

$$
\delta = p-y.
$$

一旦这步有了，剩下就是普通线性层反传。

## 9. 数值稳定性：为什么实现时不用直接先算 softmax 再取 log

直接计算

$$
\log p_i = \log \frac{e^{z_i}}{\sum_k e^{z_k}}
$$

在数值上可能不稳定，因为 $e^{z_i}$ 可能溢出。

更稳定的写法是：

$$
\log p_i = z_i - \log\sum_k e^{z_k}.
$$

进一步可以减去一个常数 $c=\max_k z_k$：

$$
\log\sum_k e^{z_k}
=
c + \log\sum_k e^{z_k-c}.
$$

这就是 `log-sum-exp trick`。

因此实际实现里常常直接用：

- `log_softmax`
- `cross_entropy_with_logits`

而不是自己先做 softmax 再做 log。

#### 关键点

数值稳定写法改变的是前向计算形式，不改变最终梯度：

$$
\nabla_z L = p-y
$$

仍然成立。

## 10. 二分类为什么会变成 sigmoid + BCE

二分类时你也可以用两维 softmax：

$$
z=(z_1,z_2).
$$

但更常见的是只保留一个 logit $a$，然后用 sigmoid：

$$
\hat y = \sigma(a) = \frac{1}{1+e^{-a}}.
$$

对应 binary cross entropy：

$$
L = -y\log \hat y - (1-y)\log(1-\hat y).
$$

其梯度同样化简成

$$
\frac{\partial L}{\partial a} = \hat y - y.
$$

所以你可以把它看成多分类结论在二分类下的对应版本：

- 多分类：$\nabla_z L = p-y$
- 二分类：$\frac{\partial L}{\partial a} = \hat y-y$

## 11. 一页总结

### 11.1 你需要直接记住的公式

softmax：

$$
p_i = \frac{e^{z_i}}{\sum_k e^{z_k}}
$$

softmax Jacobian：

$$
\frac{\partial p_i}{\partial z_j} = p_i(\delta_{ij}-p_j)
$$

cross entropy：

$$
L = -\sum_i y_i\log p_i
$$

最关键结论：

$$
\nabla_z L = p-y
$$

batch 平均：

$$
\nabla_Z L = \frac1B(P-Y)
$$

### 11.2 这组公式真正说明了什么

softmax 输出层的反向传播之所以简单，不是因为 softmax 或 cross entropy 本身简单，而是因为两者组合后发生了大量结构抵消。  
这使得 logits 的梯度直接等于：

- 预测分布
- 减去目标分布

也就是一个非常直观的“误差向量”。

## 12. 我的理解

我觉得 `p-y` 这个结果特别值得反复看，因为它几乎把分类学习的核心写成了最短形式：

- 预测偏高的类，梯度为正，会被压下去
- 预测偏低的类，梯度为负，会被抬上来

这比把 softmax 和交叉熵分别看成两个孤立模块清楚得多。  
真正起作用的是它们的组合。

> 所以如果从反向传播角度理解输出层，最关键的不是“最后用了 softmax”，而是“最后把 logits 上的误差直接变成了分布差值”。

## 13. 下一步

这组笔记写到这里，已经够支撑最基础的 MLP / 分类头反向传播。  
如果继续往后写，最自然的就是：

1. `batch norm / layer norm` 为什么比 softmax 难推得多
2. `attention` 里的矩阵链式法则怎么展开
3. Hessian、Gauss-Newton 和二阶方法如何和这些一阶梯度接上
