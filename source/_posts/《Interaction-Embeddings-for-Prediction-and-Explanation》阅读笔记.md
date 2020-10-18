---
title: 《Interaction Embeddings for Prediction and Explanation》阅读笔记
date: 2019-03-21 14:07:37
tags:
- 知识图谱
- 知识图谱嵌入
- 知识图谱推理
english_title: Interaction Embeddings for Prediction and Explanation
categories:
- 论文阅读笔记

---

> [论文下载地址](https://www.zora.uzh.ch/id/eprint/162876/1/interaction-embeddings-prediction_merlin_version.pdf)，此论文主要提出了实体和关系的交互作用对于知识图谱嵌入的影响，和提出了新的嵌入评估方案 - 搜索预测解释。

<!-- more -->

## 论文贡献

- 提出了CrossE，一种通过学习一个交互矩阵来给实体和关系的交互建模的新型知识图谱嵌入。
- 我们使用三个基准数据集评估CrossE与链接预测任务上的各种其他KGE的比较，并显示CrossE在具有适度参数大小的复杂且更具挑战性的数据集上实现最先进的结果。
- 我们提出了一种新的嵌入评估方案 - 搜索预测解释，并表明CrossE能够生成比其他方法更可靠的解释。 这表明交互嵌入更能在不同的三元组环境中捕捉实体和关系之间的相似性。

## 介绍

给定知识图谱和一个要预测的三元组的头实体和关系，在预测尾实体的过程中，头实体和关系之间是有交叉交互的crossover interaction, 即关系决定了在预测的过程中哪些头实体的信息是有用的，而对预测有用的头实体的信息又决定了采用什么逻辑去推理出尾实体，文中通过一个模拟的知识图谱进行了说明如下图所示：

![20190321155314902217434.png](http://image.nysdy.com/20190321155314902217434.png)

## 相关工作

论文中在这部分对KGE（Knowledge graph embedding）进行了分类总结：

- KGEs with general embeddings
- KGEs with multiple embeddings.
- KGEs that utilize extra information.

这部分总结中对大量的方法进行描述，可以作为背景知识进行阅读。

## CrossE模型

基于对头实体和关系之间交叉交互的观察，本文提出了一个新的知识图谱表示学习模型CrossE. CrossE除了学习实体和关系的向量表示，同时还学习了一个交互矩阵C，C与关系相关，并且用于生成实体和关系经过交互之后的向量表示，所以在CrossE中实体和关系不仅仅有通用向量表示，同时还有很多交互向量表示。CrossE核心想法如下图：![20190321155314970714298.png](http://image.nysdy.com/20190321155314970714298.png)

目标函数粉四步生成：

1. Interaction Embedding for Entities：根据头实体向量和交互矩阵（以关系确定的）来确定头实体的交互表示。
2. Interaction Embedding for Relations：根据头实体的交互表示和关系作用生成关系的交互表示
3. Combination Operator：将头实体的交互表示和关系的交互表示相结合，并进行非线性处理（tanh）
4. Similarity Operator：计算结合后表示和尾实体表示之间的相似度。

最后分数函数：

![20190321155315073712112.png](http://image.nysdy.com/20190321155315073712112.png)

损失函数：（这里就是一个交叉熵函数，但是写的有问题f(x)项应该在括号外）![20190321155315081421205.png](http://image.nysdy.com/20190321155315081421205.png)

## 对于预测的解释

这部分作者描述了如何生成预测三元组的解释，并介绍了基于嵌入的路径搜索算法，主要步骤如下：

1. Search for similar relations：修剪掉不合理路径
2. Search for paths between h and t：作者定义了6种路径（班汉一个或两个关系）
3. Search similar entities：捕获实体之间的相似性方面越有能力，就越有可能存在（hs，r，ts）
4. : Search for similar structures as supports：我们只将知识图中至少有一个支持的路径视为解释。

## 实验

### 数据集

![20190321155315142112584.png](http://image.nysdy.com/20190321155315142112584.png)

### 链接预测

![image-20190321145750608](/Users/dy/Library/Application Support/typora-user-images/image-20190321145750608.png)

![20190321155315147862891.png](http://image.nysdy.com/20190321155315147862891.png)

从实验结果中我们可以看出，CrossE实现了较好的链接预测结果。我们去除CrossE中的头实体和关系的交叉交互，构造了模型 CrossES，CrossE 和 CrossES 的比较说明了交叉交互的有效性。

### 生成解释

我们提出了一种基于相似结构通过知识图谱的表示学习结果生成预测结果解释的方法，并提出了两种衡量解释结果的指标，AvgSupport和Recall。Recall是指模型能给出解释的预测结果的占比，其介于0和1之间且值越大越好；AvgSupport是模型能给出解释的预测结果的平均support个数，AvgSupport是一个大于0的数且越大越好。可解释的评估结果如下：

![2019032115531515625385.png](http://image.nysdy.com/2019032115531515625385.png)

链接预测和可解释的实验从两个不同的方面评估了知识图谱表示学习的效果，同时也说明了链接预测的准确性和可解释性没有必然联系，链接预测效果好的模型并不一定能够更好地提供解释，反之亦然。

## 参考链接

- http://blog.openkg.cn/%E8%AE%BA%E6%96%87%E6%B5%85%E5%B0%9D-interaction-embeddings-for-prediction-and-explanation/

## 