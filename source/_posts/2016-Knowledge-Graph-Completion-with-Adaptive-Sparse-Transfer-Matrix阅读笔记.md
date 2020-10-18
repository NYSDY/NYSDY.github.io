---
title: 2016 Knowledge Graph Completion with Adaptive Sparse Transfer Matrix阅读笔记
copyright: true
top: false
date: 2019-09-10 09:16:53
password:
english_title: 2016_Knowledge_Graph_Completion_with_Adaptive_Sparse_Transfer_Matrix
tags:
- 2016
- AAAI
- KGE
categories:
- 论文阅读笔记
---

> 为解决heterogeneity和imbalance问题，作者针对同一关系链接实体对的数量和同一关系不同头尾实体数量，分别设计了不同稀疏程度的转移矩阵。存在缺点：并没有同时解决这两个问题。
>
> [论文下载地址](https://link.zhihu.com/?target=https%3A//www.aaai.org/ocs/index.php/AAAI/AAAI16/paper/download/11982/11693)

<!-- more -->

# Problem Statement
- heterogeneity（异质性）：一些关系链接了很多实体对，另一些没有
- imbalance（不平衡）： 在一种关系中，头实体和尾实体的数量不同


# Contribution

1. 作者提出了一种新颖的方法，该方法考虑了先前模型中未使用的异质性和不平衡性，以嵌入知识图来完成它们； 
2. 作者的方法高效且参数较少，因此很容易扩展到大规模知识图；
3. 作者为转移矩阵提供了两种稀疏模式，并分析了它们的优缺点；
4. 在三元组分类和链接预测任务中，作者的方法达到了最先进的性能

# Sparse Matrix

### 定义

稀疏矩阵是指大多数条目（entries）为零的矩阵。 零元素占矩阵元素总数的比例称为稀疏度（sparse degree）。

### 类别

- 结构化的
- 非结构化的

![20191011157077419915060.png](http://image.nysdy.com/20191011157077419915060.png)

### 两者重要区别

- 结构化模式有利于矩阵向量乘积运算。
- 非结构化往往可以带来更好的实验结果：由于更加灵活地远范围线性组合。

# Sparse Matrix vs Low-Rank Matrix

### 特点

- low-rank矩阵强制一些变量要满足特定的约束，因此，矩阵**M**无法自由地进行赋值。
- sparse矩阵是作者令其中的部分元素值为0，并且在训练过程中不改变它的值，其他的非零值进行训练。

### 对比

- 稀疏矩阵比低秩矩阵更灵活，可以有更大的自由度：使用低秩矩阵，那么矩阵的自由度会受到严格的秩限制。然而，sparse matrix的稀疏性只是控制矩阵元素中零元素的个数。
- 稀疏矩阵比低秩矩阵更有效率：只有非零条目参与计算，极大地减少了计算量

# Model

## TranSpare(share)

- 解决heterogeneity问题

### 特点

- 转移矩阵的稀疏度由关系链接的实体对的数量确定
- 并且关系的两侧共享相同的转移矩阵
- 对于复杂关系的转移矩阵更加稀疏

```
不知道为什么复杂转移矩阵会更加稀疏？
```



### 转移矩阵的稀疏程度

$$
\theta_{r}=1-\left(1-\theta_{\min }\right) N_{r} / N_{r^{*}}
$$

其中，$N_r$代表链接关系$r$的实体对的数量，$r^*$代表链接最多实体对的关系，$\theta_{\min }\left(0 \leq \theta_{\min } \leq 1\right)$是一个超参数，代表矩阵$M_{r^{*}}$的最小系数程度。

```latex
不理解这里为什么把稀疏程度定义成这么麻烦，为什不直接定义成$\theta_{\min} N_{r} / N_{r^{*}}$
```

### 映射向量

$$
\mathbf{h}_{p}=\mathbf{M}_{r}\left(\theta_{r}\right) \mathbf{h}, \quad \mathbf{t}_{p}=\mathbf{M}_{r}\left(\theta_{r}\right) \mathbf{t}
$$



## TranSpare(separate)

- 解决imbalance问题

### 特点

- 每个关系具有两个单独的稀疏传递矩阵，一个用于头实体，另一个用于尾实体
- 稀疏度取决于通过关系链接的头（尾）实体的数量

### 转移矩阵的稀疏程度

$$
\theta_{r}^{l}=1-\left(1-\theta_{\min }\right) N_{r}^{l} / N_{r^{*}}^{l^{*}} \quad(l=h, t)
$$

和share类似，只是头尾实体不相同，增加l来代表头尾实体数量。

### 映射向量

$$
\theta_{r}^{l}=1-\left(1-\theta_{\min }\right) N_{r}^{l} / N_{r^{*}}^{l^{*}} \quad(l=h, t)
$$

## 分数函数

两者的分数函数相同均为：
$$
f_{r}(\mathbf{h}, \mathbf{t})=\left\|\mathbf{h}_{p}+\mathbf{r}-\mathbf{t}_{p}\right\|_{\ell_{1 / 2}}^{2}
$$

## 总loss

采用margin-based ranking loss
$$
$L=\sum_{(h, r, t) \in \Delta\left(h^{\prime}, r, t\right) \in \Delta^{\prime}}\left[\gamma+f_{r}(\mathbf{h}, \mathbf{t})-f_{r}\left(\mathbf{h}^{\prime}, \mathbf{t}^{\prime}\right)\right]_{+}$
$$

## 训练过程

为了加速训练时收敛以及防止过拟合，作者使用TransE算法的结果进行初始化实体和关系的embedding向量，对于转化矩阵，作者使用单位矩阵进行初始化。但是这不是非必须的

对于转化矩阵(假设为单位矩阵)，非零向量的个数$n z=\lfloor\theta \times n \times n\rfloor$，由于作者使用单位向量初始化，所以除了对角线上的非零元素之外，其他非零元素的个数为$n z^{\prime}=n z-n$，如果$n z \leq n$，那么作者设置$n z^{\prime}=0$。

- 在构建结构化的转化矩阵$\mathbf{M}(\theta)$的时候，作者要让$n z^{\prime}$非零元素对称分布在对角线的两边，如果$n z^{\prime}$不能满足要求，那么作者选择另外一个整数。
- 在构建非结构化的转化矩阵$\mathbf{M}(\theta)$的时候，作者只随机散布$\mathbf{M}(\theta)$中的$n z^{\prime}$非零元素（但不在对角线上）。

在训练前，作者首先设置超参数$\theta_{\min }$，然后计算每个转化矩阵的稀疏程度，然后，作者使用结构化或非结构化模式构建稀疏转化矩阵。

# 实验

与常规不同的就是加了一个实验![20191011157077982280661.png](http://image.nysdy.com/20191011157077982280661.png)

