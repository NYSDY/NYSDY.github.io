---
title: Adaptive Graph Encoder for Attributed Graph Embedding 阅读笔记
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-08-31 20:02:39
password:
english_title: Adaptive Graph Encoder for Attributed Graph Embedding
tags:
- GCN
- adaptive learning
- attributed graph embedding
categories: 
- 论文阅读笔记
---



# 0. 总览

## 0.1 文章是关于什么的？（what？）

图卷积网络，属性图嵌入，自适应学习，拉布拉斯平滑

## 0.2 要解决什么问题？（why？|challenge）

已经存在的GCN-based 模型有三个主要缺陷：

- 作者实验显示图卷积网络的滤波器和权重矩阵的纠缠会损害性能和鲁棒性。
- 作者表明在这些方法中的图卷积滤波器是广义拉普拉斯平滑滤波器的特例，但它们并没有保留最佳的低通特性。
- 现有算法的训练目标通常是回复邻接矩阵或特征矩阵，而这些矩阵与现实应用不总是是一致的。

## 0.3 用什么方法解决？（how？）

作者提出了一个自适应图编码 Adaptive Graph Encoder (AGE)的属性图嵌入框架：

- 为了更好地避免节点特征中的高频噪音，作者首次应用了精心设计的拉普拉斯平滑滤波器。
- AGE采用自适应编码器，该编码器可以迭代地增强滤波后的功能，以实现更好的节点嵌入。

## 0.4文章有什么创新？

- 上述方法即为创新点。

## 0.5 效果如何？

AGE超过最好的图嵌入模型在节点聚类和链接预测任务上。

## 0.6 还存在什么问题？

# 1 背景知识

# 2 模型

## 符号标识

节点向量：$ \mathcal{V}=\left\{v_{1}, v_{2}, \cdots, v_{n}\right\}$

特征矩阵：$\mathbf{X}= \left[\mathbf{x}_{1}, \mathbf{x}_{2} \ldots, \mathbf{x}_{n}\right]^{T}$

邻接矩阵：$\mathbf{A}=\left\{a_{i j}\right\} \in \mathbb{R}^{n \times n}$, $a_{i j}=1 \text { if }\left(v_{i}, v_{j}\right) \in \mathcal{E}$

邻居矩阵$\mathbf{A}$度矩阵:$\mathbf{D}=\operatorname{diag}\left(d_{1}, d_{2}, \cdots, d_{n}\right) \in \mathbb{R}^{n \times n}$, $d_{i}=\sum_{v_{j} \in \mathcal{V}} a_{i j}$是节点$v_i$的度。

拉普拉斯矩阵：$\mathbf{L}=\mathbf{D}-\mathbf{A}$



属性图嵌入的目的是将节点映射到低维嵌入。 我们以Z为嵌入矩阵，并且嵌入应保留图G的拓扑结构和特征信息。

对于下游任务：

1. **节点聚类任务**：旨在将节点划分为𝑚个不相交的组{𝐺1，𝐺2，····，𝐺𝑚}，其中相似的节点应在同一组中。
2. **链接预测任务**：需要模型预测两个给定节点之间是否存在潜在边缘。

## 总体框架

![](http://image.nysdy.com/20200902122905.png)

- 拉普拉斯平滑滤波器：设计的滤波器$\mathbf{H}$用作低通滤波器，以对特征矩阵$\mathbf{X}$的高频分量进行去噪。平滑后的特征矩阵$\tilde{\mathbf{X}}$被用作自适应编码器的输入。
- 自适应编码器：为了获得更具代表性的节点嵌入，该模块通过自适应选择高度相似或不相似的节点对来构建训练集。 然后以监督的方式训练编码器。

## 2.1 拉普拉斯平滑滤波器

图学习基本假设：图上的邻近节点应该相似，因此，节点特征在图流形上应该是平滑的。

### 2.1.1 平滑信号

为了衡量信号$x$的平滑程度，计算图的Rayleigh quotient（瑞利商）：
$$
R(\mathrm{L}, \mathbf{x})=\frac{\mathbf{x}^{\top} \mathbf{L} \mathbf{x}}{\mathbf{x}^{\top} \mathbf{x}}=\frac{\sum_{(i, j) \in \mathcal{E}}\left(x_{i}-x_{j}\right)^{2}}{\sum_{i \in \mathcal{V}} x_{i}^{2}}
$$

- 该商实际上是$x$的标准化方差得分。 如上所述，平滑信号应在相邻节点上分配相似的值。 因此，具有较低瑞利商的信号被假定为更平滑。

考虑图的拉普拉斯特征分解：$\mathbf{L}=\mathbf{U} \mathbf{\Lambda} \mathbf{U}^{-1}$, $\mathbf{U} \in \mathbb{R}^{n \times n}$组成特征向量，$\Lambda=\operatorname{diag}\left(\lambda_{1}, \lambda_{2}, \cdots, \lambda_{n}\right)$是特征值的对角矩阵。所以特征向量$u_i$的平滑为：
$$
R\left(\mathbf{L}, \mathbf{u}_{i}\right)=\frac{\mathbf{u}_{i}^{\top} \mathbf{L} \mathbf{u}_{i}}{\mathbf{u}_{i}^{\top} \mathbf{u}_{i}}=\lambda_{i}
$$

- 该等式表示更平滑的特征向量与较小的特征值相关联，这意味着频率较低。

因此，作者基于上述两个等式创造出如下信号$x$:
$$
\mathbf{x}=\mathbf{U p}=\sum_{i=1}^{n} p_{i} \mathbf{u}_{i}
$$

- $p_i$是特征向量$u_i$的系数。

至此，$x$的平滑实际上是：
$$
R(\mathbf{L}, \mathbf{x})=\frac{\mathbf{x}^{\top} \mathbf{L} \mathbf{x}}{\mathbf{x}^{\top} \mathbf{x}}=\frac{\sum_{i=1}^{n} p_{i}^{2} \lambda_{i}}{\sum_{i=1}^{n} p_{i}^{2}}
$$

- 因此，为了获得更平滑的信号，我们的滤波器的目标是在滤除高频分量的同时保留低频分量。 由于其高计算效率和令人信服的性能，拉普拉斯平滑滤波器[28]通常用于此目的

### 2.1.2 广义拉普拉斯平滑滤波器

广义拉普拉斯平滑滤波器被定义为：
$$
\mathbf{H}=\mathbf{I}-k \mathbf{L}
$$

- $k$是一个实值。

使用$\mathbf{H}$作为滤波器矩阵，被过滤的信号$\tilde{\mathbf{x}}$为：
$$
\tilde{\mathbf{x}}=\mathbf{H x}=\mathbf{U}(\mathbf{I}-k \mathbf{\Lambda}) \mathbf{U}^{-1} \mathbf{U} \mathbf{p}=\sum_{i=1}^{n}\left(1-k \lambda_{i}\right) p_{i} \mathbf{u}_{i}=\sum_{i=1}^{n} p^{\prime} i \mathbf{u}_{i}
$$
因此，为了实现低通滤波，频率响应函数1-𝑘𝜆应该是一个减量和非负函数。 堆叠𝑡Laplacian平滑滤波器，我们将滤波后的特征矩阵表示为：
$$
\tilde{\mathbf{X}}=\mathbf{H}^{t} \mathbf{X}
$$

- 注意到，滤波器中是没有参数的

### 2.1.3 k的选择

在实际中，使用重归一化技巧：$\tilde{A}=I+A$，作者设计一个对称归一化图拉普拉斯：
$$
\tilde{\mathbf{L}}_{s y m}=\tilde{\mathbf{D}}^{-\frac{1}{2}} \tilde{\mathbf{L}} \tilde{\mathbf{D}}^{-\frac{1}{2}}
$$

- 其中$\tilde{\mathbf{D}}$和$\tilde{\mathbf{L}}$分别是与$\tilde{\mathbf{A}}$相对应的度矩阵和拉普拉斯矩阵

然后滤波器变成：
$$
\mathbf{H}=\mathbf{I}-k \tilde{\mathbf{L}}_{s y m}
$$

- 注意到，这里如果取$k=1$，则滤波器变成GCN滤波器。

为了选择最佳$k$，应该仔细地发现特征值$\tilde{\Lambda}$的分布（从$\tilde{\mathbf{L}}_{s y m}=\tilde{\mathbf{U}} \tilde{\Lambda} \tilde{\mathbf{U}}^{-1}$的分解获得）。

$\tilde{x}$的平滑为：
$$
R(\mathrm{L}, \tilde{\mathrm{x}})=\frac{\tilde{\mathbf{x}}^{\top} \mathrm{L} \tilde{\mathbf{x}}}{\tilde{\mathbf{x}}^{\top} \tilde{\mathbf{x}}}=\frac{\sum_{i=1}^{n} p_{i}^{2} \lambda_{i}}{\sum_{i=1}^{n} p_{i}^{2}}
$$

- 因此，$p \prime^2$随着$\lambda_i$增加应当减少。作者表示最大特征值为$\lambda_{max}$。
- $k<1 / \lambda_{\max }$就不能滤除所有高频部分，
- $k>1 / \lambda_{\max }$，滤波器不是一个低通滤波器在$\left(1 / k, \lambda_{\max }\right]$上，因为这段区间中，$p \prime^2$随着$\lambda_i$增加而增加。

## 2.2 自适应编码

对于属性图嵌入任务，两个节点之间的关系至关重要，这要求训练目标是合适的相似性度量。作者认为GAE方法选用邻接矩阵作为节点对的真实标签仅仅记录了一跳的结构信息，这远远不够。同时，平滑特征或经过训练的嵌入的相似度更加准确，因为它们将结构和特征融合在一起。为此，作者自适应地选择相似度高的节点对作为正训练样本，而相似度低的节点对作为负训练样本。

节点嵌入通过被过滤的节点特征线性编码为：
$$
\mathbf{Z}=f(\tilde{\mathbf{X}} ; \mathbf{W})=\tilde{\mathbf{X}} \mathbf{W}
$$

- $\mathbf{W}$是一个权重矩阵。
- 作者用了min-max scaler来讲嵌入缩减到[0,1]

作者使用余弦相似度来计算节点对的相似程度：
$$
\mathrm{s}=\frac{\mathrm{ZZ}^{\mathrm{T}}}{\|\mathrm{Z}\|_{2}^{2}}
$$

### 2.2.1训练负样例选择：

$r_{ij}$是节点对$\left(v_{i}, v_{j}\right)$的相似度将排序rank。节点对$\left(v_{i}, v_{j}\right)$的标签为：
$$
l_{i j}=\left\{\begin{array}{ll}1 & r_{i j} \leq r_{p o s} \\ 0 & r_{i j}>r_{n e g} \\ \text { None } & \text { otherwise }\end{array}\right.
$$

- 然后作者设置了正样例的最大rank值$r_{pos}$，和负样例的最小rank值$r_{neg}$。

用这种方法作者的训练集由$r_{pos}$个正样例和$n^2-r_{neg}$个负样例（因为负样例实际是远远多于正阳里的，所以这里作者生成的负样例比较多）

对于首次的训练集构建，由于encoder部分还未训练，直接应用平滑特征来计算初始的相似度矩阵：
$$
\mathrm{S}=\frac{\tilde{\mathbf{X}} \tilde{\mathbf{X}}^{\top}}{\|\tilde{\mathbf{X}}\|_{2}^{2}}
$$
在每个epoch中，作者随机采样$r_{pos}$个负样例来平和正负样例的数量。

作者采用交叉熵损失：
$$
\mathcal{L}=\sum_{\left(v_{i}, v_{j}\right) \in O}-l_{i j} \log \left(s_{i j}\right)-\left(1-l_{i j}\right) \log \left(1-s_{i j}\right)
$$

### 2.2.2 阈值更新

受课程学习(curriculum learning)理念的启发，作者针对$r_{pos}$和$r_{neg}$设计了一种特定的更新策略，以控制训练集的大小。

在训练过程开始时，会为编码器选择更多样本以找到粗糙的聚类模式。 之后，将保留较高置信度的样本以进行训练，从而迫使编码器捕获精炼的模式。 实际上，随着训练过程的进行，$r_{pos}$减小而$r_{neg}$线性增加。 所以作者设计了不同的开始和结束时的阈值，并随着更新的次数$T$来不断变化，以达到动态调整。
$$
\begin{array}{l}r_{p o s}^{\prime}=r_{p o s}+\frac{r_{p o s}^{e d}-r_{p o s}^{s t}}{T} \\ r_{n e g}^{\prime}=r_{n e g}+\frac{r_{n e g}^{e d}-r_{n e g}^{s t}}{T}\end{array}
$$
整个算法流程如下：

![](http://image.nysdy.com/20200902162217.png)

# 3 实验

## 3.1 数据集

![](http://image.nysdy.com/20200901165315.png)

> Features in Cora and Citeseer are binary word vectors, while in Wiki and Pubmed, nodes are associated with tf-idf weighted word vectors.

## 3.2 Baseline

- GAE and VGAE
- ARGA and ATVGA
- GALA

在节点聚类任务中可以分为以下三组：

- 只用特征模型：Kmeans 和 Spectral-Clustering
- 只用图结构模型:Spectral-G, DeepWalk
- 特征和图都用模型:TADW, MGAE, AGC, DAEGC

## 3.3 评价标准

节点聚类

- Accuracy (ACC), Normalized Mutual Information (NMI), and Adjusted Rand Index (ARI)

链接预测

- Area Under Curve (AUC) and Average Precision (AP) scores

## 3.4 实验结果

作者设计了辅助实验去回答以下三个假设：

1. 滤波器和权重矩阵的纠缠对嵌入质量没有提升；
2. 对比于重构损失，作者的自适应学习策略是有效的，并且每个机制有它自己的贡献；
3. 对于拉普拉斯平滑滤波器$k=1 / \lambda_{\max }$是最佳选择。

### 3.4.1 节点分类

![](http://image.nysdy.com/20200901204347.png)



### 3.4.2 链接预测结果

作者使用简单的内积来得到预测的邻接矩阵：
$$
\hat{\mathbf{A}}=\sigma\left(\mathbf{Z} \mathbf{Z}^{\top}\right)
$$

- 其中$\sigma$是sigmoid函数

![](http://image.nysdy.com/20200902100141.png)

GALA还为链接预测任务增加了重建损失，而AGE不使用显式链接进行监督。

### 3.4.3 GAE v.s. LS+RA

回答假设1.滤波器和权重矩阵的纠缠对嵌入质量没有提升；

GAE在每一层中结合力权重矩阵和滤波器，如图一所示；

LS+RA把权重矩阵移动到滤波器之后。

实验结果：![](http://image.nysdy.com/20200902102829.png)

![](http://image.nysdy.com/20200902100844.png)



### 3.4.4 消融实验

为了验证假设2：对比于重构损失，作者的自适应学习策略是有效的，并且每个机制有它自己的贡献；

LS+RA and LS+RX 展现出了可以和基线模型相比的表现，AGE模型要好于这两个模型说明自适应目标是更好的。

LS+RA 和 LS+RX分别在Cora, Wiki and Pubmed 和 Citeseer上表现效果更好。这说明了对于不同的数据集结构信息和特征信息的重要程度不同。此外，在Citeseer和Pubmed上，重建损失对平滑特征产生负面影响。

对Cora进行消融研究，以证明AGE中四种机制的功效。所有五个变体都通过对节点特征或嵌入的余弦相似度矩阵执行谱聚类来对节点进行聚类。

- “raw features”仅对原始节点特征执行频谱聚类；
- “ +Filter”使用平滑节点特征对节点进行聚类；
-  “ +Encoder”从平滑节点特征的相似度矩阵初始化训练集，并通过固定的训练集学习节点嵌入；
- “ +Adaptive”以固定的阈值自适应地选择训练样本；
-  “ +Thresholds Update”进一步添加了阈值更新策略，并且完全是完整模型。

![](http://image.nysdy.com/20200902104404.png)



### 3.4.5 $k$的选择

验证假设3：对于拉普拉斯平滑滤波器$k=1 / \lambda_{\max }$是最佳选择。

![](http://image.nysdy.com/20200902104911.png)

### 3.4.6 可视化

![](http://image.nysdy.com/20200902113935.png)



# 参考链接

- [论文原文下载地址](https://arxiv.org/pdf/2007.01594)

