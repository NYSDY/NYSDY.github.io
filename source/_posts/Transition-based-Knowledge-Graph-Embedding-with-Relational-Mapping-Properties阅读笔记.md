---
title: Transition-based Knowledge Graph Embedding with Relational Mapping Properties阅读笔记
copyright: true
top: false
date: 2019-08-31 21:36:06
english_title: Transition-based_Knowledge_Graph_Embedding_with_Relational_Mapping_Properties
tags:
- KGE
- 2014
categories:
- 论文阅读笔记
- KG
---

> [论文下载地址](https://link.zhihu.com/?target=https%3A//pdfs.semanticscholar.org/0ddd/f37145689e5f2899f8081d9971882e6ff1e9.pdf)

<!-- more -->

# Related Work

### 把知识图谱映射到低维向量的原因：

符号和离散的存储结构使我们很难利用这些知识来增强其他智能获取的应用程序（例如问答系统），因为许多与AI相关的算法更倾向于进行关于连续数据计算。

```
文中用ONE-TO-ONE (husband-to-wife), MANY-TO-ONE (children-to-father), ONE-TO- MANY (mother-to-children), MANY-TO-MANY (parents-to-children)
来进行非单一关系的例子觉得很好。
```

### 以前算法的目标函数和参数复杂度：

![20190901156734544015276.png](http://image.nysdy.com/20190901156734544015276.png)

# Model

```
理解不了是如何区分1对多关系的？？？？
```

作者将最优函数将通过对应于该关系的预先计算的权重给出每个训练三元组的不同方面。

![20190902156739285578897.png](http://image.nysdy.com/20190902156739285578897.png)

实际上，大约只有26.2％的ONE-TO-ONE三元组适合由TransE建模。 另一方面，其余三元组（73.8％）受到影响，如图1左侧所示。

### 动机

根据作者的观察，三元组的映射属性在很大程度上取决于它的关系。

### 目标函数

权重是关系特定的，作者为三元组（h，r，t）提出的新评分函数是：
$$
f_{r}(h, t)=w_{\mathbf{r}}\|\mathbf{h}+\mathbf{r}-\mathbf{t}\|_{L_{1} / L_{2}}
$$

$$
w_{r}=\frac{1}{\log \left(h_{r} p t_{r}+t_{r} p h_{r}\right)}
$$

![20190902156739214061518.png](http://image.nysdy.com/20190902156739214061518.png)

测量关系的映射属性程度的一种简单方法是计算每个不同头部实体的尾部实体的平均数量，反之亦然。

```
这里还是不理解这个权重的意义在哪里？🧐
```



### 损失函数

$$
\begin{array}{l}{\mathcal{L}=\min \sum_{(h, r, t) \in \Delta\left(h^{\prime}, r, t^{\prime}\right) \in \Delta_{(h, r, t)}^{\prime}}\left[\gamma+f_{r}(h, t)-f_{r}\left(h^{\prime}, t^{\prime}\right)\right]_{+}} \\ {\text {s.t. } \quad \forall e \in E,\|e\|_{2}=1}\end{array}
$$

#### 对于$\|e\|_{2}=1$的解释：

约束位于单位球上的每个实体的原因是为了保证它们可以以**相同的比例更新**，而不是太大或太小而不能满足最佳目标。

# Experiments

见论文，没什么好阐述的。

All the codes for the related models can be downloaded from https://github.com/glorotxa/SME