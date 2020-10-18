---
title: Graph Neural Networks with Generated Parameters for Relation Extraction阅读笔记
date: 2019-05-23 10:41:51
tags:
- GNNs
- relation extraction
- relation reasoning
english_title: Graph_Neural_Networks_with_Generated_Parameters_for_Relation
categories:
- 论文阅读笔记

---

> 本文将GNNs应用到处理非结构化文本的（多跳）关系推理任务来进行关系抽取。采用从句子序列中获取的实体构建全链接图，应用编码（sequence model），传播（节点间信息）和分类（预测）三个模块来处理关系推理。本文提供了三个数据集。

<!-- more -->

# problem statement

- existing relation extraction models fail to infer the relationship without multi-hop relational reasoning.
- existing GNNs can't process multi-hop relational reasoning in natural language relational reasoning 

# research objective

enable GNNs to porcess relational reasoning on unstructed text inputs

# contribution

- extend a GNN with generated parameters, which could be applied to process relational reasoning on unstructured inputs
- verify GP-GNNs in the taks of relation extraction from text; present three datasets

# GP-GNNs

- construct a fully-connected graph with the entities in the sequence of text
- employs three models to process relational reasoning
  - an encoding modul: enable edges to encode rich information from natural languages 
  - a propagation modul: propagates realtional information among various nodes 
  - a classification modul: make prediction with node representations 

As compared to tradtional GNNs, GP-GNNs could learn edges' parameters from natural lanuages

# Related work

## Graph Neural Networks(GNNs)

- existing models still perfom message-passing on predefined graphs
- *Learning Graphical State Transitions* is most related
  - introduecs a nove lnerual architecture to generate a graph based on the textal input
  - dynamically update the relationship during the learning process

## relational reasoning

- existing models could not make full use of the multi-hop inference patterns among multiple entity pair and their relaitons within the sentence 
- *LEARNING GRAPHICAL STATE TRANSITIONS* is the most related work
  - the proposed model incorporates contextual relations with attention mechanism when predicting the relation of a target entity pair

# Graph Neural Network with Grenerated Parameters(GP-GNNs)

The picture is overall architecture: encoding module, propagation module and classification module

![20190523155858968594021.jpg](http://image.nysdy.com/20190523155858968594021.jpg)

## Encoding Module

formula:
$$
\mathcal{A}_{i, j}^{(n)}=f\left(E\left(x_{0}^{i, j}\right), E\left(x_{1}^{i, j}\right), \cdots, E\left(x_{l-1}^{i, j}\right) ; \theta_{e}^{n}\right)
$$
where $f(\cdot)$ could be any model that could sequential(such as LSTMs); $E(\cdot)$ indicates an embedding function. $x^{i, j}$ is the word in sentence labeled( $i,j$)

## Porpagation Module

the representations of layer n + 1 are calculated by:
$$
\mathbf{h}_{i}^{(n+1)}=\sum_{v_{j} \in \mathcal{N}\left(v_{i}\right)} \sigma\left(\mathcal{A}_{i, j}^{(n)} \mathbf{h}_{j}^{(n)}\right)
$$
where $\mathcal{N}\left(v_{i}\right)$ denotes the neighbors of node $v_i$

## Classification Module

the loss of GP-GNNs:
$$
\mathcal{L}=g\left(\mathbf{h}_{0 :|\mathcal{V}|-1}^{0}, \mathbf{h}_{0 :|\mathcal{V}|-1}^{1}, \ldots, \mathbf{h}_{0 :|\mathcal{V}|-1}^{K}, Y ; \theta_{c}\right)
$$

# Relation Extraction with GP-GNNs

Authors introduce how to apply GP-GNNs to relation extraction

## Encoding Module

encoding then context of entity pairs (or edges in the graph)
$$
E\left(x_{t}^{i, j}\right)=\left[\boldsymbol{x}_{t} ; \boldsymbol{p}_{t}^{i, j}\right]
$$
where $x_t$ denotes the word embedding; $\boldsymbol{p}_{t}^{i, j}$denotes the position embedding of word posistion t relative to the entity pair's position $i, j$.

### position embedding

we mark each token in the sentence as either belonging to the first entity $v_i$, the second entity $v_j$ or to neither of those

## Propagation Module 

 the formula is the same as the front

### The Initial Embeddings of Nodes 

- when extracting the relationship between entity $v_i$ and entity $v_j$, the initial embeddings of them are annotated as $\mathbf{h}_{v_{i}}^{(0)}=a_{\text { subject }}$, and $h_{v_{j}}^{(0)}=a_{\text { object }}$, while the intial embeddings of other entities are set to all zeros.
- In our experiments, we generalize the idea of Gated Graph Neural Networks (Li et al., 2016) by setting $a_{\text { subject }}=[1 ; 0]^{\top}$and $a_{\text { object }}=[0 ; 1]^{\top}$.

## classification Module

**As the  target entity pair $(v_i, v_j)$:**
$$
\boldsymbol{r}_{v_{i}, v_{j}}=\left[\left[\boldsymbol{h}_{v_{i}}^{(1)} \odot \boldsymbol{h}_{v_{j}}^{(1)}\right]^{\top} ;\left[\boldsymbol{h}_{v_{i}}^{(2)} \odot \boldsymbol{h}_{v_{j}}^{(2)}\right]^{\top} ; \ldots ;\left[\boldsymbol{h}_{v_{i}}^{(K)} \odot \boldsymbol{h}_{v_{j}}^{(K)}\right]^{\top}\right]
$$
where $\odot$ represents element-wise multiplication

**classification:**
$$
\mathbb{P}\left(r_{v_{i}, v_{j}} | h, t, s\right)=\operatorname{softmax}\left(M L P\left(\boldsymbol{r}_{v_{i}, v_{j}}\right)\right)
$$
**loss:**
$$
\mathcal{L}=\sum_{s \in S} \sum_{i \neq j} \log \mathbb{P}\left(r_{v_{i}, v_{j}} | i, j, s\right)
$$

# Experiments

## aim

- showing their best models could improve the performance of relation extraction under a variety of settings
- illlustrating that how the number of layers affect the performance of their model
- performing a qualitiative investigation to highlight the diference between their models and baseline models

## design 

as the first and second aim

- show that our models could improve instance-level relation extraction on a human annotated test set
- we will show that our models could also help enhance the performance of bag-level relation extraction on a distantly labeled test set
- split a subset of distantly labeled test set, where the number of entities and edges is large

## Dataset

### distantly label set 

- Sorokin and Gurevych (2017) proposed 
- modify their dataset
  - added reversed edges
  - for all of the entity pairs with no relations, added "NA" labels to them

### Human annotated test set

- Sorokin and Gurevych (2017)
- select the distantly lablel pairs which all 5 annotaters are accepted.
- There are 350 sentences and 1,230 triples in this test set 

### Dense distantly labeled test set

- criteria
  - the number of entities should be strictly larger than 2
  - there must be at least one circle (with at least three entities) in the ground-truth label of the sentence
- There are 1,350 sentences and more than 17,915 triples and 7,906 relational facts in this test set.

## Models for comparison

- Context-aware RE
- Multi-Window CNN
- PCNN
- LSTM or GP-GNN with K = 1 layer
- GP-GNN with K = 2 or K = 3 layerss

## Evaluation Details

**To evaluation models in bag-level:**
$$
E\left(r | v_{i}, v_{j}, S\right)=\max _{s \in S} \mathbb{P}\left(r_{v_{i}, v_{j}} | i, j, s\right)
$$
**result**:

![20190523155859369893411.jpg](http://image.nysdy.com/20190523155859369893411.jpg)

## Effectiveness of Reasoning Mechanism

![2019052315585938129506.jpg](http://image.nysdy.com/2019052315585938129506.jpg)

- Context-Aware RE may **introduce more noise,** for it may mistakenly increase the probability of a relation with the similar topic with the context relations
- sentences from Wikipedia corpus are always complex, which may be hard to model for CNN and PCNN

## The Effectiveness of the Number of Layers

![20190523155859455443298.jpg](http://image.nysdy.com/20190523155859455443298.jpg)

- the improvement of the third layer is much smaller on the overall distantly supervised test set than the one on the dense subset
  - This observation reveals that the reasoning mechanism could help us identify relations especially on sentences where there are more entities
- as the number of layers grows, the curves get higher and higher precision, 
  - indicating considering more hops in reasoning leads to better performance

## Qualitative Results: Case Study

![20190523155859457710223.jpg](http://image.nysdy.com/20190523155859457710223.jpg)

Context-Aware RE makes a mistake by predicting (Kentucky, share boarder with, Ohio). As we have discussed before, this is due to its mechanism to model co-occurrence of multiple relations

# 思考

文章是刘知远组的论文，针对的方向是关系抽取，在其中结合了关系推理，最近许多任务都在结合推理的思想。文章整体的结构，逻辑十分清晰，论述的也比较详细，属于标准论文。感觉文章中GP-GNNs结构图还可以画的更好一点，展现一下encoding module的层，可以更好理解。文章的精髓应该是这个propagation module的部分，还需要消化一下，不过这部分可能是有先前的知识支撑的。