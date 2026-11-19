---
title: 数据分布与金融机器学习
description: 从分布漂移与 OOD 泛化出发，讨论 RAVEN、DDG-DA 及金融机器学习中的数据配比思路。
published: 2026-11-19
tags:
  - 金融机器学习
  - 分布漂移
  - 论文阅读
draft: false
---

## 1. 问题背景

在金融领域的机器学习中，一直存在一个巨大的数据挑战：

- 信噪比异常低；
- 数据分布不平稳，存在概念漂移。

当训练集与测试集不满足 IID 条件（独立同分布）时，模型的泛化性和鲁棒性就会变得非常困难。我们在这篇文章中主要考虑 OOD 条件下金融机器学习的一些原理分析和解决方法。

### 1.1 分布漂移

我们先考虑监督学习中关于数据分布漂移的基本概念（主要参考 *Towards Out-Of-Distribution Generalization: A Survey*）。

最小化训练集上的平均误差，在分布漂移时可能对模型有害。因此，也许我们应该先优化训练集的分布。

分布偏移分为：

- **协变量偏移**：边际分布的变化 $P(X)$，也即输入变量（观测值）的分布变化；
- **概念偏移**：条件分布的变化 $P(Y \mid X)$，也即对于每一个输入变量，输出变量（特征标签）的分布变化。

### 1.2 面向 OOD 的监督学习泛化假设

#### 因果学习

学习潜在的不变因果结构：

$$
Y^e \leftarrow f_Y\left(X^e_{\operatorname{pa}(Y)}, \varepsilon_Y^e\right),
\qquad
\varepsilon_Y^e \perp X^e_{\operatorname{pa}(Y)}
$$

其中：

- $e \in \mathcal{E}$ 表示一个环境；
- $Y^e$ 表示环境 $e$ 中的目标变量；
- $X^e_{\operatorname{pa}(Y)}$ 表示环境 $e$ 中，$Y$ 的因果父节点对应的变量；
- $\operatorname{pa}(Y)$ 表示 $Y$ 在因果图中的父节点集合；
- $f_Y$ 表示生成目标变量 $Y$ 的结构方程；
- $\varepsilon_Y^e$ 表示环境 $e$ 中影响 $Y$ 的噪声项；
- $\perp$ 表示统计独立。

但是，通过DL直接找到因果关系也许比较困难，所以设计一个更接近因果框架的技术是必要的。

#### 不变学习

我们认为数据中存在不变特征：

$$
P\left(Y^e \mid X^e_{S^*}\right)
\text{ is the same for all } e \in \mathcal{E}
$$

其中：

- $\mathcal{E}$ 表示所有环境的集合；
- $S^* \subseteq \{1,\ldots,p\}$ 表示不变特征的索引集合；
- $X^e_{S^*}$ 表示环境 $e$ 中由不变特征组成的变量子集；
- $P\left(Y^e \mid X^e_{S^*}\right)$ 表示给定不变特征后目标变量的条件分布；
- “the same for all $e \in \mathcal{E}$” 表示该条件分布在所有环境中保持一致。

> 我们现在希望借助这两个假设，跨越分布偏移的鸿沟，接下来结合两篇论文来讨论一下方法。

## 2. RAVEN

RAVEN 对 `prompt`（时序信息）进行切割，并使用类似 MoE 的架构。实际上这里我们相当于是认为对于不同时间段的时序特征序列会属于不同的有限个**固定模式分布**，再根据不同的模式路由到不同的模型。

这里的技术比较复杂但是关键在于专家网络的架构，也即我们的模型因为接受到的不同长度序列不能当作同一分布，因此需要不同的模型来处理

### 2.1 输入表示

将输入窗口

$$
X \in \mathbb{R}^{L_{\max} \times D}
$$

变换为

$$
E = [e_1, \ldots, e_N] \in \mathbb{R}^{N \times d}.
$$

具体步骤如下：

1. 对 $X$ 进行归一化。
2. 将归一化后的输入分组为 $N$ 组：

   $$
   \{P_1, \ldots, P_N\}.
   $$

   其中，$X$ 表示有 $L$ 天、每一天有 $D$ 维特征的矩阵。
3. 对每个分组进行投影和正弦位置编码，得到 $d$ 维嵌入：

   $$
   e_i = \operatorname{PatchEmbed}(P_i) + \operatorname{PE}(i)
   \in \mathbb{R}^{d},
   \qquad i = 1, \ldots, N.
   $$

此时，我们得到了 $E$。

> 原文似乎没有给出具体的划分方式，也许可以使用分时或成交量作为 Bar 划分。

### 2.2 双分支骨干网络

一个双分支骨干网络接受矩阵 $E$。

#### 2.2.1 骨干网络一

##### 路由

先用两层 MLP 给 $e_i$ 评分，得到类别分布。**这个 MLP 的训练方法文中没有给出。**

首先计算评分：

$$
s_i = \phi(e_i).
$$

然后进行归一化：

$$
\tilde{s}_i
= \frac{\exp(s_i)}
{\sum_{j=1}^{N}\exp(s_j)},
\qquad i = 1,\ldots,N.
$$

设置阈值：

$$
0 < \tau_1 < \tau_2 < \cdots < \tau_K \leq 1.
$$

按时间的逆序累计分数。每当累计分数突破一个阈值，就路由到对应的模型。注意，这里是嵌套的而不是分割的，这出于金融时序的惯例：

$$
\Psi_i = \sum_{j=1}^{i}\tilde{s}_j,
$$

$$
G_k
= \left\{i \in \{1,\ldots,N\} : \Psi_i \leq \tau_k\right\}.
$$

##### 处理

对于数据 $G_k$，我们用专家 $k$ 进行处理。它是一个基于 Transformer 的三层 Encoder：

$$
H_k
= \operatorname{Encoder}_k\left(\{e_i\}_{i\in G_k}\right)
\in \mathbb{R}^{L_k \times d}.
$$

对于隐状态 $H_k$，我们进行池化：

$$
z_k = \operatorname{AvgPool}(H_k) \in \mathbb{R}^{d}.
$$

由于 $G_1 \subseteq \cdots \subseteq G_K$，嵌套后的数据可能包含前一个集合的信息，因此为每个专家分配不同的置信度。最终的专家权重为：

$$
w_k
= \frac{\alpha_k\exp(-\lambda r_k)}
{\sum_{k'=1}^{K}\alpha_{k'}\exp(-\lambda r_{k'})},
\qquad \lambda \geq 0.
$$

其中 $\lambda$ 为可学习参数。融合 $K$ 个专家后得到局部表示：

$$
z_{\mathrm{local}}
= \sum_{k=1}^{K}w_k z_k
\in \mathbb{R}^{d}.
$$

#### 2.2.2 骨干网络二

关注全局 Context：

$$
E_{\mathrm{sa}} = \operatorname{SelfAttn}(E),
$$

$$
z_{\mathrm{global}}
= \operatorname{AvgPool}(E_{\mathrm{sa}})
\in \mathbb{R}^{d}.
$$

### 2.3 输出

将结果投影到标量预测目标。

$$
\hat{y}_t(H)
= \operatorname{MLP}\left(
\operatorname{Concat}\left(
z_{\mathrm{local}}, z_{\mathrm{global}}
\right)\right)
\in \mathbb{R}.
$$

## 3. DDG-DA

这里我们认为市场风格的生成器具有一定的不变性，即前面的行情如何影响后面的行情，在一定程度上是稳定的。

DDG-DA 先给出未来数据的分布估计，再得到样本并训练模型：这里我们一定程度上改善了iid(用历史数据回测并得到模型从原理上是用了iid假设的，但是我们先预测并承认ood条件再训练模型从理论上讲会有所改善)

1. 先优化预测数据；
2. 再训练模型。

具体流程如下：

1. 接收训练数据，输出训练数据的分布信息。
2. 根据分布信息给出重采样概率。
3. 生成数据集。
4. 最小化重采样数据集与测试集之间的分布差异：

   $$
   p_{\mathrm{resam}}(x, y; \Theta)
   \quad \text{与} \quad
   p_{\mathrm{test}}(x, y)
   $$

是不是有点像llm预训练的scaling law：先用小模型测试数据配比再上大模型训练，我们这里也是先有了一堆数据(历史高开低收等行情信息)再得到最优数据配比后再训练我们的模型。

## 4. 历史任务示例

以下是某个历史任务。当时的“下个月”现在已经有真实标签。

### 4.1 小网络生成各时间块权重

```python
block_weights = weight_net(history_performance)
```

### 4.2 为每条历史样本分配所属时间块的权重

```python
w = block_weights[block_id]
```

### 4.3 使用加权历史数据求解线性回归

```python
phi = solve(
    X.T @ diag(w) @ X + alpha * I,
    X.T @ diag(w) @ y,
)
```

### 4.4 评估模型在下个月的预测效果

```python
pred = X_next @ phi
loss = prediction_loss(pred, y_next)
```

### 4.5 更新权重网络

梯度穿过上述求解过程，更新权重网络：

```python
optimizer.zero_grad()
loss.backward()
optimizer.step()
```


## "数据配比(data recipe)是大模型的命根"——tangjie
 从某些意义上讲也许对各种领域的建模都差不多，面对相似的困境，用着相似的思想
 
 这里我们处于一些原因没办法给出实证性的分析，但是面对ood问题我们可以总结一下监督学习的的解决方法的基本思想：先估计分布再训练，而不是只是简单的回测(因为你明明知道你的训练与测试数据不是iid的)
