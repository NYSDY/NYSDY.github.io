---
title: '2016 TransG : A Generative Model for Knowledge Graph Embedding阅读笔记'
copyright: true
top: false
date: 2019-10-12 10:03:44
password:
english_title: TransG_:_A_Generative_Model_for_Knowledge_Graph_Embedding
tags:
- 2016
- ACL
- KGE
categories:
- 论文阅读笔记
---

> [论文下载地址](https://www.aclweb.org/anthology/P16-1219.pdf)

# 解决问题

multiple relation semantics（多重关系语义）：一个关系可能具有与对应的三元组关联的实体对揭示的多种含义。

![](http://image.nysdy.com/20191015092730.png)

这里以TransE的可视化为例，表明：特定关系有不同的聚类，并且不同的聚类表示不同的潜在语义，证实了该问题的存在性。

### 该现象产生原因

- 人为简化
  - 知识库策展人不能涉及太多相似关系，因此将多个相似关系抽象为一个特定关系是一种常见的技巧
- 知识性质
  - 语言和知识表示形式常常涉及不明确的信息。 知识的模糊性意味着语义上的混合

# TransG

利用贝叶斯非参数无限混合模型通过为关系生成多个翻译组件来处理多个关系语义