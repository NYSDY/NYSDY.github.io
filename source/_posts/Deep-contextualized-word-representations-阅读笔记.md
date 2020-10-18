---
title: Deep contextualized word representations 阅读笔记
date: 2019-05-10 15:28:06
tags:
- embedding
english_title: Deep contextualized word representations
categories:
- 论文阅读笔记
---

> [论文下载地址](https://arxiv.org/pdf/1802.05365.pdf)，ELMo事先用语言模型学好一个单词的 Word Embedding，此时多义词无法区分，不过这没关系。在我实际使用 Word Embedding 的时候，单词已经具备了特定的上下文了，这个时候我可以根据上下文单词的语义去调整单词的 Word Embedding 表示，这样经过调整后的 Word Embedding 更能表达在这个上下文中的具体含义，自然也就解决了多义词的问题了。**所以 ELMO 本身是个根据当前上下文对 Word Embedding 动态调整的思路。**

<!-- more -->

# Introduction

## ELMo(Embedddings from Language Models):

### why call ELMo:

Using vectors derived from a bidirectional LSTM that is trained with a coupled language model(LM) objective on a large text corups.

### characteristics

- ELMo representations are a function of all of the internal layers of the biLM.

- learn a linear combination of the vectors stacked above each input word for each end task

- the higher-level LSTM states capture context-dependent aspects of word meaning

  the lower-level states model aspects of syntax

### Extensive experiments

- EMLo representations can be easily added to existing models
- improve the state of art in every case
- ELMo outperform those derived from just the top layer of a LSTM

# Related work

- Some approaches for learning word vectors only allow a single context-independent representation for each word.

- to overcome some shortcomings of traditional word vectors:

  - enriching them with subword information
  - learning separate vectors for each word sense

  Authors uses subword units through the use of character convolutions, seamlessly incorporate multi-sense information into downstream tasks without explicitly training to predict predefined sense classes.

- context-depends representations

   Authors take full advantage of access to plentiful monolingual data

- Previous work also shown that different layers of deep biRNNs encode different types of information

  - introducing multi-task syntactic supervision at the lower levels of a deep LSTM can improve overall performance of higher level tasks
  - the top layer of an LSTM for encoding word context (Melamud et al., 2016) has been shown to learn representations of word sense.

  ELMo representations can also induce similar signals.

# ELMo: Embeddings from Language Models

## Bidirectional language models

- model the probability of token $t_k$ given the history($t_1, … , t_{k-1}$):

  ![20190512155766627486478.png](http://image.nysdy.com/20190512155766627486478.png)

-  a backward LM:![2019051215576663543534.png](http://image.nysdy.com/2019051215576663543534.png)

Authors' formulation jointly maximizes the log likelihood of the forward and backward directions:

![20190512155766643539454.png](http://image.nysdy.com/20190512155766643539454.png)

## ELMo

- For each token $t_k$, a L-layer biLM computes a set of 2L + 1 representations:![20190512155767113638451.png](http://image.nysdy.com/20190512155767113638451.png)

- For a downstream model, ELMo collapses all layers in R into a single vector.

  In the simplest case, ELMo just selects the top layer.

- For a task specific weighting of all biLM layers:![20190512155767132777658.png](http://image.nysdy.com/20190512155767132777658.png)

  $s^{task}$ are softmax-normalized weithts and the scalar parameter $γ^{task}$ allows the task model to scale the entire ELMo vector

## Using biLMs for supervised NLP tasks

- Given a pre-trained biLM and a supervised architecture for a target NLP task
- let the end task model learn a linear combination of these representations
  1. consider the lowest layers of th supervised model without the biLM
  2. add ELMo to the supervised model
     - freeze the weights of the biLM
     - concatenate the ELMo vector $ELMo^{task}_k$ with $x_k$ and pass the ELMo enhanced representation $[x_k,;ELMo^{task}_k ]$ into the task RNN.
     - for some tasks, authors also include ELMo ar the output of task RNN by introducing another set of out put specific linear weights and replacing $h_k$ with $[h_k,;ELMo^{task}_k ]$
     - add a moderate amount of dropout to ELMo and in some case to regularize the ELMo weights

## Pre-trained bidirectional language model architecture

- the biLM provides three layers of representations for each input token, both directions and a residual connection between LSTM layers 
- fine tuning the biLM on domain specific data

# Evaluation

the following picture shows the performance of ELMo in Question answering, Textual entailment, Semantic role labeling, Corefrence resolution, Named entity extraction, Sentiment analysis.

![2019051315577106394943.png](http://image.nysdy.com/2019051315577106394943.png)

In every task considered, simply adding ELMo establishes a new state-of-the-art result.

# Analysis

## Alternate layer weighting schemes

![20190512155767132777658.png](http://image.nysdy.com/20190512155767132777658.png)

the following picture compares these alternatives.

![20190513155771140936480.png](http://image.nysdy.com/20190513155771140936480.png)

Including representations from all layers improves overall performance over just using the last layer, and including contextual representations from the last layer improves performace over the baseline.

Also shows the $\lambda$ is important.

## Where to include ELMo?

The ELMo can be included in both the input and output.

![20190513155771190646517.png](http://image.nysdy.com/20190513155771190646517.png)

the results show including the ELMo in both input and output can preform better.

## What information is captured by the biLM’s representations?

Intuitively, the biLM must be disambiguating the meaning of words using their context.![20190513155771262634271.png](http://image.nysdy.com/20190513155771262634271.png)

The GloVe can only capure the speech. but the biLM is able to disambiguate both the part of speech and word sense in the source sentence.

### Word sense disambiguation

given a sentence, predicting  the sense of a target word using a simple 1-nearst negihbor approach

![20190513155771312514655.png](http://image.nysdy.com/20190513155771312514655.png)

### POS tagging

to examine whether the biLM captures basic syntax.

![20190513155771328944169.png](http://image.nysdy.com/20190513155771328944169.png)

## Sample efficiency

Adding ELMo to a model increases the sample efficiency considerably, both in terms of number of parameter updates to reach state-of-the-art performance and the overall training set size.![20190513155771349825964.png](http://image.nysdy.com/20190513155771349825964.png)

## Visualization of learned weights

![20190513155771355211483.png](http://image.nysdy.com/20190513155771355211483.png)

# 参考链接

- [NAACL2018:高级词向量(ELMo)详解(超详细) 经典](https://zhuanlan.zhihu.com/p/63115885)，这篇文章中阐述了一些使用的细节，并用图来表示，更加清晰。
- [ELMo算法介绍](https://blog.csdn.net/triplemeng/article/details/82380202)，这篇博客中自己对整个论文的概述和总结和好，需要学习。