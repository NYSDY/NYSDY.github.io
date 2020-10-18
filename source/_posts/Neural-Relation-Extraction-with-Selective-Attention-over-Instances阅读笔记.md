---
title: Neural Relation Extraction with Selective Attention over Instances阅读笔记
date: 2019-06-26 13:49:57
tags:
- RE
english_title: Neural_Relation_Extraction_with_Selective_Attention_over_Instances
categories:
- 论文阅读笔记
---

> 这篇文章之前看过😂。
>
> 论下载地址

<!-- more -->

# Problem Statement

​	Distant supervision inevitably accompanies with the wrong labelling problem, and thse noisy data will substantially hurt the performance of relation extraction.

# Contribution

- As compared to existing neural relation extraction model, our model can make full use of all informative sentences of each entity pair.
- To address the wrong labelling problem in distant supervision, we propose selective attention to de-emphasize those noisy instances.
- In the experiments, we show that selective attention is beneficial to two kinds of CNN models in the task of relation extraction.

# Methodology

模型整体架构如下所示：

![20190626156153649268323.jpg](http://image.nysdy.com/20190626156153649268323.jpg)

