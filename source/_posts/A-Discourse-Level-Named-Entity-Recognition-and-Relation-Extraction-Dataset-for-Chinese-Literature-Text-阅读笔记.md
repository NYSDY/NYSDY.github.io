---
title: A Discourse-Level Named Entity Recognition and Relation Extraction Dataset for
  Chinese Literature Text 阅读笔记
date: 2018-12-10 11:07:23
tags:
- NER
- RE
english_title: READ_A_Discourse-Level_Named_Entity_Recognition_and_Relation_Extraction_Dataset_for_Chinese_Literature_Text
categories:
- 论文阅读笔记
---

> 该论文最主要的贡献就是这个数据，[数据集地址](https://github.com/lancopku/Chinese-Literature-NER-RE-Dataset)。论文中提到的标标签过程也是一个创新点，运用了启发式和机器辅助标标签，这样可以提高准确度并减少标注人员工作。

<!-- more -->

## contribution

- provide a new dataset for joint learning of NER and RE for **Chinese literature text**
- the proposed dataset is based on the **discourse level** which provides additional context information
- introduce some widely used models to conduct experiments 

## tagging process

**two methods:**one is a **heuristic tagging** method and another is a **machine auxiliary tagging** method. 

### Step 1: First Tagging Process

find a problem of data inconsistency.

### Step 2: Heuristic Tagging Based on Generic disambiguating Rules

- For example, remove all adjective words and only tag “entity header” .
- **re-annotate** all articles and **correct** all inconsistency entities and relations based on the heuristic rules.

### Step 3: Machine Auxiliary Tagging

- The core idea is to train a model to **learn annotation guidelines** on the subset of the corpus and produce predicted tags on the rest data. 
- CRF

## ![](https://i.loli.net/2018/12/10/5c0dd578c91a0.jpg)tagging set

![](https://i.loli.net/2018/12/10/5c0dd6230f1c3.jpg)

## Annotation Format

### Entity

Each entity is identified by T tag, which takes several attributes.

- Id: a unique number identifying the entity within the document. It starts at 0, and is incremented every time a new entity is identified within the same document.
- Type: one of the entity tags.
- Begin Index: the begin index of an entity. It starts at 0, and is incremented every character.
- End Index: the end index of an entity. It starts at 0, and is incremented every character.
- Value: words being referred to an identifiable object.

### Relation

Each relation is identified by R tag, which can take several attributes:

- Id: a unique number identifying the relation within the document. It starts at 0, and is incremented every time a new relation is identified within the same document.
- Arg1 and Arg2: two entities associated with a relation.
- Type: one of the relation tags.