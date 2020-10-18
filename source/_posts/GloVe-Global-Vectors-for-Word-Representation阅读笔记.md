---
title: 'GloVe: Global Vectors for Word Representation阅读笔记'
date: 2019-05-20 21:35:25
tags: 
- word vector
- GloVe
english_title: GloVe:Global Vectors for Word Representation
categories:
- 论文阅读笔记



---

> 论文[下载地址](https://www.aclweb.org/anthology/D14-1162)，GloVe是一个新的全球对数双线性回归模型，属于经典的词向量表示方法之一。

<!-- more -->

# Introduction

evaluate the intrinsic quality

- Most word vector methods rely on the distance or angle between pairs of word vectors 
- Mikolov et al. (2013c) introduced word analogies that examines word vector's various dimensions of difference.

two main model families for learning vectors:

- global matrix factorization methods
- local context window methods

Authors propose a specific weighted least squares model that trains on globla word-word co-occurrence counts and thus makes efficient use of statistics.

# Related Work

## Matix Facroization Methods

These methods utilize low-rank approximations to decompose large matrices that capture statistical information about a corpus.

### shortcoming

the most frequent words contribute a dispropoertionate amount to the similarity measure.

## Shallow Window-Based Methods

Another approach is to learn word representations that aid in making predictins within local context windows.

### shortcoming

do not operate directly on the co-occurrence statistics of the corpus and fails to take advantage of the vast amount of repetition in the data.

# The GloVe Model

### GloVe: Global Vectors

the global corpus statistics are captured directly by the model

### the question about the model using the statistics of word occurrences in a corpus

- how meaning is generated from these statistics
- how the resulting word vectors might represent that meaning

## some notation

$X_{ij}$ : the number of times word j occurs in the context of word i

$X_i = \sum_{k} X_{i k}$ : the number of times any word appears in the context of word i

$P_{i j}=P(j | i)=X_{i j} / X_{i}$: the probability that word j appear in the context of word i

![20190515155788329289127.jpg](http://image.nysdy.com/20190515155788329289127.jpg)

above that, werd vector learning should be with ratios of co-occurrence probabilities:

![20190515155788352871603.jpg](http://image.nysdy.com/20190515155788352871603.jpg)

$w \in \mathbb{R}^{d}$are word vectors and $\tilde{w} \in \mathbb{R}^{d}$are separate context word vectors

For F, we should select a unique choice by enforcing a few desiderata.

- encoding the information present the ratio $P_{i k} / P_{j k}$ in the word vector space. 

  Since vector spaces are inherently linear structures

  ![20190515155788379647745.jpg](http://image.nysdy.com/20190515155788379647745.jpg)

- put F to be a compicated function parameterized, and avoiding bofuscating the linear structure![20190515155788397136355.jpg](http://image.nysdy.com/20190515155788397136355.jpg)

- the word-word co-occurrence matrices, we can exchange a word and a context word(because a word can also be a context word)

  1. F should be a homomorphism![2019051515578842869345.jpg](http://image.nysdy.com/2019051515578842869345.jpg)

     by Eqn.(3)![20190515155788435674239.png](http://image.nysdy.com/20190515155788435674239.png)

     F = exp or ![20190515155788448759173.jpg](http://image.nysdy.com/20190515155788448759173.jpg)

  2. the Eqn(6) would have the exchange symmetry if not $\log \left(X_{i}\right)$ and $\log \left(X_{i}\right)$ is independent of k, so it can be absorbed into a bias $b_i$![20190515155788566687599.jpg](http://image.nysdy.com/20190515155788566687599.jpg)

  3. for avoiding diverge, $\log \left(X_{i k}\right) \rightarrow \log \left(1+X_{i k}\right)$

  4. a new weighted least squares regression model to address the problem that LSA wirhts all co-occuttences equally.

     cost function:![20190515155788560186237.jpg](http://image.nysdy.com/20190515155788560186237.jpg)

  5. ![20190515155788571777804.jpg](http://image.nysdy.com/20190515155788571777804.jpg)

## Relationship to Other Models

In this subsection authors show how these models are related to their proposed model.

#### the defect of cross entropy 

- it has the unfortunate property that distributions with long tails are often modeled poorly with too much wieght given to the unlikely events.

## Complexity of the model

the computational complexity of the model depends on the number of nonzero elects in the matrix $X$

#### some assumptions about the distribution of word co-occurrences

- the number of co-occurrences of word $i$ with word $j$, $X_{ij}$, can be modeled as a power-law function of the frequency rank of that word pair, $r_{ij}$:

  $X_{i j}=\frac{k}{\left(r_{i j}\right)^{\alpha}}$

# Experiments

## Evaluation methods

authors conduct experiments on the word analogy taks of Mikolov et al. (2013a)

### Word analogies

The word analogy task consists of questions like, “a is to b as c is to ?”

### Word similarity

![20190520155833277478435.jpg](http://image.nysdy.com/20190520155833277478435.jpg)

### Named entity recognition

## Results

Table 2 shows the CloVe model performs significantly better than the other baslines, often with smaller vector sizes and smaller corpora.

![20190520155833353468570.jpg](http://image.nysdy.com/20190520155833353468570.jpg)

Table 3 shows results on five different word similarity datasets.

Table 4 shows results on the NER task with the CRF-based model.

![20190520155833377540169.jpg](http://image.nysdy.com/20190520155833377540169.jpg)

## Model Analysis: Vector Length and Context Size

![2019052015583339399087.jpg](http://image.nysdy.com/2019052015583339399087.jpg)

### Model Analysis: Corpus Size

![20190520155833403240640.jpg](http://image.nysdy.com/20190520155833403240640.jpg)

- On the syntactic subtask, larger corpora typically produce better statistics so that there is a monotonic increase in performance as the cor- pus size increases.
- But the same trend is not true for the semantic subtask, which is probably because of analogy dataset

## Model Analysis: Run-time

![20190520155833432462881.jpg](http://image.nysdy.com/20190520155833432462881.jpg)

## Model Analysis: Comparison with word2vec

For the same corpus, vocabulary, window size, and training time, GloVe consistently outperforms word2vec

# 参考链接

- https://blog.csdn.net/coderTC/article/details/73864097