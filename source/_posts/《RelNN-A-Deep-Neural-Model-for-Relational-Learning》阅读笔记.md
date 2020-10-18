---
title: 《RelNN A Deep Neural Model for Relational Learning》阅读笔记
date: 2019-03-25 09:34:57
tags:
- 关系抽取
- 统计学习
english_title: RelNN A Deep Neural Model for Relational Learning
categories:
- 论文阅读笔记
---



> [论文下载地址](https://arxiv.org/pdf/1712.02831.pdf)，这篇文章相当于结合了统计学习和深度神经网络。里面有些公式没有理解，应该是有许多先前论文需要阅读。但是本篇论文扩展了思路如何结合统计学和深度学习，并且基于其余数据来预测一个类中对象的一个属性，想法也比较好。

<!-- more -->

## Introduction

作者主要集中于基于其余数据来预测一个类中对象的一个属性。

## Challenge

当类中每个对象的属性依赖于不同数量的其他对象的属性和关系时，此问题具有挑战性。 在StarAI社区中，此问题称为聚合（aggregation）。

## Relational Logistic Regression and Markov Logic Networks

StarAI模型旨在模拟对象之间关系的概率。 

### Relational logistic regression (RLR) (Kazemi et al. 2014)

定义的概率公式如下：

![20190327155364804947436.png](http://image.nysdy.com/20190327155364804947436.png)

上面定义的RLR模型仅适用于布尔值或多值父项。作者采用的是连续的原子（continuous atoms）（Fatemi, Kazemi, and Poole (2016)）

### Relational Neural Networks

作者通过设计神经网络中线性层（LL），激活层（AL）和误差层（EL）的关系对应物，对具有分层架构的RLR / MLN模型进行编码。

关系神经网络（RelNN）是包含作为图形彼此连接的若干RLL和RAL的结构。

![20190327155364863172988.png](http://image.nysdy.com/20190327155364863172988.png)

## Motivations for hidden layers

- 使喜欢看动作电影的人数增加时，男性的概率变为[0, 1]重的任何数值，不至于直接变为0或者1。
- 因此，隐藏层通过使模型能够学习通用规则并相应地对对象进行分类，然后以不同方式处理不同类别的对象，从而提高了建模能力。
- 使用RLR表示不同类型的现有显式聚合器，然而有些情况需要使用2个RLLs和2个RALs

## Learning latent properties directly

对象可能包含无法使用常规规则指定的潜在属性，但可以在训练期间直接从数据中学习。

![20190327155364936417890.png](http://image.nysdy.com/20190327155364936417890.png)

考虑图2中的模型，让Latent（m）成为电影的数字潜在属性，其值将在训练期间学习。

## From ConvNet Primitives to RelNNs

我们解释为什么RelNN也可以被视为ConvNets的一个实例。

ConvNets的输入矩阵中的单元（例如，图像像素）具有空间相关性和空间冗余：彼此更接近的单元比更远的单元更依赖。 例如，如果M表示图像的输入通道，则M [i，j]和M [i + 1，j + 1]之间的依赖性可能远大于M [i，j]和M [i，j+20]之间的依赖性。

对于关系数据，输入矩阵中的依赖关系（关系）是不同的：同一行或列中的单元（即同一对象的关系）具有比不同行和列中的单元更高的依赖性（即不同对象的关系）。 因此，为了使ConvNets适应关系数据，我们需要矢量形状的过滤器，这些过滤器对行和列交换是不变的，并且更好地捕获关系依赖性和可交换性假设。



![2019032715536511162060.png](http://image.nysdy.com/2019032715536511162060.png)

## Datasets

### Movielens 1M dataset (Harper and Konstan 2015)

第一个数据集是Movielens 1M dataset (Harper and Konstan 2015)，忽略了实际的评级，只考虑电影是否被评级，只考虑动作和戏剧类型。

### PAKDD15

[获取地址](https://knowledgepit.fedcsis.org/contest/view.php?id=107)

### all Chinese and Mexican restaurants in Yelp dataset challenge

[获取地址](https://www.yelp.com/dataset_challenge)

## Empirical Results

作者提出了三个问题来进行实验：

### Q1：RelNN的性能与其他众所周知的关系学习算法相比如何？

![20190327155365126319780.png](http://image.nysdy.com/20190327155365126319780.png)

### Q2：基于数字和规则的潜在属性如何影响RelNN的性能?

更改了RelNN中隐藏图层和数字潜在属性的数量，以查看它们如何影响性能。

![20190327155365168099042.png](http://image.nysdy.com/20190327155365168099042.png)

请注意，添加图层只会添加一定数量的参数，但添加k个数字潜在属性会增加k * |Δm|参数。

### Q3：RelNN如何推断出看不见的案例并解决指向的规模大小问题 （Poole et al.2014）?

作者实施了两个实验：

- 我们在大量数据中训练一个RelNN，并在一小数据上进行测试：该实验可以看作每个模型受冷启动问题的严重程度
- 然后我们在一小数据上训练一个RelNN并在一大数据上进行测试：可以看作这些模型对更大群体的推断

![20190327155365190971381.png](http://image.nysdy.com/20190327155365190971381.png)

## Future

作者将从数据中自动学习这些结构的问题留作未来的工作。