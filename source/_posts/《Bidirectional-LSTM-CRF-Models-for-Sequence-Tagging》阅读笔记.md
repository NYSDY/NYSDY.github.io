---
title: 《Bidirectional LSTM-CRF Models for Sequence Tagging》阅读笔记
date: 2018-10-05 13:05:12
tags:
- LSTM
- NER
- BI-LSTM
english_title: read_Bidirectional_LSTM-CRF_Models_for_Sequence_Tagging
categories:
- 论文阅读笔记
---

> 这篇论文可以作为一个RNN和LSTM学习的一个例子来看，有利于新手对LSTM的理解。对于NER的处理主要是作为一个序列标注问题。但是作为经典文章还是可以读一读了解一下的。

<!-- more -->

在本篇论文中，作者提出了4种模型：LSTM、BI-LSTM、LSTM-CRF和BI-LSTM-CRF。

## contribution(贡献)

1. 作者在NLP标注数据集上系统的对比了以上四个模型；
2. 作者是首先提出把BI-LSTM-CRF模型用于NLP序列标注，并且达到了state-of-the-art的水平；
3. 作者展示了BI-LSTM-CRF是robust，并且极少依赖于词向量。

## model(模型)

### LSTM

首先，作者先介绍了RNN的结构和工作原理，如图：![](https://i.loli.net/2018/10/04/5bb5a3e22e140.jpg)

其中输入为句子：EU rejects German call to boycott British lamb。输出为标签：B-ORG O B-MISC O O O B-MISC O O，其中B-，I-表示实体开始和中间位置。标签种类为：other (O)和四种实体标签：Person (PER), Location (LOC), Organization (ORG), and Miscellaneous (MISC).

输入层表示在时间步 t 的特征。它们可以是 one-hot-encoding 的词特征，稠密或者稀疏的向量特征。输入层与特征有相同大小的维度。输出层表示在时间步 t 的标签上的概率分布，维度与标注数量相同。相比前馈神经网络，RNN 引入前一个隐藏状态和当前隐藏状态的结合，因此可以储存历史信息。

涉及公式为：![](https://i.loli.net/2018/10/04/5bb5a556ad0ab.jpg)

![](https://i.loli.net/2018/10/04/5bb5a57379fd7.jpg)

其中，U，W，V都是权重，函数f，g分别为sigmoid和softmax函数。  

接下来，作者展示了LSTM的结构和原理，如图：![](https://i.loli.net/2018/10/04/5bb5a61bdb483.jpg)

公式：![](https://i.loli.net/2018/10/04/5bb5a670e3b88.jpg)

其中，σ是逻辑sigmoid函数，i, f, o 和 c分别是输入门，忘记门，输出门和细胞向量，所有的大小都和向量h一样。w权重的含义如其下表所示。

LSTM序列标注模型如图所示：![](https://i.loli.net/2018/10/04/5bb5a84cd95d5.jpg)

其中，中间的画斜线的格子即为图2中所示部分。

### Bidirectional LSTM(双向LSTM)

作者展示了双向LSTM的结构，如图所示：![](https://i.loli.net/2018/10/04/5bb5a8f44f16e.jpg)

双向LSTM网络可以有效利用过去特征和未来特征。在作者的实现中，对于整个句子的前向和后向操作，作者只需要在每个句子开始时将隐藏状态重置为0。作者采用批处理，使得可以同时处理多个句子。

### CRF

使用邻居标记信息预测当前标记有两种不同的方法：

1. 预测每个时间步长的标签分布，然后使用波束式解码来找到最优的标签序列，代表方法：MEMMs
2. 注重句子层次而不是个体位置，代表方法：CRF，输入和输出是直接相连的；如图：![](https://i.loli.net/2018/10/04/5bb5ac191d2dd.jpg)

研究表明，CRFs一般能够产生更高的标签精度。

### LSTM-CRF

作者展示了LSTM-CRF的结构，如图：![](https://i.loli.net/2018/10/04/5bb5ad05ebec0.jpg)

这网络可以有效地通过 LSTM 利用过去的输入特征和通过 CRF 利用句子级的标注信息。图中CRF层由连接连续输出层的线表示。CRF层有一个状态转移矩阵作为参数。

公式为：![](https://i.loli.net/2018/10/04/5bb5b6ea7f87e.jpg)

函数f为网络的输出分数，[x]为输入， [fθ]i,t 为带有参数θ（句子x，第i 个标签，第t个单词）的网络输出；

[A]i,j为转移分数，从连续的时间步i状态到j状态的转移分数。注意，该转换矩阵是位置无关的。

### BI-LSTM-CRF

作者展示了BI-LSTM-CRF的结构，如图所示：![](https://i.loli.net/2018/10/04/5bb5b87ca950f.jpg)

作者在实验中展示了额外的未来特征可以提高标签的准确率。

## 训练过程

本文使用的所有模型都共享一个通用SGD前向和后向训练过程。作者展示了BI-LSTM-CRF的算法，如图![](https://i.loli.net/2018/10/04/5bb5ba7ce18c7.jpg)

作者设置了批次大小为100。

## 实验

### data

作者在以下三个数据集上测试自己的模型：Penn TreeBank (PTB) POS tagging, CoNLL 2000 chunking, and CoNLL 2003 named entity tagging.数据集信息展示如下：![](https://i.loli.net/2018/10/04/5bb5bc263ec9f.jpg)

### Features

作者从三个数据集中提取出其公共特征。特征可以分为拼写特征和上下文特征。最终，作者对于POS（词性标注）、chunking（组块）和NER（命名实体识别）分别提取401K，76K和341K个特征。

### spelling features（拼写特征）

除了小写字母特征之外，我们提取给定单词的以下特征。

![](https://i.loli.net/2018/10/04/5bb5bdde879c9.jpg)

### context featurs（上下文特征）

对于单词特征，作者使用unigram和bi-grams特征。对于在CoNLL2000数据集中的POS特征和在CoNLL2003数据集中的 POS & CHUNK特征，作者使用了unigram，bi-gram和tri-gram特征。

### 词向量

词向量在改进序列标注任务的表现方面起着至关重要的作用，我们使用 130K 词汇并且每个词汇的词向量维度是 50 维。我们将 one-hot-encoding词表示替换每个词对应的词向量。

### Features connection tricks

我们可以将拼写和上下文特征与单词特征一样对待。这样网络的输入包括单词，单词的拼写和上下文特征。然而，==我们发现将拼写和上下文特征与输出直接连接可以加速训练过程，同时也能保持标注的准确率，==如下图所示：![](https://i.loli.net/2018/10/04/5bb5bfc63f961.jpg)

我们注意到，这种特征的使用与使用的最大熵特征类似。区别在于采用特征三列技术可能会发生特征冲突。由于序列标注数据集中的输出标签小于语言模型（通常为数十万），所以我们可以在特征和输出之间建立完整的连接，以避免潜在的特征冲突。

### 结果

在相同的数据集上分别训练LSTM，BI-LSTM，CRF，LSTM-CRF和BI-LSTM-CRF模型，并且采用两种方式初始化word embedding：随机和Senna方式。模型的训练速率为0.1，隐藏层数量为300.不同模型在不同word embedding下的结果如表2所示，同时列出了之前最好模型Cov-CRF。

![](https://i.loli.net/2018/10/04/5bb5c1da95e97.jpg)

- 与Cov-CRF比较

  实验中设置了3个基准模型，分别为LSTM、BI-LSTM和CRF，结果中LSTM在三个数据集中效果最差，BI-LSTM跟CRF在POS和chunking中效果接近，但是在NER中后者要优于前者。有趣的是表现最好的模型BI-LSTM-CRF相对于Cov-CRF来说对Senna embedding的依赖程度更小。

- (robustness)模型鲁棒性  

  为验证模型的鲁棒性，对不同模型只采用word feature特征进行训练，训练结果如表3，括号中数字表示相比于全部特征，模型的结果下降数值。![](https://i.loli.net/2018/10/05/5bb6ee3958635.jpg)

- 与其他系统的比较

  这里就不贴图了，总之就是阐述作者自己模型好。

## 结论

总之作者的模型是基于之前模型的一些改进，主要运用了IBI-LSTM和CRF的结合。

### [论文下载地址](https://arxiv.org/pdf/1508.01991.pdf)