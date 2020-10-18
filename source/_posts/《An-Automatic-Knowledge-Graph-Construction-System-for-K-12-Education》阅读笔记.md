---
title: 《An Automatic Knowledge Graph Construction System for K-12 Education》阅读笔记
date: 2019-01-19 10:32:57
tags:
- 知识图谱
- NER
english_title: An Automatic Knowledge Graph Construction System for K-12 Education
categories:
- 论文阅读笔记
---

> [论文原址](http://delivery.acm.org/10.1145/3240000/3231698/a40-chen.pdf?ip=59.64.129.211&id=3231698&acc=NO%20RULES&key=BF85BBA5741FDC6E%2E66A15327C2E204FC%2E4D4702B0C3E38B35%2E4D4702B0C3E38B35&__acm__=1547779677_7b4dce90b008fa6300a47916eeb139d9)。本篇文章主要提出了一个自动化构建数学领域知识图谱的系统，主要应用的事NER和数据挖掘技术，其中NER主要是抽取数学概念，概念间的关系是作者自己构建的（例如先修关系）。对于数据集，作者主要从the Chinese curriculum standards of mathematics上提取的概念实体，从自己的SLP平台上，通过对学生表现来提取关系（把这部分作为数据挖掘）。**本篇文章实际上可以作为构建特定领域的知识图谱的一个参考。**

<!-- more -->

### challenges

- the desired educational concept entities are more abstract than real world entities like PERSON, ORGANIZATION, LOCATION
- the desired relations are more cognitive and implicit, so cannot be derived from the literal meanings of text like generic knowledge graphs

### contributions

- a novel but practical system
- entity recognition (NER) & association rule mining algorithms
- demonstrate an exemplary case with constructing a knowledge graph for the subject of mathematics

## SYSTEM OVERVIEW

![](https://i.loli.net/2019/01/19/5c428c37364b4.jpg)

- Educational Concept Extraction Module:
- Implicit Relation Identification Module

## CONCEPT EXTRACTION

- 线性链式CRF模型

  ![](https://i.loli.net/2019/01/18/5c4141ca00be1.jpg)

- 标签预测

  ![](https://i.loli.net/2019/01/18/5c4141ed6ac27.jpg)

## RELATION IDENTIFICATION

### 两种方法

- support
- confidence

> From the perspective of prerequisite relation, if concept si is a prerequisite of concept sj, learners who do not master sivery likely do not master sj, and learners who master sjmost likely master si. 
>
> ![](https://i.loli.net/2019/01/18/5c41425aafc99.jpg)



## EXEMPLARY CASE AND SYSTEM EVALUATION

### Concept Extraction

#### Dataset

the Chinese curriculum standards of mathematics published by the ministry of education as the main data source

#### Evaluation

- adopt precision, recall and F1- score 
- The ground truth is manually labeled by two domain experts.

### Relation Identification

#### Dataset

 students’ performance data collected by our SLP platform.

#### Evaluation

The ground truth of the prerequisite relations between selected 9 concepts are annotated manually by two domain experts.

