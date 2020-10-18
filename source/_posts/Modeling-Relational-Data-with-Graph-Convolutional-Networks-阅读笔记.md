---
title: Modeling Relational Data with Graph Convolutional Networks 阅读笔记
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-08-22 21:53:24
password:
english_title: Modeling Relational Data with Graph Convolutional Networks
tags:
- ESWC
- 2018
- GCN
- Link Prediction
- Entity Classification
categories:
- 论文阅读笔记
---



# 1. 总览

## 要解决什么问题？（what？）

链接预测和实体分类

## 为什么要解决这个问题？（why？）

因为虽然知识图谱用途很多，而现有的知识图谱都存在不完整的问题。

## 用什么方法解决？（how？）

1. 用图卷积网络和因式分解相结合来解决链接预测问题；
2. 用图卷积网络单独解决实体分类问题。
3. 用权重矩阵的基本分解和块分解来解决参数随着关系数量增加而迅速增长的问题。

## 文章有什么创新？

1. 首次把GCN引入关系数据建模；
2. 提出了一种参数共享和增强稀疏限制的方法——权重矩阵的基本分解和块分解
3. 用GCN与因式分解组成auto-encoder的方法，可以提高因式分解模型在链接预测上的效果。

## 效果如何？

在FB15k-237上高出baseline 29.8%

## 还存在什么问题？

模型并没有优于基于CNN的模型

# 2 模型R-GCN

![](http://image.nysdy.com/20200823150715.png)



## 2.1 Relational graph convolutional networks

作者基于消息传递框架：
$$
h_{i}^{(l+1)}=\sigma\left(\sum_{m \in \mathcal{M}_{i}} g_{m}\left(h_{i}^{(l)}, h_{j}^{(l)}\right)\right)
$$

- 其中$h_{i}^{(l)} \in \mathbb{R}^{d^{(l)}}$是节点$v_i$在第$l$层神经网络的隐藏状态；$d^{(l)}$是当前层表示的维度；$g_m$是传入消息的累积形式。

设计了一个在关系多图中的传播策略：
$$
h_{i}^{(l+1)}=\sigma\left(\sum_{r \in \mathcal{R}} \sum_{j \in \mathcal{N}_{i}^{r}} \frac{1}{c_{i, r}} W_{r}^{(l)} h_{j}^{(l)}+W_{0}^{(l)} h_{i}^{(l)}\right)
$$

- 其中$\mathcal{N}_{i}^{r}$代表在关系$r \in \mathcal{R}$下节点$i$的邻居索引集合。其中$c_{i, r}$是一个特定于问题的归一化常数，可以预先学习或选择$c_{i, r}=\left|\mathcal{N}_{i}^{r}\right|$。$W_{0}^{(l)} h_{i}^{(l)}$是作者添加的对于每个节点的一个特定关系的自连接。
- 此处采取简单的线性消息转换，其实**可以选择更灵活的函数，如多层神经网络(以牺牲计算效率为代价)**。

## 2.2 Regularization | 正则化

**核心问题：**在处理多元关系数据时，图中参数数量和关系数量快速增长可能会导致对稀有关系的过拟合和模型规模过大。

为了解决这个问题作者提出了两种调整R-GCN层的权重的方式：

**basis-decomposition**:
$$
W_{r}^{(l)}=\sum_{b=1}^{B} a_{r b}^{(l)} V_{b}^{(l)} 
$$

- 其中$V_{b}^{(l)} \in \mathbb{R}^{d^{(l+1)} \times d^{(l)}}$是一个作为基础变换的线性组合，$a_{r b}^{(l)}$是一个只依赖于$r$的系数。
- 基函数分解可以看作是不同关系类型之间有效权重共享的一种形式



**block-diagonal-decomposition:**
$$
W_{r}^{(l)}=\bigoplus_{b=1}^{B} Q_{b r}^{(l)}
$$

- 其中$Q_{b, r}^{(l)} \in \mathbb{R}^{\frac{d(l+1)}{B} \times \frac{d^{(l)}}{B}}$,$W_{r}^{(l)}$是对角块矩阵：$\operatorname{diag}\left(Q_{1 r}^{(l)}, \ldots, Q_{B r}^{(l)}\right)$。
- 而块分解可以看做是每种关系类型对权重矩阵的系数约束。
- 块分解结构编码一种直觉，即可以将潜在特征分组为变量集，这些变量集在组内比组间更加紧密地联系。

同时，作者期望基本参数化可以减轻稀疏关系的过度拟合，因为稀疏关系和更频繁关系之间共享参数更新。

对所有的R-GCN模型都采取以下形式：

- 按照作者提出的传播模型进行堆叠$L$层
- 如果不存在其他特征，则可以将第一层的输入选择为图中每个节点的唯一one-hot向量；
- 对于块表示，作者将one-hot向量通过一个单一的线性转换为一个dense表示；
- 作者只考虑用一个featureless的方法，不同于GCN模型。

# 3 实体分类

![](http://image.nysdy.com/20200823152704.png)

作者通过堆叠R-GCN的传播函数，最后一层通过softmax函数输入，在所有节点上采用交叉熵损失（忽略无标签节点）：
$$
\mathcal{L}=-\sum_{i \in \mathcal{Y}} \sum_{k=1}^{K} t_{i k} \ln h_{i k}^{(L)}
$$

- 其中 $\mathcal{Y}$代表有标签的节点索引集，$h_{i k}^{(L)}$代表第$i$个标记节点的网络输出的第$k$个条目，$t_{ik}$表示真实标签。

# 4 链接预测

![](http://image.nysdy.com/20200823152727.png)

作者引入一个auto-encoder模型：

- encoder：将每个实体$v_{i} \in \mathcal{V}$ 映射到一个是指向量 $\vec{e}_{i} \in \mathbb{R}^{d}$ ；
- decoder：根据顶点表示重建图的边：通过一个scorce函数 $s: \mathbb{R}^{d} \times \mathcal{R} \times \mathbb{R}^{d} \rightarrow \mathbb{R}$ 来将三元组(subject, relation, object)映射为一个实数分数。

在本文中，作者采用的是DistMult函数作为解码器，每一个关系$r$被映射为一个对角矩阵 $R_{r} \in \mathbb{R}^{d \times d}$，一个三元组(s,r,o)的分数为：
$$
f(s, r, o)=e_{s}^{T} R_{r} e_{o}
$$
最后作者得出自己的损失函数：
$$
\mathcal{L}=-\frac{1}{(1+w)|\hat{\mathcal{E}}|} \sum_{(s, r, o, y) \in \mathcal{T}} y \log l(f(s, r, o))+(1-y) \log (1-l(f(s, r, o)))
$$

- 其中 $\mathcal{T}$ 是真实三元组和破坏得到的负样例三元组的总样本。

# 5 实验

![dataset](http://image.nysdy.com/20200823154310.png)

## 5.1 实体分类实验

> 数据集：AIFB、MUTAG、BGS、AM
>
> baseline：Feat、WL、RDF2Vec
>
> 评价准则：准确率

结果：![](http://image.nysdy.com/20200823154454.png)

## 5.2 链接预测

![dataset](http://image.nysdy.com/20200823154557.png)

> **关系抽取实验**：
>
> 数据集：WordNet(WN18)，Freebase(FB15K)
>
> baseline：LinkFeat,DistMult,CP,TransE,HolE,ComplEx
>
> 评价准则：MRR(mean reciprocal rank)（Raw，Filtered），Hits @（1，3，10）

实验结果：

![](http://image.nysdy.com/20200823154643.png)

- 其中 $f(s, r, t)_{\mathrm{R}-\mathrm{GCN+}} = \alpha f(s, r, t)_{\mathrm{R}-\mathrm{GCN}}+(1-\alpha) f(s, r, t)_{\text {DistMult }}$,R-GCN 和 DistMult都是各自训练好的。

# 参考链接

- https://blog.csdn.net/yyl424525/article/details/102764903

- https://blog.csdn.net/weixin_35505731/article/details/104581783

- 论文下载地址：[Modeling Relational Data with Graph Convolutional Networks](https://arxiv.org/pdf/1703.06103v4.pdf)

  