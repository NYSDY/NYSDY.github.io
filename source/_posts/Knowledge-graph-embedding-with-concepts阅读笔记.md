---
title: Knowledge graph embedding with concepts阅读笔记
date: 2019-07-25 13:19:44
copyright: true
top: false
cover: false
password:
toc: true
mathjax: true
tags:
- KGE
- ontology
- concept
english_title: Knowledge_graph_embedding_with_concepts
categories:
- 论文阅读笔记
- KG
---



> 这篇论文，运用skip-gram方法，将实体对应相关概念引入实体向量表示，以增强表示效果。实体和概念在同一空间中，但是概念是空间中的一个超平面（类似于transH）。文中举例很多例子来辅助说明，使得文章可读性大幅提升。文中实验最后俩个比较有意思。本文值得思考借鉴的东西不少，值得再好好回顾。
>
> [论文下载地址](https://www.sciencedirect.com/science/article/pii/S0950705118304945/pdfft?md5=b9bad12f2bc771990ad0feaefa7402d4&pid=1-s2.0-S0950705118304945-main.pdf)

<!-- more -->

# problem statement

- 已经存在的KGE模型主要集中于实体-关系-实体三元组或者文本语料交互。
  - 三元组是缺少信息的，并且域内文本不总是可以获得的——导致嵌入结果偏离实际
- 常识概念知识发挥很重要的作用。

# background



> For example, for two triplets (Apple, Developer, IPhone) and (Apple, Developer, Samsung Mobile), it is quite difficult to distinguish which is the true triplet that contains fact triplets only, because ‘‘IPhone’’ and ‘‘Samsung Mobile’’ both belong to mobile phones. However, in the concept graph, ‘‘IPhone’’ has a concept ‘‘apple device’’, but ‘‘Samsung Mobile’’ does not. Thus, it is easy to infer the correct triplet by mapping ‘‘IPhone’’ to the ‘‘apple device’’concept

```
很好的一个举例关于如何运用concept来辅助关系识别
```

> Specifically, when a corpus about technology is provided, embedding methods with technical textual descriptions could easily infer the fact (Apple, Developer, IPhone), because the keywords ‘‘hardware products’’ and ‘‘iPhone smartphone’’ occur frequently in the textual description of ‘‘Apple’’. However, it is difficult to infer the fact (Apple, Taste, Sweet), which is irrelevant to textual descriptions of ‘‘Apple’’ about the specific topic of ‘‘technology company

```
这里作者举例说明：与具有文本信息的嵌入方法相比，具有概念信息的嵌入方法在其任务中更加通用，并且它不依赖于语料库的主题。
```

作者把KGE分成了三类，如下：

- Embedding with symbolic triplets：trans系列都放到了这部分中
- Embedding with textual information
- Embedding with category information

# Methodology

## concept graph embedding 

作者采用skip-gram来学习可以捕获其语义相关性的概念和实体的表示。

![20190725156403562012591.png](http://image.nysdy.com/20190725156403562012591.png)

其中，每个实体对应多个概念，每个概念又包含多个实体（这些实体作为实体的上下文）。

则，skip-gram函数可以写为：
$$
\begin{array}{l}{P\left(e_{c} | e_{t}\right)=\frac{\exp \left(e_{c} \cdot e_{t}\right)}{\sum_{e \in E} \exp \left(e \cdot e_{t}\right)}} \\ {P\left(e_{c} | c_{i}\right)=\frac{\exp \left(e_{c} \cdot c_{i}\right)}{\sum_{e \in E} \exp \left(e \cdot c_{i}\right)}}\end{array}
$$
故损失函数为：
$$
L=\frac{1}{|D|} \sum_{\left(e_{c}, e_{t}\right) \in D}\left[\log P\left(e_{c} | e_{t}\right)+\sum_{c_{i} \in C\left(e_{t}\right)} \log P\left(e_{c} | c_{i}\right)\right]
$$
学习率设为：

α = starting_alpha×(1−count_actual/(real)(iter × total_size+1))

```
这里作者说为了避免过拟合，对优化目标采用“负抽样”方法。"负抽样"方法还可以避免过拟合？
```



## knowledge graph embedding

将特定三元组嵌入到概念子空间中，首先构建一个超平面，其中法向量$c$为概念子空间：
$$
c=C\left(e_{h}, e_{t}\right)=\frac{e_{h}-e_{t}}{\left\|e_{h}-e_{t}\right\|_{2}^{2}}
$$
根据TransE三元组的嵌入损失为：
$$
l=h+r-t
$$
所以，可以计算出法向量方向上的损失分量是：
$$
\left(c^{T} l c\right)
$$
然后，投影到超平面上的另一个正交分量是：
$$
\left(l-c^{T} l c\right)
$$
![20190725156403899071000.png](http://image.nysdy.com/20190725156403899071000.png)

定义总损失函数：
$$
f_{r}(h, t)=-\lambda\left\|l-c^{T} l c\right\|_{2}^{2}+\|l\|_{2}^{2}
$$

## Model interpretation

1. 可以通过概念来辅助三元组识别，文中以(Christopher Plummer, /people/person/nationality, Canada)举例
2. 可以解决在当两个候选实体在KGE，中计算loss相等时辨别这两个哪个是真实的。文中以“which the director made the film ‘‘WALL-E’’”为例来进行说明

```
都是通过查询实体对应概念来进行辅助
```



# Objectives and training

margin-based loss function：
$$
L=\sum_{(h, r, t) \in S} \sum_{\left(h^{\prime}, r, t^{\prime}\right) \in S_{(h, r, t)}^{\prime}}\left[\gamma+f_{r}(h, t)-f_{r^{\prime}}\left(h^{\prime}, t^{\prime}\right)\right]_{+}
$$

## train

1. 先预训练概念图模型嵌入，获得在概念空间中的实体向量
2. 利用1中获得的实体向量进行更新。

# datasets

- WN18 and FB15K

- Microsoft Concept Graph![20190725156404326996082.png](http://image.nysdy.com/20190725156404326996082.png)

  其中，relations表示频率

  ```
  真的有统计频率的这种
  ```

  

# Experiments

### Knowledge graph completion

### Entity classification

### Concept relevance analysis

![20190725156404278694395.png](http://image.nysdy.com/20190725156404278694395.png)

这个实验比较有意思：每个单元格中的数字表示在TransE中排名大于m且在我们的模型中小于n的三元组的数量。

### Precise semantic expression analysis

我们在链接预测（换句话说，这些是TransE的难以证明的例子）中收集那些得分略高于真实三元组作为负三元组

然后在KEC中对比两者的分数差值。

![20190725156404301070773.png](http://image.nysdy.com/20190725156404301070773.png)

右边条表示KEC在TransE失败时作出正确决定，左边条表示KEC和TransE都失败。