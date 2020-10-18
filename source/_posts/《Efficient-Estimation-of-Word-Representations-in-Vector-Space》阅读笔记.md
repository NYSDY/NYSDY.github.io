---
title: 《Efficient Estimation of Word Representations in Vector Space》阅读笔记
date: 2019-04-30 10:37:23
tags:
- word2vec
english_title: Efficient Estimation of Word Representations in Vector Space
categories:
- 论文阅读笔记
---

> [论文下载地址](https://arxiv.org/pdf/1301.3781.pdf)，该篇论文的大篇幅都在讨论实验结果的分析，模型的部分比较简单，没有详细分析，本来是想读一下CBOW和skip-gram的原始论文，发现并没有想象中的那么大的用处。

<!-- more -->

## Goals of paper

- 开发了两种新模型，并保留了单词之间的线性规律
- 设计了一个新的综合测试集，用于测量句法和语义规律
- 讨论了训练时间和准确性如何取决于单词向量的维度和训练数据的数量

## Model Architectures

训练复杂度：

![20190506155712909488421.png](http://image.nysdy.com/20190506155712909488421.png)

其中，E是训练次数，T是训练集单词数量，Q是模型结构。

### Feedforward Neural Net Language Model (NNLM)

它由输入，映射，隐藏和输出层组成。通过简化方法，Q= N x D x H

### Recurrent Neural Net Language Model (RNNLM)

克服了模型需要固定的上下文长度的问题，并且只有输入，隐藏和输出层。

Q= H x H + H x V，其中H = D（单词表示），H x V 可以通过分级softmax被简化为H x log_2(V)。所以主要的复杂度来自于H x H。

### Parallel Training of Neural Networks

模型使用的DistBelief框架允许我们并行运行同一模型的多个副本，每个副本通过集中的服务器同步其梯度更新，该服务器保留所有参数

## New Log-linear Models

大多数复杂性是由于模型中的非线性隐藏层引起的。模型结构如下：![20190506155713050638684.png](http://image.nysdy.com/20190506155713050638684.png)

### Continuous Bag-of-Words Model(CBOW)

第一个提出的体系结构类似于前馈NNLM，其中去除了非线性隐藏层，并且所有单词（不仅仅是投影矩阵）共享投影层。 因此，所有单词都被投射到相同的位置（它们的向量被平均）。 将这个架构称为词袋模型，因为历史中的单词顺序不会影响投影。

模型的复杂度：Q = N × D + D × log_2(V )

### Continuous Skip-gram Model

基于同一句子中的另一个单词最大化单词的分类。 更准确地说，使用每个当前单词作为具有连续投影层的对数线性分类器的输入，并预测当前单词之前和之后的特定范围内的单词。

模型的复杂度：Q = C × (D + D × log2(V ))，其中C是单词的最大距离。

## 实验

### 任务描述 

为了度量词向量的质量，我们定义了一个复杂的测试集，它包括了五种类型的语义问题。九个类型的句法问题。包括每个类别的两个样本集在上表展示；总之，共拥有8869个语义问题和10675个句法问题

作者通过：最大化精确度 ，模型体系结构的比较，模型的大规模并行训练来证明提出模型的运速度和精确的优势。