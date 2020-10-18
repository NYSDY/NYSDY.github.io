---
title: 《Differentiating Concepts and Instances for Knowledge Graph Embedding》阅读笔记
date: 2018-12-24 15:15:30
tags:
- 知识图谱
english_title: read_Differentiating_Concepts_and_Instances_for_Knowledge_Graph_Embedding
categories:
- 论文阅读笔记
---

> [论文获取地址](http://aclweb.org/anthology/D18-1222)。这篇文章最大的亮点就是把concept映射为一个球面，然后把instance映射为一个向量，通过这种空间关系来进行embedding。如果instance和concept满足InstanceOf的关系，则instance应该在球内；如果两个concept满足SubClassOf的关系，则一个球会在另一个球面内。

### concept

A concept is a fundamental category of existence (Rosch, 1973) and can be reified by all of its actual or potential instances.Concepts, which **represent a group of different instances sharing common properties**, are essential information in knowledge representation. 

![](https://i.loli.net/2018/12/16/5c1634f6c26c3.jpg)

## drawback of the previous method

 ignore to distinguish between concepts and instances will lead to two drawbacks:

- **Insufficient concept representation**：

  cannot explicitly represent the difference between concepts and instances

- **Lack transitivity of both isA relations**:

  *instanceOf* and *subClassOf* (generally known as isA)isA relations exhibit transitivity

## contributions

- **the first to propose** and formalize the problem of knowledge graph embedding which **differentiates** between concepts and instances
- a novel knowledge embedding method named **TransC**
- **state-of-the-art** on link prediction and triple classification

## Translation-based Models

### TransE

- triple (h, r, t) should satisfy **h + r ≈ t**
- loss function:$f_r(h,t) = ||h + r - t||^2_2$
- suitable for 1-to-1 relations

### TransH

- It regards a relation vector r as a translation on a hyperplane with $w_r$ as the normal vector. 
- loss function:$f_r(h,t) = ||h_{\bot} + r - t_{\bot}||^2_2$，其中$h_{\bot}=h-w^{\top}_r h w_r$，$t_{\bot}=t-w^{\top}_r t w_r$
- suitable for 1-to-N, N-to-1, and N-to-N relations

### TransR/CTransR 

- addresses the issue in TransE and TransH that some entities are **similar in the entity space** but comparably **different in other specific aspects**.
- loss function:$f_r(h,t) = ||M_rh +r -M_rt||^2_2$，$M_r$ for each relation r

### TransD

- considers the different types of entities and relations at the same time
- loss function:$f_r(h,t) = ||h_{\bot} + r - t_{\bot}||^2_2$，$h_{\bot} = M_{rh}h$和$t_{\bot} = M_{rt}t$，$M_{r,e}$ for each relation-entity pair (r, e)

## Bilinear Models

### RESCAL

- the **first** bilinear model
- It associates each entity with a vector to capture its **latent semantics**. Each relation is represented as a **matrix** which models pairwise interactions between latent factors.

## External Information Learning Models

- textual information
- entity descriptions

## Problem Formulation

这部分中作者详细介绍了知识图谱的组成部分：概念和实例集、关系集（包括instanceOf、subClassOf和instance relation），三元组集（按照关系集同样分为三个部分）。为了能表达is A关系的传递性，作者将instanceOf和subClassof两种关系进行了精心的设计，也是该论文的重点。

> For each concept c ∈ C, we learn a sphere s(p, m) with $p \in R^k$ and m denoting the sphere center and radius.

## TranC

Specifically, TransC encodes each concept in knowledge graph as a sphere and each instance as a vector in the same semantic space. 

### InstanceOf Triple Representation 

loss function：$f_e(i,c) = ||i-p||_2 - m$，当该函数大于0时进行优化，使其小于零。

### SubClassOf Triple Representation

![](https://i.loli.net/2019/03/01/5c789a0f00510.jpg)

如图所示，其中子图（a）为目标状态。两个概念的的圆心距离：$d = ||p_i - p_j||_2$。需要做到的就是$d-(m_j -m_i)  \leq 0$并且$ (m_j > m_i)$。

### Relational Triple Representation

这部分按照TranE的思路进行处理，$||h+r-t||^2_3$

## train model

### [margin based loss](https://zhuanlan.zhihu.com/p/27748177)详解

### [unit and bern](https://pdfs.semanticscholar.org/2a3f/862199883ceff5e3c74126f0c80770653e05.pdf) 

>  Regarding the strategy of constructing negative labels, we use “unif” to denote the traditional way of replacing head or tail with equal probability, and use “bern.” to denote reducing false negative labels by replacing head or tail with different probabilities.

## the following research directions

- find a more expressive model instead of spheres to represent concepts

- A concept may have different meanings in different triples. 

  use several typical vectors of instances as a concept’s centers to represent different meanings of a concept. 