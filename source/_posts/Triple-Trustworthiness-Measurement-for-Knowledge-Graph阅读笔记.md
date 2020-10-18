---
title: Triple Trustworthiness Measurement for Knowledge Graph阅读笔记
date: 2019-05-21 14:45:26
tags:
- Knowledge Graph
- KG
- Triple
english_title: Triple Trustworthiness Measurement for Knowledge Graph
categories:
- 论文阅读笔记
---

> 本文提出了一种通过计算triple trustworthiness来评估知识图谱的准确程度的方法。模型利用神经网络综合来自实体（借鉴Resource allocation）、关系（借鉴翻译模型的思想，如TransE）和KG全局（借鉴关系路径，RNN）三个层面的语义和全局信息，输出最后的 trustworthiness作为判断依据。
>
> [下载地址](http://delivery.acm.org/10.1145/3320000/3313586/p2865-jia.pdf?ip=59.64.129.22&id=3313586&acc=OPEN&key=BF85BBA5741FDC6E%2E66A15327C2E204FC%2E4D4702B0C3E38B35%2E6D218144511F3437&__acm__=1558515578_57e0bf75d529cf4656975ffe7da506b9)

<!-- more -->

# Summary

This paper proposed a method for estimating the accuracy of a knowledge graph by computing triple trustworthiness. The model uses neural network to synthesize semantic and global information from three levels: entity(resource allocation), relationship(translation model ideas, such as TransE)m and KG global(relationship path, RNN) and outputting the final trustworthiness as the basis for judgment.

# Problem statement

possible noises and conflicts are inevitably intoduced in the process of constructing the KG

# research objective

quantify the KG's semantic correctness and the true degree of the facts expressed

# Contribution

- Knowledge graph triple trustworthiness measurement
  - use the  triple semantic information and globally inferring information
  - three levels measurement and an intergration of confidence value
- experiment result verified the model valid on large-scale KG Freebase
- the KGTtm could be utilized in knowledge graph construction or improvement

# THE TRIPLE TRUSTWORTHINESS MEASUREMENT MODEL

![20190521155842605796518.jpg](http://image.nysdy.com/20190521155842605796518.jpg)

- Longitudinally, the model can be divided into two level.
  - the upper is a pool of multiple trustworthiness estimate cells(estimator)
  - the output of these Estimator forms the input of lower-level fusion device(Fusioner)
- Viewed laterally, three progressive levels   are be considered, as following.

## Is there a possible relationship between the entity pairs?

![20190521155842810227816.jpg](http://image.nysdy.com/20190521155842810227816.jpg)



ResourceRank:

- The algorithm assumes that the association between entity paires $(h,t)$ will be stronger, and more resource is passed from the  head $h$ through all associated paths to the tail $t$ in a graph
- The amount of resource aggregated into $t$ ingeniously indicateds the association strength from $h$ to $t$.

As pair $(e_1,e_2)$, there only one directed edge from $e_1$ to $e_2$ in the graph, where the different bandwidth of the edge indicates the number of the multiple relations.

output:
$$
\left\{\begin{array}{c}{u=\alpha\left(W_{1} V+b_{1}\right)} \\ {R R(h, t)=W_{2} u+b_{2}}\end{array}\right.
$$
Authors constructed a $V$ vector by combining six characteristics.

1. R (t | h); 
2. In-degree of head node ID(h); 
3. Out-degree of head node OD(h); 
4. In-degree of tail node ID(t);
5. Out-degree of tail node OD(t);
6. The depth from head node to tail node Dep

As for 1. the formula:
$$
R(t | h)=(1-\theta) \sum_{e_{i} \in M_{t}} \frac{R\left(e_{i} | h\right) \cdot B W_{e_{i} t}}{O D\left(e_{i}\right)}+\frac{\theta}{N}
$$

- $M_t$is the set of all nodes that have outgoing links to the node $t$, $OD (e_i)$ is the out-degree of the node eiand the $BW_{e_it}$ is the bandwidth from the $e_i$ to $t$.
- In order to improve the model fault-tolerance, we assume that the resource fow from each node may directly jump to a random node with the same probability θ

## Can the determined relationship $r$ occur between the entity pair $(h,t)$ ?

![20190522155850919251079.jpg](http://image.nysdy.com/20190522155850919251079.jpg)

Translation-based energy function (TEF)：depended on TransE

$E(h, r, t)=\|\mathbf{h}+\mathbf{r}-\mathbf{t}\|$

output:
$$
P(E(h, r, t))=\frac{1}{1+e^{-\lambda\left(\delta_{r}-E(h, r, t)\right)}}
$$

## Can the relevant triples in the KG infer that the triple is trustworthy?

![20190522155851013726520.jpg](http://image.nysdy.com/20190522155851013726520.jpg)

Reachable paths inference (RPI):

There two challenges to exploit the reachable paths for inferring triple trustworthiness:

### reachable paths selection

Semantic distance-based path selection![2019052215585105592950.jpg](http://image.nysdy.com/2019052215585105592950.jpg)

### Reachable Paths Representation

using a RNN to deal with the embeddings of the three elements of each triple in the selected path

## Fusing the Estimators

a classifer based on a multi-layer perceptron 

# EXPERIMENTS

## dataset

FB15K

## Interpreting the Validity of the Trustworthiness

![20190522155851139148623.jpg](http://image.nysdy.com/20190522155851139148623.jpg)

- The left picture shows that the positives examples are mainly concentrated in the upper region, vice versa.
- As for the right picture
  - only if the value of a triple is higher than the threshold can it be considered trustworthy
  - shows that the positive examples universally have higher confidence values

## Comparing With Other Models on The Knowledge Graph Error Detection Task

![20190522155851269267340.jpg](http://image.nysdy.com/20190522155851269267340.jpg)

Authors' model has beter results in terms of accuracy and the F1-score than the other models.

## Analyzing the ability of models to tackle the three type noises.

![20190522155851290149973.jpg](http://image.nysdy.com/20190522155851290149973.jpg)

- a higher recall shows that authors' model can more accurately find the right from noisy triples
- higher average trustworthiness values show that authors' model can better identify the correct instances and with high confidence 
- the worst among the $(h, ?, t)$, because the various relations between a certain entity  increase the difficulty of model judgment.

## Analyzing the Efects of Single Estimators

![20190522155851337652153.jpg](http://image.nysdy.com/20190522155851337652153.jpg)

It can be found that the accuracy obtained by each model is above 0.8, which proves the effectiveness of each Estimator

# 思考

本文在方法上几乎没有什么创新，本质上就是一个老方法的多个组合。最大亮点就是作者能提出trustworthiness来把这个评价知识图谱准确度的问题进行了量化。这种能力比提出方法上的创新更加厉害，也是需要学习的地方。