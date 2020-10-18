---
title: DocRED A Large-Scale Document-Level Relation Extraction Dataset阅读笔记
date: 2019-07-01 09:11:49
tags:
- dataset
- RE
english_title: DocRED_A_Large-Scale_Document-Level_Relation_Extraction_Dataset
categories:
- 论文阅读笔记
---



> 这是一个介绍数据集的论文，主要是文档级别的关系抽取数据集。
>
> [论文下载地址](http://arxiv.org/abs/1906.06127)

# Problem Statement

existing datasets for document-level RE 

- either only have a small number of manually-annotated relations and entities, 
- or exhibit noisy annotations from distant supervision, 
- or serve specific domains or approaches.

# Contribution (DocRED)

- constructed from Wikipedia and Wikidata
- DocRED contains 132, 375 entities and 56, 354 relational facts annotated on 5, 053 Wikipedia documents
- As at least 40.7% of the relational facts in DocRED can only be extracted from multiple sentences
- also provide large-scale distantly supervised data to support weakly supervised RE research

- indicate the existing methods deal with the taks document level RE is  more difficult sentence-level RE.

# data

![20190701156194521958350.jpg](http://image.nysdy.com/20190701156194521958350.jpg)