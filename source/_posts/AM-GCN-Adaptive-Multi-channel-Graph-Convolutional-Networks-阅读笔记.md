---
title: 'AM-GCN: Adaptive Multi-channel Graph Convolutional Networks 阅读笔记'
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-16 10:32:15
password:
english_title: Adaptive Multi-channel Graph Convolutional Networks
tags:
- GCN
categories:
- 论文阅读笔记
---



# 0. 导读

## 0.1 文章是关于什么的？（what？）

图卷积网络

## 0.2 要解决什么问题？（why？|challenge）

- 目前最好的GCN模型在融合节点特征和拓扑结构的能力上不能使人满意；
- GCN真正从拓扑结构和节点特征中学习并融合了哪些信息？

## 0.3 用什么方法解决？（how？）

- 作者从节点特征，拓扑结构及其组合中同时抽取了特定和常见的嵌入，并使用注意力机制学习嵌入的自适应重要性权重。

## 0.4文章有什么创新？

- 研究了如何实现GCN的拓扑结构和节点特征的融合；
- 提出了用注意力机制来自适应融合拓扑结构和节点特征；
- 

## 0.5 效果如何？

- 在benchmark datasets 上超过了sota GCN

## 0.6 还存在什么问题？

# 1 背景知识

1. [Gram矩阵](https://blog.csdn.net/lilong117194/article/details/78202637)

## GCN融合能力实验

猜想：如果GCN可以自适应学习节点特征和拓补结构的话，那么调整网络和节点特征或者拓扑结构的相关性，GCN应该不会出现剧烈的效果差异变化。

作者设计了以下两个case来验证上述猜想：

### 随机拓扑和相关节点特征

这里作者建立了一个节点标签和节点特征高度相关，但是和拓扑结构无关。

对比模型：GCN和MLP

### 相关拓扑和随机节点特征

这里作者把节点分为3个团体，节点标签由团体决定。

对比模型：GCN和DeepWalk

### 总结

这些情况表明，目前的GCN融合机制[14]远未达到理想甚至令人满意的水平。

# 2 模型

AM-GCN整体模型框架如下图所示：![](http://image.nysdy.com/20200916114215.png)

## 2.1 特定卷积模型

**特征空间：**

作者首先构建一个$k$最近邻居（kNN)图： $G_{f}=\left(\mathbf{A}_{f}, \mathbf{X}\right)$依据节点特征矩阵$\mathbf{X}$，其中$\mathbf{A}_{f}$是kNN图的邻接矩阵。特别的，作者还计算量一个相似度矩阵$\mathbf{S} \in \mathbb{R}^{n \times n}$，并介绍了两种方式：

1. 余弦相似度：
   $$
   \mathbf{S}_{i j}=\frac{\mathbf{x}_{i} \cdot \mathbf{x}_{j}}{\left|\mathbf{x}_{i}\right|\left|\mathbf{x}_{j}\right|}
   $$

2. Heat Kernel:
   $$
   \mathbf{S}_{i j}=e^{-\frac{\left\|\mathbf{x}_{i}-\mathbf{x}_{j}\right\|^{2}}{t}}
   $$
   作者取：$t=2$

作者选择使用余弦相似度来计算，并且选择前k个相似度的节点建立边，最终得到$\mathbf{A}$。

至此， 作者可以得到第l层输出：
$$
\mathbf{Z}_{f}^{(l)}=\operatorname{ReLU}\left(\tilde{\mathbf{D}}_{f}^{-\frac{1}{2}} \tilde{\mathbf{A}}_{f} \tilde{\mathbf{D}}_{f}^{-\frac{1}{2}} \mathbf{Z}_{f}^{(l-1)} \mathbf{W}_{f}^{(l)}\right)
$$

- $ \mathbf{Z}_{f}^{(0)}=\mathbf{X}$, $\mathbf{w}_{f}^{(l)}$是GCN中第l层的权重矩阵
- $\tilde{\mathbf{A}}_{f}=\mathbf{A}_{f}+\mathbf{I}_{f}$， $\tilde{\mathbf{D}} f$是对角度矩阵。



**拓扑空间：**

整体流程和特征空间一样，只不过其中的一些矩阵替换如下：

- $G_{t}=\left(\mathbf{A}_{t}, \mathbf{X}_{t}\right) \text { where } \mathbf{A}_{t}=\mathbf{A} \text { and } \mathbf{X}_{t}=\mathbf{X}$



## 2.2 公共卷积模型

因为特征空间和拓扑空间不是完全不相关的，所以作者设计了一个Common-GCN来来提取被两个空间共享的公共信息。

从拓扑图中提取节点嵌入：
$$
\mathbf{Z}_{c t}^{(l)}=\operatorname{ReLU}\left(\tilde{\mathbf{D}}_{t}^{-\frac{1}{2}} \tilde{\mathbf{A}}_{t} \tilde{\mathbf{D}}_{t}^{-\frac{1}{2}} \mathbf{Z}_{c t}^{(l-1)} \mathbf{W}_{c}^{(l)}\right)
$$
从特征图中提取节点嵌入：
$$
\mathbf{Z}_{c f}^{(l)}=\operatorname{Re} L U\left(\tilde{\mathbf{D}}_{f}^{-\frac{1}{2}} \tilde{\mathbf{A}}_{f} \tilde{\mathbf{D}}_{f}^{-\frac{1}{2}} \mathbf{Z}_{c f}^{(l-1)} \mathbf{W}_{c}^{(l)}\right)
$$
合并两个输出嵌入变为公共嵌入：
$$
\mathbf{Z}_{C}=\left(\mathbf{Z}_{C T}+\mathbf{Z}_{C F}\right) / 2
$$

- 其中字母对应于2.1中字母，$\mathbf{W}_{c}^{(l)}$是一个共享的权重矩阵。

### 2.3 注意力机制

现在作者得到了3个向量，采用一个注意力机制来自适应学习三者的重要程度：
$$
\left(\alpha_{t}, \alpha_{c}, \alpha_{f}\right)=a t t\left(\mathbf{Z}_{T}, \mathbf{Z}_{C}, \mathbf{Z}_{F}\right)
$$
对于节点$i$的向量$ \mathbf{z}_{T}^{i} \in \mathbb{R}^{1 \times h}$在矩阵$\mathbf{Z}_{T} $中。为了获得注意力的值，作者首先通过一个非线性变换转换嵌入，然后使用一个共享的注意力向量$\mathbf{q} \in \mathbb{R}^{h^{\prime} \times 1}$去得到注意力值：
$$
\omega_{T}^{i}=\mathbf{q}^{T} \cdot \tanh \left(\mathbf{W}_{T} \cdot\left(\mathbf{z}_{T}^{i}\right)^{T}+\mathbf{b}_{T}\right)
$$

- 其中$\mathbf{e} \mathbf{W}_{T} \in \mathbb{R}^{h^{\prime} \times h}$和$\mathbf{b}_{T} \in \mathbb{R}^{h^{\prime} \times 1}$分别是罪域矩阵$\mathbf{z}_T$的权重矩阵和偏移向量；
- 注意，上角标的$T$应该是转置的意思。

同理可得其他两种向量的注意力值，最后权重为：
$$
\alpha_{T}^{i}=\operatorname{softmax}\left(\omega_{T}^{i}\right)=\frac{\exp \left(\omega_{T}^{i}\right)}{\exp \left(\omega_{T}^{i}\right)+\exp \left(\omega_{C}^{i}\right)+\exp \left(\omega_{F}^{i}\right)}
$$

- 该值越大说明对应的嵌入越重要。

最终获得的最终向量$\mathbf{Z}$表示为：
$$
\mathbf{Z}=\boldsymbol{\alpha}_{T} \cdot \mathbf{Z}_{T}+\boldsymbol{\alpha}_{C} \cdot \mathbf{Z}_{C}+\boldsymbol{\alpha}_{F} \cdot \mathbf{Z}_{F}
$$

- $\alpha_{T}=\operatorname{diag}\left(\alpha_{t}\right)$
- $\boldsymbol{\alpha}_{t}=\left[\alpha_{T}^{i}\right], \boldsymbol{\alpha}_{c}=\left[\alpha_{C}^{i}\right], \boldsymbol{\alpha}_{f}=\left[\alpha_{F}^{i}\right] \in \mathbb{R}^{n \times 1}$

## 2.4目标函数

### 2.4.1 Consistency Constraint 一致性限制

对于Common-GCN输出的两个向量$\mathbf{Z}_{C T} \text { and } \mathbf{Z}_{C F}$，作者谁记录一个一致性限制来进一步增强他们的通用性。

作者使用L2方式去归一化矩阵为$\mathbf{Z}_{C T n o r}, \mathbf{Z}_{C F n o r}$，并计算节点的相似度：
$$
\begin{array}{l}\mathbf{s}_{T}=\mathbf{Z}_{C T n o r} \cdot \mathbf{Z}_{C T n o r}^{T} \\ \mathbf{s}_{F}=\mathbf{Z}_{C F n o r} \cdot \mathbf{Z}_{C F n o r}^{T}\end{array}
$$
一致性意味着两个相似性矩阵应该相似，这引起以下约束：
$$
\mathcal{L}_{c}=\left\|\mathbf{S}_{T}-\mathbf{S}_{F}\right\|_{F}^{2}
$$

### 2.4.2 Disparity Constraint 差异限制

在这里，由于从同一图$G_{t}=\left(\mathbf{A}_{t}, \mathbf{X}_{t}\right)$学习嵌入$$\mathbf{Z}_{T} \text { and } \mathbf{Z}_{C T}$$，为确保它们可以捕获不同的信息，我们采用了希尔伯特-施密特独立性准则（HSIC），这很简单，但是有效的独立性措施，以扩大这两个嵌入之间的差距。

作者给予以下定义：
$$
H S I C\left(\mathbf{Z}_{T}, \mathbf{Z}_{C T}\right)=(n-1)^{-2} \operatorname{tr}\left(\mathbf{R} \mathbf{K}_{T} \mathbf{R} \mathbf{K}_{C T}\right)
$$

- $\mathbf{K}_{T} \text { and } \mathbf{K}_{C T}$是格拉姆矩阵(Gram matrices)，$k_{T, i j}=k_{T}\left(\mathbf{z}_{T}^{i}, \mathbf{z}_{T}^{j}\right)$and $k_{C T, i j}=k_{C T}\left(\mathbf{z}_{C T}^{i}, \mathbf{z}_{C T}^{j}\right)$
- $\mathbf{R}=\mathbf{I}-\frac{1}{n} e e^{T}$，$\mathbf{I}$是单位矩阵，$e$是一个全1列向量。

同理，得到$\mathbf{Z}_{F} \text { and } \mathbf{Z}_{C F}$的HSIC：
$$
H S I C\left(\mathbf{Z}_{F}, \mathbf{Z}_{C F}\right)=(n-1)^{-2} \operatorname{tr}\left(\mathbf{R} \mathbf{K}_{F} \mathbf{R} \mathbf{K}_{C F}\right)
$$
所以，差异化限制：
$$
\mathcal{L}_{d}=H S I C\left(\mathbf{Z}_{T}, \mathbf{Z}_{C T}\right)+H S I C\left(\mathbf{Z}_{F}, \mathbf{Z}_{C F}\right)
$$

### 2.4.3 优化目标

作者在下面等式中使用输出嵌入$\mathbf{Z}$用于具有线性变换和softmax函数的半监督多类分类:
$$
\hat{\mathbf{Y}}=\operatorname{softmax}(\mathbf{W} \cdot \mathbf{Z}+\mathbf{b})
$$

- $\hat{\mathbf{Y}}=\left[\hat{y}_{i c}\right] \in \mathbb{R}^{n \times C}$， $\hat{y}_{i c}$表示节点$i$属于类$c$的概率

对于节点分类任务，作者采用交叉熵损失函数：
$$
\mathcal{L}_{t}=-\sum_{l \in L} \sum_{i=1}^{C} \mathbf{Y}_{l} \ln \hat{\mathbf{Y}}_{l}
$$
**至此，**整个任务的目标函数如下：
$$
\mathcal{L}=\mathcal{L}_{t}+\gamma \mathcal{L}_{c}+\beta \mathcal{L}_{d}
$$

- $\gamma \text { and } \beta$分别是consistency and disparity constraint的超参数。



# 3 实验

## 3.1 数据集

![](http://image.nysdy.com/20200917102036.png)

## 3.2 baseline

DeepWalk，LINE，Chebyshev，GCN，kNN-GCN，GAT， DEMO-Net，MixHop

## 3.3 节点分类

![](http://image.nysdy.com/20200917102456.png)

- 与GCN和kNN-GCN相比，我们可以了解到拓扑图和特征图之间确实存在结构差异，并且在传统拓扑图上执行GCN并不总是比在特征图上显示更好的结果。 例如，在BlogCatalog，Flickr和UAI2010中，功能图的性能优于拓扑。 这进一步证实了在GCN中引入特征图的必要性。

### 3.4 变体分析

![](http://image.nysdy.com/20200917102800.png)

![](http://image.nysdy.com/20200917102855.png)

- 比较图2和表2的结果，可以发现AM-GCN-w/o尽管没有任何限制，但在基准方面仍然具有非常好的竞争性能，这表明作者的框架是稳定且具有竞争力的。

### 3.5 可视化展示

![](http://image.nysdy.com/20200917103107.png)

### 3.6 注意力机制分析

**分析注意力的分布**：分析数据集的构成，并且结合表2做分析。发现注意力的分布符合预期。

![](http://image.nysdy.com/20200917103448.png)

**注意力趋势分析**

![](http://image.nysdy.com/20200917103649.png)

### 3.7 调参分析

![](http://image.nysdy.com/20200917103736.png)

# 参考链接

- https://blog.csdn.net/lilong117194/article/details/78202637
- [论文下载地址](https://dl.acm.org/doi/pdf/10.1145/3394486.3403177)