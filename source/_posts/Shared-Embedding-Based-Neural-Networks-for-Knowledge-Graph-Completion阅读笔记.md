---
title: Shared Embedding Based Neural Networks for Knowledge Graph Completion阅读笔记
date: 2019-04-19 14:52:38
tags:
- 知识图谱
- 神经网络
- 知识图谱补全
english_title: Shared Embedding Based Neural Networks for Knowledge Graph Completion
categories:
- 论文阅读笔记
---

> [原文下载链接](http://delivery.acm.org/10.1145/3280000/3271704/p247-guan.pdf?ip=59.64.129.243&id=3271704&acc=ACTIVE%20SERVICE&key=BF85BBA5741FDC6E%2E66A15327C2E204FC%2E4D4702B0C3E38B35%2E4D4702B0C3E38B35&__acm__=1555657159_db1582f1a6ea923a16011064e5cc7955)，知识图谱补全（KGC，Knowledge Graph Completion)是一种自动建立图谱内部知识关联的工作。目标是补全知识图谱中三元组的缺失部分。主要方法为基于张量（或者矩阵）和基于翻译两类。在本文中，作者提出了一种基于共享嵌入的神经网络的模型（SENN）来处理KGC。

<!-- more -->

## Contribulation

- 提出了SENN模型，该模型明确区分头实体、关系和为实体预测任务，并把它们整合到一个基于全连接神经网络框架中，该框架共享的实体和关系嵌入。
- SENN提出了一个自适应全中损失机制，该方法可以很好的处理具有不同映射属性的三元组，并处理不同的预测任务。
- 由于关系预测通常比头尾实体预测具有更好的性能，我们把SENN应用到头尾实体预测，从而将SENN扩展到SENN+。

## Related works

### Tensor/Matrix Based Methods

RESCAL是一个典型的方法，该方法基于三向张量因子分解的方法。

目标函数为：![20190419155565817094503.png](http://image.nysdy.com/20190419155565817094503.png)

$M_r$是r的关系矩阵，大小为k x k。

ComlEx是最近提出的方法，该方法基于矩阵分解，并且它使用复数值来定义实体和关系的嵌入。

目标函数为：![20190419155565837140789.png](http://image.nysdy.com/20190419155565837140789.png)

Re(x)返回x的实部。

### Translation Based Methods

代表模型为经典的TransE模型（这里不再赘述）

### Translation Based Methods

ER-MLP使用多层感知器来捕获头实体，关系和尾实体之间的隐式交互。

目标函数为：![2019041915556586361053.png](http://image.nysdy.com/2019041915556586361053.png)

ProjE使用具有组合层和投影层的神经网络来对头尾实体预测建模。

## THE SENN METHOD

模型结构如图所示：![20190419155565880062506.png](http://image.nysdy.com/20190419155565880062506.png)

作者将框架划分为以下四个部分：

1. 三元组的批量预处理
2. 知识图谱的Shared embeddings表示学习
3. 独立的头尾实体及关系预测子模型训练与融合
4. 联合损失函数构成

整个KGC的流程可以描述如下：

1. 将训练数据中的完整三元组（知识图谱）划分批量后作为模型的输入
2. 对于输入的三元组，分别训练得到实体（包括头尾实体）嵌入矩阵与关系嵌入矩阵（embeddings）
3. 将头尾实体及关系embeddings分别输入到三个预测模型中（头实体预测（?, r, t），关系预测(h, ?, t)，尾实体预测(h, r, ?)）

### The Three Substructures

预测子模型具有相似的结构如下图，模型输入关系向量与实体向量后，进入n层全连接层，得到预测向量，再经过一个sigmoid（或者softmax）层，输出预测标签向量。![20190419155565925349707.png](http://image.nysdy.com/20190419155565925349707.png)

头实体预测目标函数：![20190419155565929066802.png](http://image.nysdy.com/20190419155565929066802.png)

f(x)= max(0,x).

预测标签：![20190419155565936981620.png](http://image.nysdy.com/20190419155565936981620.png)

其它两种与此头实体类似。

### Model Training

#### The General Loss Function

模型目标标签向量表示为：![20190419155565949172586.png](http://image.nysdy.com/20190419155565949172586.png)

$I_h$是在训练集中给定r和t的所有有效头实体集。

三者的平滑向量表示为：![20190419155565967448577.png](http://image.nysdy.com/20190419155565967448577.png)

三个预测任务的损失函数为：![20190419155565972110097.png](http://image.nysdy.com/20190419155565972110097.png)

总损失函数为：![20190419155565974814392.png](http://image.nysdy.com/20190419155565974814392.png)

#### The Adaptively Weighted Loss Mechanism.

该方法的动机：

- 在知识图谱中的三元组有4种类型：1-TO-1, 1-TO-M, M-TO-1 and M-TO-M。所以预测在训练集中具有的有效实体/关系越多，它就越不确定。所以作者将对应于头部实体预测，关系预测和尾部实体预测的损失的权重与有效实体的数量相关联。
- 因为关系预测比实体预测更加容易。所以作者加大对头尾实体的错误预测的惩罚。

所以作者得到新的损失函数：![20190419155566013038361.png](http://image.nysdy.com/20190419155566013038361.png)

总损失函数变为：![20190419155566016395908.png](http://image.nysdy.com/20190419155566016395908.png)

## THE SENN+METHOD

作者相信可以进一步利用关系预测的相当好的性能来辅助测试过程中的头部和尾部实体预测。

给定头部预测任务（？，r，t）并假设h是有效的头部实体。 如果我们采用SENN方法来预测h和t之间的关系，即执行关系预测任务（h，？，t），则关系r最有可能具有 预测标签高于其他关系，因此应排名高于其他关系。

![20190419155566073220975.png](http://image.nysdy.com/20190419155566073220975.png)

其中Value（x，r）返回对应于关系r的向量x的条目; Rank（x，r）以降序返回对应于关系r的向量x的条目的等级。

最后SENN+种预测标签为：![20190419155566089424123.png](http://image.nysdy.com/20190419155566089424123.png)

其中![20190419155566090898589.png](http://image.nysdy.com/20190419155566090898589.png)

## EXPERIMENTS

### Datasets

![20190419155566095994824.png](http://image.nysdy.com/20190419155566095994824.png)

### Entity Prediction

![20190419155566101939979.png](http://image.nysdy.com/20190419155566101939979.png)

![20190419155566104394441.png](http://image.nysdy.com/20190419155566104394441.png)

### Relation Prediction

![20190419155566106385514.png](http://image.nysdy.com/20190419155566106385514.png)

论文还进行了共享嵌入和自适应权重损失机制有效性的验证。

## 参考链接

- <http://blog.openkg.cn/%E8%AE%BA%E6%96%87%E6%B5%85%E5%B0%9D-%E9%9D%A2%E5%90%91%E7%9F%A5%E8%AF%86%E5%9B%BE%E8%B0%B1%E8%A1%A5%E5%85%A8%E7%9A%84%E5%85%B1%E4%BA%AB%E5%B5%8C%E5%85%A5%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/>