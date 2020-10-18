---
title: 《OK Google, What Is Your Ontology? Or/ Exploring Freebase Classification to Understand Google’s Knowledge Graph？》阅读笔记
date: 2019-03-03 21:29:07
tags:
- Ontology
- 知识图谱
- freebase
english_title: OK Google, What Is Your Ontology? Or/ Exploring Freebase Classification to Understand Google’s Knowledge Graph？
categories:
- 论文阅读笔记

---

> 本论文详细阐述Freebase中的数据格式，并进行了重构。通过考虑整体架构的三个部分：Freebase类型系统及其缺乏继承和依赖于不兼容性，允许表示值的不确定性的实现，以及合并和拆分对象的实现。来对本体进行阐述。[论文下载地址](https://arxiv.org/pdf/1805.03885.pdf)

<!-- more -->

这篇论文重构了Freebase数据转储来理解谷歌语义搜索特征背后的本体。论文将会探索Freebase本体如何由许多力量塑造的，这些力量也通过深入研究本体论和一个小的相关性研究来形成分类系统。这些发现将会提供知识图谱专有黑盒的一瞥。

> The structures found in the Freebase/Knowledge Graph ontology will be analyzed in light of the findings on classification systems in a key text by Bowker and Star (2000) [5].

# 术语定义

![](https://i.loli.net/2019/03/03/5c7b61c1bd122.jpg)

### Object

Freebase对象是一个全局唯一的标识符，它是Freebase中世界上某种东西的表示。

### Type

Freebase类型用来表达类的概念。

### Property

Freebase属性是描述对象如何链接到其他值或对象的关系。

### Property Detail

属性详细信息指的是可以通过属性链接的对象或值的约束。

### RDF triple

资源描述格式（RDF）是用于“三元组”（或N = 3元组）格式的数据表示的规范[17]。

### Ontology

对于本文，Freebase本体是类型，属性和属性详细信息的正式结构和描述，用于指定对象如何相互关联。

### Architecture

在本文中，架构指的是可以在本体中找到的一般模式和关系。

**==本体是否允许类（或Freebase用语中的类型）之间的继承？ 是否有与属性相关的默认值？ 如何处理“零”或空值？ 这些类型的问题不一定关注本体（飞机，火车或汽车）中具体表达的内容，而是关于本体表达方式的更多问题应该通过检查架构来解决。==**

# Methodology

作者把数据进行切分：按照RDF中三元组的谓语进行分类，例如：![](https://i.loli.net/2019/03/03/5c7bc58f2052d.jpg)

# Freebase Ontology and Classification

>  As Bowker and Star note, “Information infrastructure is a tricky thing to analyze...the easier they are to use, the harder they are to see.” [5]. What does the system make sense of? What is left out? What is privileged and by extension what is ignored by Google?

虽然Freebase本体可能不会立即看起来像一个分类系统，但类型（类）和属性的结构是一个基于对各种事物进行分类的系统。

作为对世界事物表征进行排序和分类的系统，将根据Bowker和Star的分类结果讨论Freebase本体。他们将对亚里士多德和原型分类（Aristotelian and prototype classification）进行了区分。

亚里士多德的分类“按照一组二元特征进行操作，被分类的物体呈现或不呈现”，而原型分类则认为“在我们心目中对于椅子是什么的广泛描述; 我们用隐喻和类比来扩展这张图片“

## 5.1. Freebase’s Type System

不兼容性的概念出现在Freebase系统中，用于表示对象如何具有某些类型，而这些类型必须将其排除在其他类型之外。

没有继承（not implement inheritance）：上述不兼容性在确保数据不表达可能在Google KP中提供的令人尴尬，有害或不正确的陈述方面发挥了足够强大的作用。

缺乏继承也可能是一种允许实体具有更大灵活性的特征。这里作者举了一个狗为电影演员的例子。

## 5.2. Has Value or Has No Value?

三元组如何表达估计值，不确定值或空值？实际处理时用“Has Value” (HV) and “Has No Value” (HNV)来分别表达不确定值和空值。

以这种方式表达未知数和空值的有趣实现可能表明Freebase / KG最初并不是为了支持这种不确定性而建立的。Google的数据编码某些不确定性的概念并未向最终用户公开，尽管它肯定以这种独特的方式实现。

## 5.3. Dealing with Doppelgangers and Chimeras

涉及Freebase如何处理“合并”重复对象（doppelgangers）和“拆分”混合对象（嵌合体）。 

> the property “/dataworld/gardening hint/replaced by” is used to implement merges be- tween various objects (e.g. by saying “/m/xyz123 - Replaced By - /m/abc123”).

# A Small Correlational Study

主要探索这个问题：域的本体的复杂性（人物，电影等领域的类型，属性等）与表达与本体相关的事实（“知识库”）的三元组数量之间是否存在关联？

对于本研究，通过考虑与域相关的属性详细信息量（多少描述，约束等）来实现“复杂性”和“成熟度”。

对于89个域中的每一个，获得了关于每个域的本体的以下统计：

- ==**域中的类型和属性数**==
- ==**每种类型和属性的描述数**==
- ==**每种类型和属性的属性详细信息数==**

**通过获取域中每种类型和属性的平均描述数和属性详细信息来计算简单的复杂性分数。 所有域的RDF三元组计数与此复杂性得分之间的Pearson相关系数与0.2824呈正相关，简单线性回归的斜率为78,424.08（见图6）。 当排除异常音乐切片时，相关性和斜率分别变为0.6680和33,899.53。 虽然需要进一步的工作来探索这个研究问题，但这个小的相关性研究为进一步的实验提供了一些有希望的初步结果**

# discussion

考虑整体架构的三个部分：Freebase类型系统及其缺乏继承和依赖于不兼容性，允许表示值的不确定性的实现，以及合并和拆分对象的实现。此外，还进行了一项小型相关研究，以检验基于Bowker和Star推动的预感的假设。在很大程度上，分类系统中的许多特征也可以在Freebase的本体和体系结构中找到。

本文具体而言，探讨了支持整个交付流程的基础结构（本体和体系结构），而不是Freebase / KG中表示的特定事实。

# conclusion

 应通过探索Freebase本体和体系结构的其他方面以及对Freebase进行更全面的实验分析来进行进一步的研究。 

