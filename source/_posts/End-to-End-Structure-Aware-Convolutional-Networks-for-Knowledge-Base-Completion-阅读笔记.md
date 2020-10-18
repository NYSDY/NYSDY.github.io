---
title: End-to-End Structure-Aware Convolutional Networks for Knowledge Base
  Completion--阅读笔记
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-08-25 15:41:18
password:
english_title: End-to-End Structure-Aware Convolutional Networks for Knowledge Base Completion reading note
tags:
- 2019
- AAAI
- GCN
- KGC
categories:
- 论文阅读笔记
---



# 0. 总览

## 0.1 要解决什么问题？（what？）

知识图谱补全中：

1. 以前的方法只建模关系三元组，忽略了大量图节点相关属性；
2. 以前的方法并没有增强在嵌入空间中大规模连接结构，完全忽略了图结构。

## 0.2 用什么方法解决？（how？）

用GCN和ConvErt相结合的方法组成一个端到端的学习方法。

1. encoder部分利用一个带权重的GCN的方法（利用图的结构并保留节点的属性）。
2. decoder部分利用Conv-TransE，利用啊convE的方法，但是去掉了实体和关系的矩阵重组部分。（为了保留TransE的 `h+r=t` 的特性）。

## 0.3文章有什么创新？

见上述方法

## 0.4 效果如何？

FB15k-237 and WN18RR数据机上，较ConvE提升了10%。

## 0.5 还存在什么问题？

# 1 背景知识



# 2 模型

end-to-end SACN![](http://image.nysdy.com/20200825165457.png)



## 2.1 权重图卷积层（WGCN）

节点$v_i$的第$l$层输出为：
$$
h_{i}^{l+1}=\sigma\left(\sum_{j \in \mathbf{N}_{i}} \alpha_{t}^{l} g\left(h_{i}^{l}, h_{j}^{l}\right)\right)
$$

- 其中$h_{j}^{l} \in \mathbb{R}^{F^{l}}$是节点$v_j$的输入，$v_j$是节点$v_i$的邻居节点（一共$N_i$个）。$\alpha_{t}^{l} $是第$l$层，第$t$种边（关系）类型的权重参数。

$g$函数是整合邻居信息函数，在这里坐着采用以下函数：
$$
g\left(h_{i}^{l}, h_{j}^{l}\right)=h_{j}^{l} W^{l}
$$

- 其中$W^{l} \in \mathbb{R}^{F^{l} \times F^{l+1}}$

作者再加入$v_i$的自连接，则节点$v_i$的传播过程可以被定义为：
$$
h_{i}^{l+1}=\sigma\left(\sum_{j \in \mathbf{N}_{i}} \alpha_{t}^{l} h_{j}^{l} W^{l}+h_{i}^{l} W^{l}\right)
$$

- 节点$h_i^{l+1}$是节点第$l$层输出的节点特征矩阵$H^{l+1} \in \mathbb{R}^{N \times F^{l+1}}$的第$l+1$行，代表$v_i$节点在第$l+1$层的特征。

![](http://image.nysdy.com/20200825171648.png)

以上的为不同类型的边加权重的过程可以看做一个矩阵乘法：通过一个邻接矩阵同时为所有节点计算embeddings，见上图：

邻接矩阵可以被写作：
$$
A^{l}=\sum_{t=1}^{T}\left(\alpha_{t}^{l} A_{t}\right)+I
$$

- 其中$I$是一个单位矩阵。$A^t$是一个二进制矩阵，当$v_i$和$v_j$节点间有边连接则其中第$ij$项值为1，否则为0。

故，由此可以等到每一层线性变换的所有一节邻居：
$$
H^{l+1}=\sigma\left(A^{l} H^{l} W^{l}\right)
$$

## 2.2 属性节点

在知识图谱中，属性节点的形式为(entity, relation, attribute)，例如：(s = Tom, r = people.person.gender, a = male)。但是如果用向量表达节点属性会出现两个问题：

1. 由于每个节点的属性数量少，会产生属性向量稀疏的问题；
2. 属性向量的0值会产生歧义：该节点没有该属性或者该属性缺失值。

本文还使用了节点的属性作为图的节点，如属性（Tom，gender，male）。这样做的目的是将属性也作为节点，起到“桥”的作用，相同属性的节点可以共享信息。还有作者为了减少过多的属性节点，对节点进行了合并， 将gender也作为了图中的节点，而不是建立male和female两个属性，理由是gender已经能够确定实体的person，而不必过多区分性别。（即作者把属性三元组变为了两元组（实体，属性名）

## 2.3 Conv-TransE

作者仅将节点向量$e_s$和关系向量$e_r$进行堆叠，然后进行卷积操作：
$$
\begin{aligned} m_{c}\left(e_{s}, e_{r}, n\right)=& \sum_{\tau=0}^{K-1} \omega_{c}(\tau, 0) \hat{e}_{s}(n+\tau) +\omega_{c}(\tau, 1) \hat{e}_{r}(n+\tau) \end{aligned}
$$

- 其中$K$是卷积核的宽度，$n$索引输出向量中的条目,$\omega_{c}$是卷积核参数。

分数函数被定义为：
$$
\psi\left(e_{s}, e_{o}\right)=f\left(\operatorname{vec}\left(\mathbf{M}\left(e_{s}, e_{r}\right)\right) W\right) e_{o}
$$

- $W \in \mathbb{R}^{C F^{L} \times F^{L}}$是一个线性变换，$\operatorname{vec}(\mathbf{M}) \in \mathbb{R}^{C F^{L}}$reshape成一个向量.

然后应用了一个逻辑sigmod函数：
$$
p\left(e_{s}, e_{r}, e_{o}\right)=\sigma\left(\psi\left(e_{s}, e_{o}\right)\right)
$$


# 3 实验

![](http://image.nysdy.com/20200826205435.png)



> 数据集：FB15k-237, WN18RR and FB15k-237-Attr
>
> baseline:DistMult，ComplEx，R-GCN，ConvE
>
> 评价准则：MRR(mean reciprocal rank)（Raw，Filtered），Hits @（1，3，10）

## 3.1 链接预测实验结果：![](http://image.nysdy.com/20200826205515.png)

## 3.2 卷积核大小分析

![](http://image.nysdy.com/20200826210216.png)

内核“ 2×1”表示在实体向量的一个属性和关系向量的对应属性之间转换的知识或信息。 如果将内核大小增加到“ 2×k”，其中k = {3，5}，则会在实体向量的s属性组合和关系向量的k属性组合之间转换信息。

## 3.3 节点度分析

![](http://image.nysdy.com/20200826210546.png)

知识图中节点的度数是连接到该节点的边数。

# 参考链接

- https://www.cnblogs.com/jws-2018/p/11519383.html

