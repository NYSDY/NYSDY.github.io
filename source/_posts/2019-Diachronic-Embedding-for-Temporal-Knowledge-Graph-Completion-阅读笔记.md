---
title: Diachronic Embedding for Temporal Knowledge Graph Completion
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-07-02 19:09:49
password:
english_title: Diachronic Embedding for Temporal Knowledge Graph Completion
tags:
- KG
- temporal KG
- KGE
- 2019
categories:
- 论文阅读笔记
---

> [论文下载地址](https://arxiv.org/pdf/1907.03143.pdf)

# Introduction

## what your paper is about?

temporal knowledge graph completion

## what problem it solves?

提出了一个Diachronic Embedding 能和已有的static模型结合来解决temporal knowledge graph completion，并且能生成一个unseen timestamps。

## why the problem is interesting?

Developing temporal KG embedding models is an increasingly important problem.

## what is really new (and what isn't)?

1. novel models for temporal KG completion through equipping static models with a diachronic entity embedding function which provides the characteristics of entities at any point in time. This is in contrast to the existing temporal KG embedding approaches where only static entity features are provided.
2. The proposed embedding function is model-agnostic and can be potentially combined with any static model.
3. combining it with SimplE results in a fully expressive model for temporal KG completion.（作者还提供了证明，不过我没看）

## Diachronic Embedding

![](http://image.nysdy.com/20200702191834.png)

![](http://image.nysdy.com/20200702191909.png)

γ用来控制temporal的比例。

# experiments

![](http://image.nysdy.com/20200702192000.png)

- Activation Function
- Adding Diachronic Embedding for Relations:
  - 实验展示了关系随时间变化影响小，即使加入关系和时间的联系对结果提升也不大
- Generalizing to Unseen Timestamps
  - we created a variant of the ICEWS14 dataset by including every fact except those on the 5th, 15th, and 25th day of each month in the train set. We split the excluded facts randomly into validation and test sets (removing the ones including entities not observed in the train set). This ensures that none of the timestamps in the validation or test sets has been observed by the model in the train set.