---
title: 《Learning Phrase Representations using RNN Encoder–Decoder for Statistical Machine Translation》阅读笔记
date: 2019-01-05 10:23:27
tags:
- RNN
- seq2seq
english_title: Learning_Phrase_Representations_using_RNN_Encoder–Decoder_for_Statistical_Machine_Translation
categories:
- 论文阅读笔记
---

> [原文链接](https://www.aclweb.org/anthology/D14-1179)。该论文是Sequence to Sequence学习的最早原型，论文中提出一种崭新的RNN(GRU) Encoder-Decoder算法，虽然文章属于比较旧的文章，但作为seq2seq的基础原型，还是需要阅读了解一下的。文章写的比较详细，各部分细节都有讲解。

<!-- more -->

## 文章的主要结构

![](https://i.loli.net/2019/01/05/5c3019fd55653.jpg)

## contribution

- a novel RNN Encoder–Decoder

  能够处理变长序列

- a novel hidden unit

  - reset gate
  - update gate

##  RNN Encoder–Decoder

模型结构图如下：

![](https://i.loli.net/2019/01/05/5c301b9717fd8.jpg)

文中作者对齐进行总体概述为：

> From a probabilistic perspective, this new model is a general method to learn the conditional distribution over a variable-length sequence conditioned on yet another variable-length sequence

从概率的角度来看，这个新模型是学习在另一个可变长度序列条件下的可变长度序列上的条件分布的一般方法

### Encoder

这部分是一个RNN单元。每个时间步，我们向Encoder中输入一个字/词（一般为向量形式），直到我们输入这个句子的最后一个字/词$X_T$，然后输入整个句子的语义向量c。由于RNN的特带你就是把前面每一步的输入信息都考虑进来，所以理论上这个c就包含了整个句子的所有信息。我们可以把当成这个句子的一个语义表示。

### Decoder

Decoder是另一个RNN，其被训练出来以通过预测隐藏状态$h_t$的下一个符号$y_t$来生成输出序列。计算公式如下

$$h_t = f(h_{t-1},y_{t-1},c)$$

下一个序列的计算公式如下：

$$P(y_t|y_{t-1},y_{t-2},\dots,y_1,c)=g(h_t,y_{t-1},c)$$

## Hidden Unit

该部分是对各部分具体的公式讲解，实际是GRU的具体公式算法，不再此详细叙述了。

### reset gate

> In this formulation, when the reset gate is close to 0, the hidden state is forced to ignore the pre- vious hidden state and reset with the current input only. This effectively allows the hidden state to drop any information that is found to be irrelevant later in the future, thus, allowing a more compact representation.

这段原文主要讲解了复位门的作用：有效地允许隐藏状态丢弃在将来稍后发现不相关的任何信息，从而允许更紧凑的表示。

**当捕获短期依赖时，复位门活跃**

### update gate

> the update gate controls how much information from the previous hidden state will carry over to the current hidden state.

更新门控制来自先前隐藏状态的多少信息将转移到当前隐藏状态。

**当捕获长期依赖时，更新门活跃**

## Statistical Machine Translation(SMT)

![](https://i.loli.net/2019/01/05/5c30217d42b8e.jpg)

## Experiments

这部分作者主要做了量化分析和性质分析，主要就是说他的模型怎么厉害。。。（没有具体的数值指标，翻译的还不是中英翻译，想看的话可以去看一下，就不贴实验结果了）。

## future

这里作者提出了可以用decoder生成的目标短语来替换原句中短语的思路，如果没记错的话，这个想法好像对后面的机器翻译有很大的指导作用。