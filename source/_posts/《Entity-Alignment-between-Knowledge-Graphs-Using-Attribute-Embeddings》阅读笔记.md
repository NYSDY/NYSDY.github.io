---
title: 《Entity Alignment between Knowledge Graphs Using Attribute Embeddings》阅读笔记
date: 2019-03-28 08:57:15
tags: 
- 知识图谱
- 知识图谱嵌入
english_title: Entity Alignment between Knowledge Graphs Using Attribute Embeddings
categories:
- 论文阅读笔记
---

> [论文下载地址](https://people.eng.unimelb.edu.au/jianzhongq/papers/AAAI2019_EntityAlignment.pdf)，知识图之间的实体对齐的任务旨在在代表相同现实世界实体的两个知识图中找到实体。本文最主要就是提出了属性字符嵌入(attribute character embeddings)的方法。

<!-- more -->

## Abstract

我们的模型利用知识图中存在的大量属性三元组(attribute triples)并生成属性字符嵌入。 属性字符嵌入(attribute character embeddings)通过基于实体的属性计算实体之间的相似性，将实体嵌入从两个知识图移位到同一空间中。
我们使用传递规则来进一步丰富实体的属性数量以增强属性字符嵌入。

## Contribution

- 提出了两个KG之间实体对齐的框架，它由谓词对齐模块，嵌入学习模块和实体对齐模块组成。
- 提出了一种新颖的嵌入模型，它将实体嵌入与属性嵌入集成在一起，以便为两个KG学习统一的嵌入空间。
- 我们在三个真正的KG对上评估建议的模型。
  结果表明，我们的模型在实体对齐任务上始终优于最先进的模型，hits@1超过50％。

## 模型

### 模型总览

predicate alignment, embedding learning, and entity alignment

![20190329155385087492759.png](http://image.nysdy.com/20190329155385087492759.png)

### Predicate Alignment

谓词对齐模块通过使用统一的命名方案重命名两个KG的谓词来合并两个KG，以便为关系嵌入提供统一的向量空间。dbp:bornIn vs. yago:wasBornIn 统一命名为 :bornIn。

为了找到部分匹配的谓词，作者计算谓词URI的最后部分的编辑距离（例如，bornIn与wasBornIn）并将0.95设置为相似性阈值。

## Embedding Learning

### Structure Embedding

作者采用TransE来学习对于实体的结构嵌入。与TransE不同的是，模型希望更关注已对齐的三元组，也就是包含对齐谓词的三元组。模型通过添加权重来实现这一目的。Structure embedding的目标函数如下：

![2019040115540798067483.png](http://image.nysdy.com/2019040115540798067483.png)

count(r)是关系r出现的数量。

### Attribute Character Embedding

对于属性字符嵌入，也参考TransE的思想，将谓词r解释为从头部实体h到属性a的转换。但是，相同的属性a可以在两个KG中以不同的形式出现，例如50.9989对50.9988888889作为实体的纬度; “Barack Obama”与“Barack Hussein Obama”作为人名等。因此，本文提出使用组合函数对属性值进行编码，并将属性三元组中每个元素的关系定义为h +r≈fa（a）。 这里，fa（a）是组合函数，a是属性值a = {c1，c2，c3，...，ct}的字符序列。 组合函数将属性值编码为单个向量，并将类似的属性值映射到类似的向量表示。 作者定义了三个组成函数如下。

#### Sum compositional function (SUM)

存在问题：包含相同字符不同顺序的属性值会有相同的向量表示

![20190401155408045744772.png](http://image.nysdy.com/20190401155408045744772.png)

#### LSTM-based compositional function (LSTM).

![20190401155408065416786.png](http://image.nysdy.com/20190401155408065416786.png)

#### N-gram-based compositional function (N-gram)

![20190401155408070480904.png](http://image.nysdy.com/20190401155408070480904.png)

最后attribute character embedding目标函数：![20190401155408075852436.png](http://image.nysdy.com/20190401155408075852436.png)

### Joint Learning of Structure Embedding and Attribute Character Embedding

作者使用属性字符嵌入通过最小化以下目标函数将结构嵌入移动到相同的向量空间：

![20190401155408233583946.png](http://image.nysdy.com/20190401155408233583946.png)

本文整体损失函数：

![20190401155408096945378.png](http://image.nysdy.com/20190401155408096945378.png)

### Entity Alignment

在经过上述训练过程之后，来自不同KG的相似的实体将会有相似的向量表示，因此可通过

![2019040115540811204332.png](http://image.nysdy.com/2019040115540811204332.png)

获得潜在实体对齐对<h_1, h_map>。此外，模型设定相似度阈值来过滤潜在实体对齐对，得到最终的对齐结果。

### Triple Enrichment via Transitivity Rule

作者利用一阶逻辑传递关系来丰富三元组。即：存在<h_1,t_1,t>和<t, r_2,t_2>则可以推理出h_1+ (r_1.r_2) ≈ t_2

## Database

本文从 DBpedia (DBP)、LinkedGeoData (LGD)、Geonames (GEO) 和 YAGO 四个 KG 中抽取构建了三个数据集，分别是DBP-LGD、DBP-GEO和DBP-YAGO。具体的数据统计如下：

![20190401155408149973345.png](http://image.nysdy.com/20190401155408149973345.png)

## Experiments

### Entity Alignment Results

本文对比了三个相关的模型，分别是 TransE、MTransE 和 JAPE。试验结果表明，本文提出的模型在实体对齐任务上取得了全面的较大的提升，在三种组合函数中，N-gram函数的优势较为明显。此外，基于传递规则的三元组丰富模型对结果也有一定的提升。具体结果如下![20190401155408163227980.png](http://image.nysdy.com/20190401155408163227980.png)

### Rule-based Entity Alignment Results

为了进一步衡量 attribute character embedding 捕获实体间相似信息的能力，本文设计了基于规则的实体对齐模型。本实验对比了三种不同的模型：以label的字符串相似度作为基础模型；针对数据集特点，在基础模型的基础之上增加了坐标属性，以此作为第二个模型；第三个模型是把本文提出的模型作为附加模型，与基础模型相结合。具体结果如下：

![20190401155408175594725.png](http://image.nysdy.com/20190401155408175594725.png)

### KG Completion Results

本文还在KG补全任务上验证了模型的有效性。模型主要测试了链接预测和三元组分类两个标准任务，在这两个任务中，模型也取得了不错的效果。具体结果如下：

![20190401155408179146202.png](http://image.nysdy.com/20190401155408179146202.png)