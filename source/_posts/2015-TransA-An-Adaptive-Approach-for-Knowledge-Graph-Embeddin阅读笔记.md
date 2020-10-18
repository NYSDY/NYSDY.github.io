---
title: 2015 TransA An Adaptive Approach for Knowledge Graph Embeddin阅读笔记
copyright: true
top: false
date: 2019-09-02 14:32:48
mathjax: true
english_title: 2015_TransA_An_Adaptive_Approach_for_Knowledge_Graph_Embeddin
tags:
- TransA
- KGE
- 2015
- arxiv
categories:
- 论文阅读笔记
- KG
---

> 这篇文章具体的公式操作比较难以理解，但是对于我来说，它的每一维度加权和最后判断时也应该考虑不同维度差异和我的思路比较像。
>
> [论文下载地址](https://arxiv.org/pdf/1509.05490)

<!-- more -->

# Problem Statement

之前的机遇翻译的方法过于简化损失度量，使得模型没有足够的能力来模拟知识库中的各种复杂实体/关系。

# Introduction

- 由于损失度量的不灵活性，当前基于翻译的方法应用具有不同合理性的球面等势超表面，其中三元组更靠近中心，三元组更合理。如图1（a）所示，球面等势超表面不够灵活，不足以表征拓扑结构。

![20190909156799696760406.png](http://image.nysdy.com/20190909156799696760406.png)

- 其次，由于过度简化的损失度量，当前基于翻译的方法用同性的欧氏距离无法体现出各个维度的重要性，而是将各个维度的特征都等同看待。导致图2中出现的问题：![2019090915679977198009.png](http://image.nysdy.com/2019090915679977198009.png)

  由于对每一维度处理相同，不正确的实体将被匹配。但是通过对不同维度进行不同加权将会避免这种由于距离相同被匹配的错误。 

  # Model

作者提出TransA，一种利用自适应和灵活度量的嵌入方法：

- TransA采用椭圆形表面而不是球形表面：通过这种方式，复杂关系引起的复杂嵌入拓扑结构可以得到更好的表现。
- 如“自适应度量方法”中所分析的那样，TransA可以被视为加权变换特征维度。 因此，来自不相关尺寸的噪声被抑制。 

### 自适应度量分数函数

TransA采用自适应Mahalanobis绝对损失距离：
$$
f_{r}(h, t)=(|\mathbf{h}+\mathbf{r}-\mathbf{t}|)^{\top} \mathbf{W}_{\mathbf{r}}(|\mathbf{h}+\mathbf{r}-\mathbf{t}|)
$$
其中
$$
|\mathbf{h}+\mathbf{r}-\mathbf{t}| \doteq\left(\left|h_{1}+r_{1}-t_{1}\right|,\left|h_{2}+r_{2}-t_{2}\right|, \ldots, | h_{n}+\right.\left.\left.r_{n}-t_{n}\right\rfloor\right)
$$
，$W_r$是对应于特定关系的对称非负权重矩阵，也是自适应权重矩阵。与传统的得分函数不同，作者取绝对值，因为想要测量（h + r）和t之间的绝对损失。原因有以下两点：

- 作者将的分数函数扩展为诱导规范：
  $$
  N_{r}(\mathbf{e})=\sqrt{f_{r}(h, t)}
  $$
  这样可以用以下方式来简化运算：
  $$
  \begin{array}{l}{\text { inequality } N_{r}\left(\mathbf{e}_{1}+\mathbf{e}_{2}\right)=\sqrt{\left|\mathbf{e}_{1}+\mathbf{e}_{2}\right|^{\top} \mathbf{W}_{\mathbf{r}}\left|\mathbf{e}_{\mathbf{1}}+\mathbf{e}_{\mathbf{2}}\right|} \leq}  {\sqrt{\left|\mathbf{e}_{\mathbf{1}}\right|^{\top} \mathbf{W}_{\mathbf{r}}\left|\mathbf{e}_{\mathbf{1}}\right|}+\sqrt{\left|\mathbf{e}_{\mathbf{2}}\right|^{\top} \mathbf{W}_{\mathbf{r}}\left|\mathbf{e}_{\mathbf{2}}\right|}=N_{r}\left(\mathbf{e}_{\mathbf{1}}\right)+N_{r}\left(\mathbf{e}_{\mathbf{2}}\right)}\end{array}
  $$

- 在几何中，负值或正值表示向下或向上的方向。而在作者的方法中，作者不考虑这个因素。 让作者看一下如图2所示的实例。 对于实体Goniff，其损耗向量的x轴分量是负的，因此扩大该分量将使整体损失更小，而这种情况应该使整体损失更大。 因此，绝对算子对作者的方法至关重要。 对于没有绝对算子的数值例子，当嵌入维数为2时，权重矩阵为[0 1; 1 0]和损失矢量（h + r  -  t）=（e1，e2），总损失为2e1e2。 如果e1≥0且e2≤0，则绝对更大的e2将减少总损耗，这是不希望的

### 从等势面的角度

对于其他基于翻译的方法，等势超曲面是欧几里德距离定义的球体：
$$
\|(\mathbf{t}-\mathbf{h})-\mathbf{r}\|_{2}^{2}=\mathcal{C}
$$
其中，$\mathcal{C}$表示阈值或等势值。

而对于TransA来说，等势面是椭圆
$$
|(\mathbf{t}-\mathbf{h})-\mathbf{r}|^{\top} \mathbf{W}_{\mathbf{r}}|(\mathbf{t}-\mathbf{h})-\mathbf{r}|=\mathcal{C}
$$
​	由于知识库是大规模且非常复杂的实际情况，嵌入的拓扑结构不能像球体那样均匀分布，如图1所示。 因此，用椭圆形替换球面等势超曲面将增强嵌入

### 从特征权重角度

TransA可以看做是带有权重的特征变换，假设权重矩阵$W_r$是对称矩阵，那么可以通过LDL分解将权重分解为
$$
\begin{array}{c}{\mathbf{W}_{\mathbf{r}}=\mathbf{L}_{\mathbf{r}}^{\top} \mathbf{D}_{\mathbf{r}} \mathbf{L}_{\mathbf{r}}} \\ {f_{r}=\left(\mathbf{L}_{\mathbf{r}}|\mathbf{h}+\mathbf{r}-\mathbf{t}|\right)^{\top} \mathbf{D}_{\mathbf{r}}\left(\mathbf{L}_{\mathbf{r}}|\mathbf{h}+\mathbf{r}-\mathbf{t}|\right)}\end{array}
$$
相当于对loss向量通过$L_r$进行特征变换，其中，$\mathbf{D}_{r}=\operatorname{diag}\left(w_{1}, w_{2}, \ldots\right)$是一个对角矩阵，对角元素的值就是不同嵌入维度的权值。

### 对比之前方法

与关于旋转和缩放嵌入空间的TransR，TransA具有两个优势。

- 首先，作者对特征尺寸进行加权以避免噪音。 

- 其次，作者放松了PSD条件以获得灵活的表示。 

与使用预先计算的系数重新构造对特征尺寸进行加权的TransM，TransA具有两个优势。 

- 首先，作者从数据中学习权重，这使得分数函数更具适应性。 
- 其次，作者应用特征转换，使嵌入更有效

## 算法训练

loss函数为：
$$
\begin{array}{c}{\mathcal{L}=\sum_{(h, l, t) \in \Delta\left(h^{\prime}, l^{\prime}, t^{\prime}\right) \in \Delta^{\prime}}\left[\gamma+f_{r}(\mathrm{h}, \mathrm{t})-f_{r^{\prime}}\left(\mathrm{h}^{\prime}, \mathrm{t}^{\prime}\right)\right]_{+}+\lambda\left(\sum_{r \in R}\|\mathbf{W} r\|_{F}^{2}\right)+C\left(\sum_{e \in E}\|\mathbf{e}\|_{2}^{2}+\sum_{r \in R}\left\|\mathbf{r}_{2}^{2}\right\|\right)} \\ {\text { s.t. }\left[\mathbf{W}_{r}\right]_{i j} \geq 0}\end{array}
$$
保证非负性，作者将所有否定条目权重的值赋为0
$$
\mathbf{W}_{r}=-\sum_{(h, r, t) \in \Delta}\left(|\mathbf{h}+\mathbf{r}-\mathbf{t} \| \mathbf{h}+\mathbf{r}-\mathbf{t}|^{\top}\right)+\sum_{\left(h^{\prime}, r^{\prime}, t^{\prime}\right) \in \Delta^{\prime}}\left(\left|\mathbf{h}^{\prime}+\mathbf{r}^{\prime}-\mathbf{t}^{\prime}\right|\left|\mathbf{h}^{\prime}+\mathbf{r}^{\prime}-\mathbf{t}^{\prime}\right|^{\top}\right)
$$

### 模型复杂度

至于作者模型的复杂性，权重矩阵完全由现有的嵌入向量计算，这意味着TransA几乎具有与TransE相同的自由参数数。 至于作者模型的效率，权重矩阵有一个封闭的解决方案，这在很大程度上加快了培训过程

# 实验

作者统计了一个ATPE值（Averaged Triple number Per Entity.）该数量衡量数据集的多样性和复杂性

作者对于TransA在WN18上表现不好进行了分析解释。

> TransA在WN18数据集上的平均排名表现不佳。 深入研究详细情况，我们发现有27个测试三元组（测试集的0.54％），其排名超过30,000，这几个案例将导致约162个平均等级损失。 所有这些三元组的尾部或头部实体从未与训练集中的相应关系共存。 训练数据不足导致权重矩阵过度扭曲，权重矩阵过度扭曲导致平均排名不良

作者还做了一个实验

![20190909156803658869046.png](http://image.nysdy.com/20190909156803658869046.png)

作者解释：

> 精度随重量差异而变化，这意味着特征加权有利于精确度。 这证明了TransA的理论分析和有效性

```
但是我并不认为这个可以说明什么，那是不是调高其他关系的权重会带来该关系效果的提升？
```

