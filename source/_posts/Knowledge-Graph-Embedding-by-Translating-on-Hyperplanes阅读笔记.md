---
title: Knowledge Graph Embedding by Translating on Hyperplanes阅读笔记
date: 2019-05-28 16:17:44
tags:
- KGE
- transH

english_title: Knowledge Graph Embedding by Translating on Hyperplanes
categories:
- 论文阅读笔记
- KG	
---

> 作为trans系列经典文献，必读。文章主要精华在于这种超平面想法的由来解决了同一实体的多关系问题。
>
> Authors proposed TransH which models a relation as a hyperplane together with a translation operation on it. It solves the problem of multi-relation and makes a good trade-off between model capacity and efficiency.

<!-- more -->

# 推测transH的想法来源

> 既然实际是表达同一关系不同实体最后通过TransE后会趋于一致，那么我直接通过一个中介来进行映射将同一表示映射成不同向量表示，那么这些向量表示就可以代表不同的实体，就达到了不同实体拥有不同表示的目的。因为关系是不变的所以想到了将关系作为映射平面，让实体向量向其中映射。

# research objective

- solves the problem of multi-relation 
- makes a good trade-off between model capacity and efficiency

# Problem Statement

-  TransE can't deal with reflexive, one-to-many, many-to-many and many -to-one relations
- some complex model sacrifice efficiency in the process(although can deal with transE's problem)

# Contribution

- proposing a method named *translation on hyperplanes*(TransH)
  - interpreting a relation as a translating operation on a hyperplane
- proposing a simple trick to reduce the chance of false negative labeling

# Embedding by Translating on Hyperplanes

## Relations’ Mapping Properties in Embedding

transE

- the representation of an  entity is the same when involved in any relations, ignoring **distributed representations of entities when invovled in different relaions**

## Translating on Hyperplanes (TransH)

**同一个实体在不同关系中的意义不同**，同时**不同实体，在同一关系中的意义，也可以相同**。

> 将每个关系定义在一个独特的平面呢，在该平面内有符合该关系的transE的表示（h,r,t)，多加入的代表该平面的法向量完成了将不同实体向平面内和h，t转化的任务，使得同一关系的不同实体拥有不同的表示，但是在关系平面内的投影相同；同一实体可以在不同的关系平面内拥有不同的含义（平面内的投影）

![20190601155935483248827.jpg](http://image.nysdy.com/20190601155935483248827.jpg)

如图所示，对于正确的三元组来说$(h, r, t) \in \Delta$，所需满足的关系如图所示。那么对于一个实体$h''$如果满足$\left(h^{\prime \prime}, r, t\right) \in \Delta	$，在transE中是需要$h''=h$，而在transH中则将约束放宽到$h,h''$在$W_r$上的投影相同就可以了，也可以实现将$h,h''$区分开并且具有不同的表示。

#### 目标函数

scoring function：
$$
d(h+r, t)=f_{r}(h, t)=\left\|h_{\perp}+d_{r}-t_{\perp}\right\|_{2}^{2}
$$
As the hyperplane $W_r$, the $w_r$ is the normal vector of it, and $\left\|w_{r}\right\|_{2}^{2}=1$, so the projection $h$ in $w_r$ is:
$$
h_{w_{r}}=w_r^{T} h w_r
$$
其中，$w_r^{T} h=|w_r||h| \cos \theta$可以表示$h$在$w_r$上的投影的长度和$w_r$长度的乘积，因为$\left\|w_{r}\right\|_{2}^{2}=1$,所以可以代表投影的长度，再乘上单位向量即可表示投影向量。所以：
$$
\mathbf{h}_{\perp}=\mathbf{h}-\mathbf{w}_{r}^{\top} \mathbf{h w}_{r}, \quad \mathbf{t}_{\perp}=\mathbf{t}-\mathbf{w}_{r}^{\top} \mathbf{t} \mathbf{w}_{r}
$$
如图所示：![2019060115593616504994.jpg](http://image.nysdy.com/2019060115593616504994.jpg)

the score function is:
$$
f_{r}(\mathbf{h}, \mathbf{t})=\left\|\left(\mathbf{h}-\mathbf{w}_{r}^{\top} \mathbf{h w}_{r}\right)+\mathbf{d}_{r}-\left(\mathbf{t}-\mathbf{w}_{r}^{\top} \mathbf{t} \mathbf{w}_{r}\right)\right\|_{2}^{2}
$$

#### Training

loss function consists of margin-based ranking loss and some constraints:
$$
\begin{aligned} \mathcal{L} &=\sum_{(h, r, t) \in \Delta\left(h^{\prime}, r^{\prime}, t^{\prime}\right) \in \Delta_{(h, r, t)}}\left[f_{r}(\mathbf{h}, \mathbf{t})+\gamma-f_{r^{\prime}}\left(\mathbf{h}^{\prime}, \mathbf{t}^{\prime}\right)\right]_{+} \\ &+C\left\{\sum_{e \in E}\left[\|\mathbf{e}\|_{2}^{2}-1\right]_{+}+\sum_{r \in R}\left[\frac{\left(\mathbf{w}_{r}^{\top} \mathbf{d}_{r}\right)^{2}}{\left\|\mathbf{d}_{r}\right\|_{2}^{2}}-\epsilon^{2}\right]_{+}\right\}, \text { (4) } \end{aligned}
$$
the constraints:
$$
\forall e \in E,\|\mathrm{e}\|_{2} \leq 1, // \text { scale }\\
\forall r \in R,\left|\mathbf{w}_{r}^{\top} \mathbf{d}_{r}\right| 
/\left\|\mathbf{d}_{r}\right\|_{2} \leq \epsilon, / / \text { orthogonal }\\
\forall r \in R,\left\|\mathbf{w}_{r}\right\|_{2}=1, / / \text { unit normal vector }
$$

- **the second grantees the translation vectot $d_r$ is in the hyperplane**
- they project each $w_r$ to unit $l_2$-ball before visiting each mini-batch

> 既然transH可以完成将同一实体映射到不同的关系平面来获得不同的含义，那么我觉得
>
> - 是不是不同代表同一含义的投影表示应该相同或者相似
> - 这样是不是可以解决同一个实体的多义性问题。

## Reducing Ralse Negative Labels

Authors set different probabilities for replacing the head or tail entity depending on the mapping property of the relation (one-to-many, many-to-one, many-to-many)

- give more chance to replacing the head entity if the relation is one-to-many

  - 分别统计每个头实体对应尾实体的数量（反之亦然），按占比进行生成负样例

> - 通过这样的方式，例如one-many关系，替换头实体显然更不容易得到正样例（因为只有一种头实体是对的，然而替换尾实体因为对于头实体对应该关系的尾实体更多，说不定就有其他不在此many中的尾实体符合这个关系。
> - 相比之下我认为在《Bootstrapping-Entity-Alignment-with-Knowledge-Graph-Embedding》采用的均匀截断负采样效果会更好一些

# Experiments

the detail can be seen in the paper

## Link prediction

### outperform TransE in one-to-one

Authors explain:

- entities are connected with relations so that better embeddings of some parts lead to better results on the whole.

> 我是觉得有些牵强，不过要是硬理解也是可以，毕竟通过投影相当于把实体和关系进行了一个联系，可能这个增强了效果。

## Triplets Classification

This means FB13 is a very dense subgraph where strong correlations exist between entities

## Relational Fact Extraction from Text

- Actually, knowledge graph embedding is able to score a candidate fact, without observing any evidence from ex- ternal text corpus

> 可以看到从14年开始就有利用知识图谱来从文本抽取关系，最近这个应用好像又有起色，这个也可作为自己实验的一部分。

# Reference

- https://blog.csdn.net/MonkeyDSummer/article/details/85273843