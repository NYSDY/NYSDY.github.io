---
title: 《RECURRENT NEURAL NETWORK REGULARIZATION》阅读笔记
date: 2019-01-07 08:56:13
tags:
- RNN
- LSTM
english_title: RECURRENT NEURAL NETWORK REGULARIZATION
categories:
- 神经网络
---

> [论文链接](http://arxiv.org/abs/1409.2329)。这篇论文提出了LSTM的dropout策略来防止过拟合，即只在非循环链接处采取dropout。在BasicLSTMCell的接口就是依据这篇论文实现的。

## 文章整体架构和重点

![](https://i.loli.net/2019/01/07/5c32ad34c99e0.jpg)

## contribution

- present a simple regularization technique

## background

- dropout Srivastava(2013)，对于前向反馈网络最有力的正则化方法并不能很好的应用在RNNs上。这导致RNNs规模都很小，因为太大会过拟合。

- Bayer et al. (2013)指出了卷积dropout不能在RNNs上很好工作的原因是循环会放大噪音。

## model

### 仿射变换（affine transform）

关于仿射变换：线性变换加上平移，盗个知乎上的图（[原文链接](https://www.zhihu.com/question/20666664)）

![](https://i.loli.net/2019/01/07/5c32a7c05c698.jpg)

### 模型主体

采用的是Graves et al. (2013)![](https://i.loli.net/2019/01/07/5c32aaae11c93.jpg)

### dropout策略

> The main contribution of this paper is a recipe for applying dropout to LSTMs in a way that success- fully reduces overfitting. The main idea is to apply the dropout operator only to the non-recurrent connections.

![](https://i.loli.net/2019/01/07/5c32aaf4084fc.jpg)

观察公式，实际上就是通过在层间传递中应用dropout。如下图中虚线所示。

![](https://i.loli.net/2019/01/07/5c32a99676abf.jpg)

从上图中也可以看到，该dropout的次数只和网络深度有关（数值为网络深度+1）。

### experiments

实验部分作者做了4部分实验来证明自己采用的方法有效，分别为language modeling, speech recognition, machine translation, image caption generation。这部分没什么需要解释了，感兴趣可以自己看一下实验。