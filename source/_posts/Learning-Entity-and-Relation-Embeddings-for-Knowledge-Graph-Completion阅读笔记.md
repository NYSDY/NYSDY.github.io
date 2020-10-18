---
title: Learning Entity and Relation Embeddings for Knowledge Graph Completion阅读笔记
date: 2019-06-26 16:18:15
tags:
- KGE
- KGR
- TransR
english_title: Learning_Entity_and_Relation_Embeddings_for_Knowledge_Graph_Completion
categories:
- 论文阅读笔记
---

> TransR embeds entities and relations in distinct entity space and relation space, and learns embeddings via translation between projected entities.CTransR models internal complicated correlations within each relation type.
>
> 论文下载地址

<!-- more -->

# Problem Statement

In fact, an entity may have multiple aspects and various relaitons may focus on different aspects of entities, which makes a common space insurficient for modeling.

# Contribution

- propose a TransR model which models entities and relations in distinct spaces
- CTransR models internal complicated correlations within each relation type.
- experiment on benchmark datasets of WordNet and Freebase and gain consistent improvements compared to state-of-the-art models

# Future work

- Existing models including TransR consider each relational fact separately.
  - relation transitive
- explore a unified embedding model of both text side and knowledge graph
- modeling internal correlations within each relation type

# TransR

![20190627156159912894954.jpg](http://image.nysdy.com/20190627156159912894954.jpg)

1. for each triple$(h, r, t)$, entities embeddings are set as $\mathbf{h}, \mathbf{t} \in \mathbb{R}^{k}$ and relation embedding is set as $\mathbf{r} \in \mathbb{R}^{d}$, $k \neq d$

2. for each relation $r$, set a projection matrix $\mathbf{M}_{r} \in\mathbb{R}^{k \times d}$

   - projects entities from entity space to relation space

3. projected vectors of entities as 
   $$
   \mathbf{h}_{r}=\mathbf{h} \mathbf{M}_{r}, \quad \mathbf{t}_{r}=\mathbf{t} \mathbf{M}_{r}
   $$

4. score function:
   $$
   f_{r}(h, t)=\left\|\mathbf{h}_{r}+\mathbf{r}-\mathbf{t}_{r}\right\|_{2}^{2}
   $$

# Cluster-based TransR (CTransR)

### why propose CTransR

TransE, TransH and TransR, learn a unique vector for each relation, which may be under-representative to fit all entity pairs under this relation, because these relations are usually rather diverse.

### basic idea

- incorporate the idea of piecewise linear regression Ritzema and others 1994
- segment input instances into several groups

### process

1. for a specific relation r, all entity pairs (h, t) in the training data are clustered into multiple groups, and entity pairs in each group are expected to exhibit similar r relation.

   - All entity pairs (h, t) are represented with their vector offsets (h − t) for clustering, where h and t are obtained with TransE.

2. learn a separate relation vector $r_c$for each cluster and matrix $M_r$ for each relation, respectively

3. projected vectors of entities as $\mathbf{h}_{r, c}=\mathbf{h} \mathbf{M}_{r} \text { and } \mathbf{t}_{r, c}=\mathbf{t} \mathbf{M}_{r}$

4. sorce fuction
   $$
   f_{r}(h, t)=\left\|\mathbf{h}_{r, c}+\mathbf{r}_{c}-\mathbf{t}_{r, c}\right\|_{2}^{2}+\alpha\left\|\mathbf{r}_{c}-\mathbf{r}\right\|_{2}^{2}
   $$
   the later item aims to ensure cluster-specific relation vector rcnot too far away from the original relation vector r

# dataset

采用和前人所用一样的数据集

| Dataset | #Rel  | #Ent   | #Train  | #Valid | # Test |
| ------- | ----- | ------ | ------- | ------ | ------ |
| WN18    | 18    | 40,943 | 141,442 | 5,000  | 5,000  |
| FB15K   | 1,345 | 14,951 | 483,142 | 50,000 | 59,071 |
| WN11    | 11    | 38,696 | 112,581 | 2,609  | 10,544 |
| FB13    | 13    | 75,043 | 316,232 | 5,908  | 23,733 |
| FB40K   | 1,336 | 39528  | 370,648 | 67,946 | 96,678 |

# Experiment

作者采取常规实验

### Link Prediction

这里作者对关系中聚类进行了展示： ![2019062715616031534723.jpg](http://image.nysdy.com/2019062715616031534723.jpg)

> 我觉得这种方式是值得尝试的。

### Triple classification

Moreover, the “bern” sampling technique improves the performance of TransE, TransH and TransR on all three data sets.

> bern采样方法需要掌握。

### Relation Extraction from Text