---
title: ERNIE Enhanced Language Representation with Informative Entities阅读笔记
date: 2019-06-24 11:10:26
tags:
- KGR
- BERT
- KG
english_title: ERNIE_Enhanced_Language_Representation_with_Informative_Entities
categories:
- 论文阅读笔记
- KG

---

> 该篇论文借鉴BERT，试图将实体信息（TransE）融入token(singal word)中，通过类似实体对齐的方法将实体与token对齐（并采取mask方式进行预训练），通过infromation fusion 将token与实体融合映射入相关联的两个向量空间。
>
> [论文下载地址](https://arxiv.org/pdf/1905.07129)

<!-- more -->

# Problem Statement

the existing pre-trained language models rarely consider incorporating knowledge graphs (KGs), which can provide rich structured knowledge facts for better language understanding.

# Challenge

For incorporating external knowledge into language representation models

- Structured Knowledge Encoding
  - regarding to the given text, how to effectively extract and encode its related informative facts in KGs for language representation models
- Heterogeneous Information Fusion
  - how to design a special pre-training objective to fuse the lexical, syntactic, and knowledge information is another challenge.

# Methodology

![20190625156142554544546.jpg](http://image.nysdy.com/20190625156142554544546.jpg)

## Model Architecture

ERNIE

- the underlying textual encoder (T-Encoder)负责从文本中捕获基本的词法和语法信息

  - $$
    \left\{\boldsymbol{w}_{1}, \ldots, \boldsymbol{w}_{n}\right\}=\mathrm{T}-\operatorname{Encoder}\left(\left\{w_{1}, \ldots, w_{n}\right\}\right)
    $$

    T-Encoder(·) is a multi-layer bidirectional Transformer encoder

- the upper knowledgeable encoder (K-Encoder)

  - entity embeddings are pre-trained by TransE负责将知识图谱集成到底层的文本信息中

## Knowledgeable Encoder

- the knowledgeable encoder K-Encoder consists of stacked aggregators
- designed for encoding both tokens and entities as well as fusing their heterogeneous features.

In the i-th aggregator

-  the input:

  - token embeddings: $\left\{\boldsymbol{w}_{1}^{(i-1)}, \ldots, \boldsymbol{w}_{n}^{(i-1)}\right\}$
  - entity embeddings :$\left\{\boldsymbol{e}_{1}^{(i-1)}, \ldots, \boldsymbol{e}_{m}^{(i-1)}\right\}$

- fed into two multi-head self-attentions(MH-ATTs)

  - $\left\{\tilde{\boldsymbol{w}}_{1}^{(i)}, \ldots, \tilde{\boldsymbol{w}}_{n}^{(i)}\right\}=\mathrm{MH}-\operatorname{ATT}\left(\left\{\boldsymbol{w}_{1}^{(i-1)}, \ldots, \boldsymbol{w}_{n}^{(i-1)}\right\}\right)$
  - $\left\{\tilde{\boldsymbol{e}}_{1}^{(i)}, \ldots, \tilde{\boldsymbol{e}}_{m}^{(i)}\right\}=\mathrm{MH}-\operatorname{ATT}\left(\left\{\boldsymbol{e}_{1}^{(i-1)}, \ldots, \boldsymbol{e}_{m}^{(i-1)}\right\}\right)$

- an information fusion layer

  - $$
    \begin{aligned} \boldsymbol{h}_{j} &=\sigma\left(\tilde{\boldsymbol{W}}_{t}^{(i)} \tilde{\boldsymbol{w}}_{j}^{(i)}+\tilde{\boldsymbol{W}}_{e}^{(i)} \tilde{\boldsymbol{e}}_{k}^{(i)}+\tilde{\boldsymbol{b}}^{(i)}\right) \\ \boldsymbol{w}_{j}^{(i)} &=\sigma\left(\boldsymbol{W}_{t}^{(i)} \boldsymbol{h}_{j}+\boldsymbol{b}_{t}^{(i)}\right) \\ \boldsymbol{e}_{k}^{(i)} &=\sigma\left(\boldsymbol{W}_{e}^{(i)} \boldsymbol{h}_{j}+\boldsymbol{b}_{e}^{(i)}\right) \end{aligned}
    $$

  $h_j$ is the inner hidden state

For the tokens without corresponding entities
$$
\begin{aligned} \boldsymbol{h}_{j} &=\sigma\left(\tilde{\boldsymbol{W}}_{t}^{(i)} \tilde{\boldsymbol{w}}_{j}^{(i)}+\tilde{\boldsymbol{b}}^{(i)}\right) \\ \boldsymbol{w}_{j}^{(i)} &=\sigma\left(\boldsymbol{W}_{t}^{(i)} \boldsymbol{h}_{j}+\boldsymbol{b}_{t}^{(i)}\right) \end{aligned}
$$

## Pre-training for Injecting Knowledge

In order to inject knowledge into language rep- resentation by informative entities.

Randomly masks some token-entity alignments and then requires the system to predict all corresponding entities based on aligned tokens.

- denoising entity auto-encoder (dEA)

- define the aligned entity distribution for the token $w_i$ as follows:
  $$
  p\left(e_{j} | w_{i}\right)=\frac{\exp \left(\text { linear }\left(\boldsymbol{w}_{i}^{o}\right) \cdot \boldsymbol{e}_{j}\right)}{\sum_{k=1}^{m} \exp \left(\text { linear }\left(\boldsymbol{w}_{i}^{o}\right) \cdot \boldsymbol{e}_{k}\right)}
  $$
  

  linear(·) is a linear layer

For dEA, perform the following operations:

- in 5% of the time, replace the entity with another random
  - aims to train model to correct the errors that the token is aligned with a wrong entity;
- In 15% of the time, mask token-entity alignments
  - aims to train model to correct the errors that entity alignment system doesn’t extract all existing alignments;
- in the rest of the time, keep token-entity alignments unchanged 
  - aims to encourage our model to integrate the entity information into token representations for better language understanding.

## Fine-tuning for Specific Tasks

![20190625156143137852366.jpg](http://image.nysdy.com/20190625156143137852366.jpg)

We can take the final output embedding for the first token, which corresponds to the special [CLS] token, as the representation of the input sequence for specific tasks.

For some knowledge-driven tasks, we design special fine-tuning procedure:

- relation classification
  - design different tokens [HD] and [TL] for head entities and tail entities respectively
  -  a similar role like position embeddings in the conventional relation classification models (Zeng et al., 2015)
- entity typing
  - the mention mark token [ENT]

> - 这里的CLS不知道有什么作用，所有的任务都有，是不同的任务重CLS的embedding有所不同吗？个人目前觉得是这样的。
> - 作者这里采用的mark token的方法代替position embedding，不知道两个对比那种效果会更好一些。直观觉得都是标记位置信息。

# Experiments

### Pre-training Dataset

- we use English Wikipedia as our pre-training corpus and align text to Wiki-data
  - 4, 500M subwords and 140M entities, and discards the sentences having less than 3 entities
- before pre-training ERINE, entity embeddings by TransE
  - sample part of Wikidata which contains 5, 040, 986 entities and 24, 267, 796 fact triples

### Training Details

- We also fine-tune ERNIE on the distant supervised dataset, i.e., FIGER (Ling et al., 2015)
- we use TAGME (Ferragina and Scaiella, 2010) to extract the entity mentions in the sentences and link them to their corresponding entities in KGs

## Entity Typing

### dataset

two well-established datasets FIGER (Ling et al., 2015) and Open Entity (Choi et al., 2018).

- The training set of FIGER is labeled with distant supervision, and its test set is annotated by human.
- Open Entity is a completely manually-annotated dataset.

![20190625156143360958681.jpg](http://image.nysdy.com/20190625156143360958681.jpg)

### Comparble model

- NFGEC
  - NFGEC is a hybrid model proposed by Shimaoka et al. (2016)
- UFET
  - (Choi et al., 2018)

#### The results on FIGER:

However, BERT has lower accuracy than the best NFGEC model. As strict accuracy is the ratio of instances whose predictions are identical to human annotations, it illustrates **some wrong labels from distant supervision are learned by BERT** due to its powerful fitting ability.

## Relation Classification

### dataset

two well-established datasets FewRel (Han et al., 2018b) and TACRED (Zhang et al., 2017).

- FewRel
  - As FewRel does not have any null instance where there isn’t any relation between entities, we adopt macro averaged metrics to present the model performances. Since FewRel is built by checking whether the sentences contain facts in Wiki-data, we drop the related facts in KGs before pretraining for fair comparison
- TACRED
  - In TACRED, there are nearly 80% null instances so that we follow the previous work (Zhang et al., 2017) to adopt micro averaged metrics to represent the model performances instead of the macro

![20190625156143680247028.jpg](http://image.nysdy.com/20190625156143680247028.jpg)

### Comparble model

- CNN:(Zeng et al., 2015).
- PA-LSTM
- C-GCN :Zhang et al. (2018) adopt the graph convolution operations to model dependency trees for relation classification.<Graph convolution over pruned dependency trees improves relation extraction.>

## GLUE

The General Language Understanding Evaluation (GLUE) benchmark (Wang et al., 2018) is a collection of diverse natural language understanding tasks

## Ablation Study

explore the effects of the informative entities and the knowledgeable pretraining task (dEA) for ERNIE using FewRel dataset

> 实验部分做的很丰富，既有两个任务的对比实验，也有对自身模块的对比实验，并且还对比了bert来检测自己模型是否对GLUE任务效果有降低。

# future research

1) inject knowledge into feature-based pre-training models such as ELMo (Peters et al., 2018); 

(2) introduce diverse structured knowledge into language representation models such as ConceptNet (Speer and Havasi, 2012) which is different from world knowledge database Wikidata; 

(3) annotate more real-world corpora heuristically for larger pre-training data

> 

# 参考链接

- https://blog.csdn.net/summerhmh/article/details/91042273