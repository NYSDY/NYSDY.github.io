---
title: NASE Learning Knowledge Graph Embedding for Link Prediction via Neural Architecture Search 阅读笔记
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-21 15:21:35
password:
english_title: Learning Knowledge Graph Embedding for Link Prediction via Neural Architecture Search
tags:
- Link Prediction
- Knowledge Representation
- 没读懂
categories:
- 论文阅读笔记
---



# 0. 导读

## 0.1 文章是关于什么的？（what？）

知识图谱表示，

## 0.2 要解决什么问题？（why？|challenge）

- AutoML只在双线性语义匹配的方法中搜索适合KG的表示方法，这显然不够全面；
- 

## 0.3 用什么方法解决？（how？）

- 提出了一种搜索方法，但是感觉具体的如何搜索并没有讲明白。

## 0.4文章有什么创新？

- 把搜索空间变为连续空间提高算法效率；
- 

## 0.5 效果如何？



## 0.6 还存在什么问题？

- 感觉没读懂，文中好多细节描述不够清晰
- 每一层是怎么选择用哪一种方法？
- g函数分解候选操作还有权重又是什么？

# 1 背景知识

我觉得在读这篇之前可能需要读一下两篇论文：

- Hanxiao Liu, Karen Simonyan, and Yiming Yang. 2018. Darts: Differentiable architecture search. arXiv preprint arXiv:1806.09055 (2018).

- Yongqi Zhang, Quanming Yao, Wenyuan Dai, and Lei Chen. 2020. AutoSF:

  Searching Scoring Functions for Knowledge Graph Embedding. In ICDE. IEEE.

# 2 模型

![](http://image.nysdy.com/20200921153104.png)

如图1所示，作者的的NAS框架包含两个搜索模块。 **表示搜索模块**旨在通过多个表示层分别优化头部，关系和尾部的嵌入$e_{h,} e_{r}, e_{t}$。 **得分函数搜索模块**负责选择浅层架构，以计算输入三元组的合理性得分。

## 2.1 表示搜索模块

该模块中，作者包含了卷积运算符和翻译运算符，分别对应了两种主流的reconstruction-based 链接预测模型。

图1中所示，每个操作都是一个融合步骤，作者以头实体嵌入为例定义了融合步骤如下：
$$
\begin{aligned} \boldsymbol{e}_{\boldsymbol{h}}^{l+1} &=\beta_{\boldsymbol{h}}^{l} \boldsymbol{e}_{\boldsymbol{h}}^{\boldsymbol{l}}+\left(1-\beta_{h}^{l}\right) \operatorname{opt}\left(\boldsymbol{e}_{r}^{l}, \boldsymbol{e}_{t}^{l}\right) \\ \beta_{h}^{l} &=\sigma\left(\boldsymbol{W}_{h}^{l}\left[\boldsymbol{e}_{h}^{l} ; \operatorname{opt}\left(\boldsymbol{e}_{r}^{l}, \boldsymbol{e}_{t}^{l}\right)\right]+b_{h}^{l}\right) \end{aligned}
$$

- 其中$e_{h}^{l}$是第l层的头嵌入输出，$\operatorname{opt}(\cdot, \cdot)$表示可搜索的运算符，$W_{h}^{l} \text { and } b_{h}^{l}$是可训练的参数。


引入融合步骤的目的是迫使新的嵌入集中在有关头部的信息上，而不是应该通过关系或尾部嵌入进行建模的事物。同样，该方法可以应用到关系和尾实体。

作者把候选运算符分为3类：

- 卷积运算符
  $$
  \boldsymbol{e}_{h}^{l+1}=\operatorname{ReLU}\left(\operatorname{Conv} 1 \mathrm{d}\left(\left[\boldsymbol{e}_{r}^{l} ; \boldsymbol{e}_{t}^{l}\right]\right)\right), \boldsymbol{e}_{h}^{l+1}=\operatorname{ReLU}\left(\operatorname{Conv} 2 \mathrm{d}\left(\left[\overline{\boldsymbol{e}}_{r}^{l} ; \overline{\boldsymbol{e}}_{t}^{l}\right]\right)\right)
  $$

  - $[\cdot ; \cdot]$代表row-wise 连接，$\overline{\boldsymbol{e}}$表示$e$的2D重建形状。

- 翻译运算符
  $$
  g_{r, 1}\left(e_{h}\right)+e_{r}-g_{r, 2}\left(e_{t}\right)=0
  $$

  - $g_{r,\cdot} (x)=\mathbf{W} \mathbf{x}$, 其中$\mathbf{W}$可以是单位矩阵或者无限制矩阵，这样就分别对应了TransE和TransR的方法。

- 本身运算符

  身份运算符无需任何转换即可直接将输入映射到输出。 引入这种运算符的目的是为了防止生成的体系结构过于复杂，并它能够使最终模型在得分函数搜索空间中退化为基本模型。

## 2.2 分数函数搜索模块

在此模块中，作者在几个基于语义匹配的模型中搜索以产生最终的合理性得分，该得分用于预测三元组是否有效。

- 基于卷积的分数函数：ConvKB

  - $$
    f(h, r, t)=W\left(\operatorname{ReLU}\left(\operatorname{Conv}\left(\left[e_{h} ; e_{r} ; e_{t}\right]\right)\right)\right)
    $$

- 基于翻译的分数函数：TransE
  $$
  f(h, r, t)=\left\|e_{h}+e_{r}-e_{t}\right\|_{p}
  $$
  
- 双线性分数函数：DistMult，SimplE
  $$
  \begin{array}{c}f(h, r, t)=\boldsymbol{e}_{h}^{T} \boldsymbol{M}_{\boldsymbol{r}} \boldsymbol{e}_{\boldsymbol{t}} \\ f(h, r, t)=1 / 2\left(\boldsymbol{e}_{h}^{T} \boldsymbol{M}_{r}, \boldsymbol{e}_{t}+\boldsymbol{e}_{h}^{T}, \boldsymbol{M}_{r}^{\prime}, \boldsymbol{e}_{t}\right)\end{array}
  $$
  
- MLP分数函数

  使用一个单一的隐藏层来处理三元组。

## 2.3 NASE搜索过程

作者把$k$个候选操作组合起来，形成一个函数如下：
$$
g=\sum_{i=1}^{k} a_{i} \varphi_{i}(x), \quad a_{i}=\frac{\exp \left(\alpha_{i}\right)}{\sum_{i=1}^{k} \exp \left(\alpha_{i}\right)}
$$

- 这里$g$代表了在表示搜索模块，每一层中应用的运算操作，也表示每层中的运算可以由多种操作同时进行然后给以不同权重作为选择过程。
- g还可以代表在分数函数搜索时的函数选择。

> 但是这里就又问题了，要是按照上面来说这是组合也不是选择了呀。

整个损失函数：
$$
\mathcal{L}=-\frac{1}{N} \sum_{i=1}^{N}\left(y_{i} \log (f(\cdot))+\left(1-y_{i}\right)(1-\log (f(\cdot)))\right.
$$


# 3 实验

常规实验，感觉没啥可以细讲的。

![](http://image.nysdy.com/20200921202107.png)

![](http://image.nysdy.com/20200921202135.png)

这个图就没咋懂，为啥三个颜色用的是不同的函数，这个是怎么选择出来的，论文中并没有解释。

![](http://image.nysdy.com/20200921202312.png)

# 参考链接

- 

