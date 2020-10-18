---
title: Knowledge Graph Embedding via Dynamic Mapping Matrix阅读笔记
date: 2019-08-24 19:58:52
copyright: true
tags:
- TransD
- KGE
english_title: Knowledge_Graph_Embedding_via_Dynamic_Mapping_Matrix
categories:
- 论文阅读笔记
---



> 论文下载地址

<!-- more -->

# Problem Statement

1. 对于特定的关系$r$，所有的实体都共享相同的映射矩阵$M_r$。然而，由关系链接的实体总是包含各种类型和属性。
2. 投影是实体和关系之间的交互过程，映射矩阵只能由关系决定是不合理的。
3. 矩阵向量乘法使其具有大量计算，并且当关系数大时，它还具有比TransE和TransH更多的参数。 由于复杂性，TransR / CTransR难以应用于大规模知识图

# Contribution

1. 作者构建了一个新颖的模型TransD，通过同时考虑实体和关系的多样性，为每一个实体-关系构建动态映射矩阵。它为实体表示映射到关系向量空间提供灵活的样式。
2. 与TransR / CTransR相比，TransD具有更少的参数并且没有矩阵向量乘法
3. 在实验中，作者的方法优于之前的模型。

# Model

模型在TransD中，每个命名的符号对象（实体和关系）由两个向量表示。 第一个捕获实体（关系）的含义，另一个用于构造映射矩阵。

![20190826156680488849411.png](http://image.nysdy.com/20190826156680488849411.png)

对于一个三元组$(h,r,t)$,向量一共有$h, h_p, r, r_p, t, t_p$,其中带$p$的为映射向量，则有
$$
\begin{aligned} \mathbf{M}_{r h} &=\mathbf{r}_{p} \mathbf{h}_{p}^{\top}+\mathbf{I}^{m \times n} \\ \mathbf{M}_{r t} &=\mathbf{r}_{p} \mathbf{t}_{p}^{\top}+\mathbf{I}^{m \times n} \end{aligned}
$$
故
$$
\mathbf{h}_{\perp}=\mathbf{M}_{r h} \mathbf{h}, \quad \mathbf{t}_{\perp}=\mathbf{M}_{r t} \mathbf{t}
$$
可以综合为：
$$
\begin{aligned} \mathbf{h}_{\perp} &=\mathbf{M}_{r h} \mathbf{h}=\mathbf{h}+\mathbf{h}_{p}^{\top} \mathbf{h} \mathbf{r}_{p} \\ \mathbf{t}_{\perp} &=\mathbf{M}_{r t} \mathbf{t}=\mathbf{t}+\mathbf{t}_{p}^{\top} \mathbf{t} \mathbf{r}_{p} \end{aligned}
$$
这样就没有矩阵和向量间的乘法运算，变成向量间运算，提升计算速度。

# Experiments and Results Analysis

常规实验：triplets classification and link prediction不再赘述。

作者在实验过程中关注了一些具有更低accuracy的关系。

![20190826156680558013943.png](http://image.nysdy.com/20190826156680558013943.png)

分析：

	1. 对于$similar_to$关系主要因为训练数据不充足，只占了1.5%。
 	2. 对于最右侧的图说明了bern方法的效果要好于unif

### Properties of Projection Vectors

作者还做了case study，通过不同类型实体和关系的投影向量的相似性表明了作者方法的合理性