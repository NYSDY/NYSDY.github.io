---
title: COMPOSITION-BASED MULTI-RELATIONAL GRAPH CONVOLUTIONAL NETWORKS 阅读笔记
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-03 15:30:05
password:
english_title: COMPOSITION-BASED MULTI-RELATIONAL GRAPH CONVOLUTIONAL NETWORKS 
tags:
- GCN
- KG
categories:
- 论文阅读笔记
---

# 0. 导读

## 0.1 文章是关于什么的？（what？）

图卷积网络，

## 0.2 要解决什么问题？（why？|challenge）

- 以前的方法主要集中于处理简单无向图
- 大多数现存模型在处理多关系图示遭受参数过大或者只学习节点表示

## 0.3 用什么方法解决？（how？）

- 在关系图中联合嵌入节点和关系的图卷积框架

## 0.4文章有什么创新？

- 在关系图中联合嵌入节点和关系的图卷积框架
- 该框架可以推广到几个现存的多关系GCN模型

## 0.5 效果如何？

在节点分类，链接预测，图分类上都取得了明显的提升。

## 0.6 还存在什么问题？

# 1 背景知识

暂无

# 2 模型

**模型图结构：**![](http://image.nysdy.com/20200903213538.png)

作者在图中添加了反关系。

> The GCN formulation as devised by Marcheggiani & Titov (2017) is based on the assumption that information in a directed edge ﬂows along both directions.

多关系图表示为：$\mathcal{G}=(\mathcal{V}, \mathcal{R}, \mathcal{E}, \mathcal{X}, \mathcal{Z})$， 其中$\boldsymbol{\mathcal { Z }} \in \mathbb{R}|\mathcal{R}| \times d_{0}$表示初始关系特征。作者拓展了自反边和关系：
$$
\left.\mathcal{E}^{\prime}=\mathcal{E} \cup\left\{\left(v, u, r^{-1}\right) \mid(u, v, r) \in \mathcal{E}\right\} \cup\{(u, u, \top) \mid u \in \mathcal{V})\right\}
$$

$$
\mathcal{R}^{\prime}=\mathcal{R} \cup \mathcal{R}_{\text {inv}} \cup\{\top\}
$$



- 其中$ \mathcal{R}_{\text {inv}}=\left\{r^{-1} \mid r \in \mathcal{R}\right\}$代表自反关系，$\top$代表自循环。

## 基于关系的合成

CompGCN学习了一个关系表示$\boldsymbol{h}_{r} \in \mathbb{R}^{d}, \forall r \in \mathcal{R}$， 节点嵌入：$\boldsymbol{h}_{v} \in \mathbb{R}^{d}, \forall v \in \mathcal{V}$把关系表示成向量可以避免过渡参数化问题，同时作者利用可以得到的关系特征作为初始表示。作者利用(Bordes et al., 2013; Nickel et al., 2016)使用的关系实体结合操作得到如下形式：
$$
\boldsymbol{e}_{o}=\phi\left(\boldsymbol{e}_{s}, \boldsymbol{e}_{r}\right)
$$

- $\phi: \mathbb{R}^{d} \times \mathbb{R}^{d} \rightarrow \mathbb{R}^{d}$是一个结合操作（作者使用了transE， multiplication， circular-correlation—方式）。$s,r,o$分别表示主语，关系和宾语，$\boldsymbol{e}_{(\cdot)} \in \mathbb{R}^{d}$表示他们的相关嵌入。

## CompGCN

节点更新公式：
$$
\boldsymbol{h}_{v}=f\left(\sum_{(u, r) \in \mathcal{N}(v)} \boldsymbol{W}_{\lambda(r)} \phi\left(\boldsymbol{x}_{u}, \boldsymbol{z}_{r}\right)\right)
$$

- 其中$\boldsymbol{x}_{u}, \boldsymbol{Z}_{r}$分别代表节点$u$和关系$r$，$h_v$代表节点$v$被更新表示。 
- $\boldsymbol{W}_{\lambda(r)} \in \mathbb{R}^{d_{1} \times d_{0}}$代表关系类型的特定参数。

其中作者使用了方向特定的权重，$\lambda(r)=\operatorname{dir}(r)$，被给与以下形式：
$$
\boldsymbol{W}_{\operatorname{dir}(r)}=\left\{\begin{array}{ll}\boldsymbol{W}_{O}, & r \in \mathcal{R} \\ \boldsymbol{W}_{I}, & r \in \mathcal{R}_{i n v} \\ \boldsymbol{W}_{S}, & r=\top(\text {self-loop})\end{array}\right.
$$
在节点更细以后，关系嵌入被转换为以下形式：
$$
\boldsymbol{h}_{r}=\boldsymbol{W}_{\mathrm{rel}} \boldsymbol{z}_{r}
$$


- $\boldsymbol{W}_{\mathrm{rel}} \in \mathbb{R}^{d_{1} \times d_{0}}$是一个可学习的转换矩阵，把所有的关系映射到与节点相同的嵌入空间，并再下一层可以被利用。

作者展示了各种GCN每层的参数对比

![](http://image.nysdy.com/20200904151209.png)

### 随着关系数量的增加而缩放

作者使用了Schlichtkrull等人（2017）提出的基础公式的一种变体。 它们不是独立定义每个关系的嵌入，而是表示为一组基本向量的线性组合。 正式地，让$\left\{\boldsymbol{v}_{1}, \boldsymbol{v}_{2}, \ldots, \boldsymbol{v}_{\mathcal{B}}\right\}$为一组可学习的基础向量。 然后，初始关系表示为：
$$
\boldsymbol{z}_{r}=\sum_{b=1}^{\mathcal{B}} \alpha_{b r} \boldsymbol{v}_{b}
$$

- $\alpha_{b r} \in \mathbb{R}$是关系和基础特定的可学习标量权重。

### 关于与Relational-GCN的比较注意

这与Schlichtkrull等人的基础公式不同，其中为每个GCN层定义了一组单独的基础矩阵。 相比之下，COMPGCN使用嵌入矢量代替矩阵，并且仅在第一层定义基本矢量。 后面的层根据等式4通过转换共享关系。这使作者的模型比Relational-GCN更有效的参数。

通过堆叠COMPGCN层，可以得到：
$$
\boldsymbol{h}_{v}^{k+1}=f\left(\sum_{(u, r) \in \mathcal{N}(v)} \boldsymbol{W}_{\lambda(r)}^{k} \phi\left(\boldsymbol{h}_{u}^{k}, \boldsymbol{h}_{r}^{k}\right)\right)
$$
相似的，可以得到堆叠的关系表示：
$$
\boldsymbol{h}_{r}^{k+1}=\boldsymbol{W}_{\mathrm{rel}}^{k} \boldsymbol{h}_{r}^{k}
$$
这里$\boldsymbol{h}_{v}^{0} \text { and } \boldsymbol{h}_{r}^{0}$代表初始的节点和关系特征。

### Proposition 4.1.

> COMPGCN generalizes the following Graph Convolutional based methods: KipfGCN (Kipf & Welling, 2016), Relational GCN (Schlichtkrull et al., 2017), Directed GCN (Marcheggiani & Titov, 2017), and Weighted GCN (Shang et al., 2019

证明：例如对于Kipf-GCN，这可以通过使权重$\left(\boldsymbol{W}_{\lambda(r)}\right)$和合成函数$(\phi)$与关系无关来简单地获得，即$\boldsymbol{W}_{\lambda(r)}=\boldsymbol{W} \text { and } \phi\left(\boldsymbol{h}_{u}, \boldsymbol{h}_{r}\right)=\boldsymbol{h}_{u}$。 对于其他方法，也可以得到类似方法获得，如表2所示。

# 3 实验

任务：链接预测、节点分类、图分类

基线：

- Relational-GCN (R-GCN)
- Directed-GCN (D-GCN)
- Directed-GCN (D-GCN)

## 数据集

![](http://image.nysdy.com/20200904153012.png)



## 链接预测

![](http://image.nysdy.com/20200904155302.png)



## 与其他GCN作对比

![](http://image.nysdy.com/20200904155509.png)
$$
\begin{array}{l}\text { Subtraction }(\mathbf{S u b}): \phi\left(e_{s}, e_{r}\right)=e_{s}-e_{r} \\ \text { Multiplication (Mult): } \phi\left(e_{s}, e_{r}\right)=e_{s} * e_{r} \\ \text { Circular-correlation (Corr): } \phi\left(e_{s}, e_{r}\right)=e_{s} \star e_{r}\end{array}
$$

- 这里有个疑问，对于那些没有关系嵌入的模型要怎么处理，是随机生成，然后通过分数函数来进行更新嘛。

![](http://image.nysdy.com/20200904153745.png)

总的来说，作者观察到观察到像循环相关这样的更复杂的算子的性能要好于或比简单的算子（如减法）要好。

## CompGCN的缩放能力

我们分析了具有不同数量的关系和基向量的COMPGCN的scalability。

### 变化关系基向量的影响

![](http://image.nysdy.com/20200904160518.png)

模型性能随着基础数量的增加而提高。作者注意到注意到在B = 100的情况下，模型的性能变得与所有关系都有其各自嵌入的情况相当。

### 关系数量的影响

![](http://image.nysdy.com/20200904161031.png)

作者报告使用5个关联基向量（B = 5）的COMPGCN相对于每个关系都使用单独的向量的相对性能。结果表明，随着关系数量的增加，COMPGCN的参数有效变体也随之缩放。

### 与R-GCN相比

![](http://image.nysdy.com/20200904160211.png)

## 节点分类和图分类

![](http://image.nysdy.com/20200904161536.png)



# 参考链接

- [论文下载地址](https://arxiv.org/pdf/1911.03082)