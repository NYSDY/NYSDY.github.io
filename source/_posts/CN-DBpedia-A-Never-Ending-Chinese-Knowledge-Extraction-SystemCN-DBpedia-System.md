---
title: CN-DBpedia A Never-Ending Chinese Knowledge Extraction SystemCN-DBpedia System
date: 2018-10-01 22:27:25
tags: [知识图谱, 机器学习, RNN]
categories: 论文阅读笔记
---

# 前言

> 本篇论文为2016年的一篇论文，主要介绍了作者构建中文知识图谱所遇到的一些问题和解决方法。

<!-- more -->

### challenge

1. 如何降低人力成本？
2. 如何保持知识库的新鲜度？

### 贡献

1. 在构建中文知识库中降低了人力成本：
   - 重复利用已经存在的本体论
   - 提出了一个不用人工监督的端到端的深度学习模型
2. 提出了一个智能主动更新策略

### 系统结构

![](https://i.loli.net/2018/09/26/5baaf081b2644.jpg)

提高知识库质量：

1. **Normalization**： normalize the attributes and values
2. **Enrichment**：reuse the ontology
3. **Correction**：two steps
   1. error detection:
      - rule-based detection
      - based on user feedbacks
   2. error correction
      - crowd-sourcing

### 降低人力成本

这部分作者采用了两种方法：

1. 重复利用已经存在在知识库的本体论和类型化的中文实体
2. 构建一个端到端提取器

#### Cross-Lingual Entity Typing（跨语言的实体类型）

- 第一步是通过用英文DBpedia类型来类型化中文实体。为了达到这个目的，作者提出了如下系统：![](https://i.loli.net/2018/10/01/5bb2273b35ab0.jpg)系统建立了监督层次分类模型，系统输入为没有标记类型的中文实体，输出为在DB中所有有效的英文类型。作者将中文实体与共享相同中文标签名称的英语实体配对，这样中文实体以及配对英语实体的类型自然是标记样本。
- 用上述方法得到的训练集可能出现下面一些问题：
  - 英文DBpedia实体类型在许多情况下可能不完全；
  - 英文DBpedia实体类型在许多情况下可能是错误的；
  - 中英文链接可能出错；
  - 中文实体的特征通常不完整。
- 为了解决以上问题，作者提出了两种方法：
  - 完善英文DBpedia实体类型；
  - 设计一个过滤步骤来剔除错误样本。

#### infobox completion

> Infobox completion is a task to extract object for a given pair of entity and predicate from encyclopedia articles.

作者建模了一个seq2seq模型，输入为包含tokens的自然语言句子，输出为每个token的标签。对于标签为0或1。

对于建立一个有效的提取器有以下两个关键：

1. 如何构建训练集：作者采用远程监督方法（利用Wikipedia）
2. 如何选取期望的提取模型：LSTM-RNN，如图所示![](https://i.loli.net/2018/10/01/5bb22b72af582.jpg)

### 知识库更新

作者采用动态更新：识别新实体或可能包含新事实的旧实体

作者根据以下两方面来辨别这些实体：

- 近期热点新闻中提及的实体
- 在搜索引擎的流行搜索关键字或其他流行网页中提到的实体

对于如何从新闻标题和搜素指令中提取实体名字，作者采用简单的词分割方法，从百科全书中判断其是否为实体，并提出IDF值低的分割子串。

### 统计数据

![](https://i.loli.net/2018/10/01/5bb22d57ab3ec.jpg)

### [论文下载链接](https://www.researchgate.net/publication/318144300_CN-DBpedia_A_Never-Ending_Chinese_Knowledge_Extraction_System)

