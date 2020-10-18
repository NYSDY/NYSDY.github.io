---
title: >-
  Universal Representation Learning of Knowledge Bases by Jointly Embedding
  Instances and Ontological Concepts阅读笔记
date: 2019-07-17 16:45:52
tags:
- KGE
- ontology
english_title: Universal_Representation_Learning_of_Knowledge_Bases_by_Jointly_Embedding_Instances_and_Ontological_Concepts
categories:
- 论文阅读笔记
- KG
---

> 
>
> [论文下载地址](http://web.cs.ucla.edu/~yzsun/papers/2019_KDD_JOIE.pdf)

<!-- more -->

# Problem Statement

Existing KG embedding models merely focus on representing of an ontology view for abstract and commonsense concepts or an instance view for special entities that are instantiated from ontological concepts. 



# Challenge

- **mappings difficult** :the semantic mappings from entities to concepts and from relations to meta-relations are complicated and difficult to be precisely captured by any current embedding models
- **inadequate cross-view links:** the known cross-view links inadequately cover a vast number of entities, which leads to insufficient information to align both views of the KB, and curtails discovering new cross-view links
- the scales and topological structures are different in ontological views and instance views

# Introduction

![20190722156378173495213.jpg](http://image.nysdy.com/20190722156378173495213.jpg)

从两种视图来学习表示有以下两点好处：

- instance embeddings provide detailed and rich information for their corresponding ontological concepts.
- a concept embedding provides a high-level summary of its instances, which is extremely helpful when an instance is rarely observed.

# contribution 

- a novel KG embedding model named JOIE, which jointly encodes both the ontology and instance views of a KB
  - cross-view association model : a novel KG embedding model named JOIE, which jointly encodes both the ontology and instance views of a KB
    - cross-view grouping technique : assumes that the two views can be forced into the same embedding space
    - cross-view transformation technique : enables non-linear transformations from the instance embedding space to the ontology embedding space
  - intra-view embedding model : characterizes the relational facts of ontology and instance views in two separate embedding spaces
    - three state-of-the-art translational or similarity-based relational embedding techniques
    - hierarchy-aware embedding: based on intra-view non- linear transformations to preserve ontologies hierarchical substructures.
- implement two experiments:
  - the triple completion task : confirm the effectiveness of JOIE for populating knowledge in both ontology and instance-view KGs, which has significantly outperformed various baseline models.
  - the entity typing task :  show that JOIE is competent in discovering cross-view links to align the ontology-view and the instance-view KGs.

## Modeling

![2019072215637837731893.jpg](http://image.nysdy.com/2019072215637837731893.jpg)

## Cross-view Association Model

![20190722156378594486796.png](http://image.nysdy.com/20190722156378594486796.png)

### Cross-view Grouping (CG)

该模型可以视为grouping-based regularization， 假设本体视图KG和实例视图KG可以被嵌入到同一空间中，并强制使实例向量靠近与它相关联的概念向量，如图3(a)所示

定义损失函数如下：


$$
J_{\text { Cross }}^{\mathrm{CG}}=\frac{1}{|\mathcal{S}|} \sum_{(e, c) \in \mathcal{S}}\left[\|\mathbf{c}-\mathbf{e}\|_{2}-\gamma^{\mathrm{CG}}\right]_{+}
$$

### Cross-view Transformation (CT)

试图在实体嵌入空间和概念空间之间转换信息，如图3(b)所示

定义映射函数，将实例映射到本体视图空间，该映射后向量应该靠近它的相关联概念：
$$
\mathbf{c} \leftarrow f_{\mathrm{CT}}(\mathbf{e}), \forall(e, c) \in \mathcal{S}
$$
其中，
$$
f_{\mathrm{CT}}(\mathbf{e})=\sigma\left(\mathbf{W}_{\mathrm{ct}} \cdot \mathbf{e}+\mathbf{b}_{\mathrm{ct}}\right)
$$
整个CT的损失函数为：
$$
J_{\text { Cross }}^{\mathrm{CT}}=\frac{1}{|\mathcal{S}|} \sum_{(e, c) \in \mathcal{S} \atop \wedge\left(e, c^{\prime}\right) \in \mathcal{S}}\left[\gamma^{\mathrm{CT}}+\left\|\mathbf{c}-f_{\mathrm{CT}}(\mathbf{e})\right\|_{2}-\left\|\mathbf{c}^{\prime}-f_{\mathrm{CT}}(\mathbf{e})\right\|_{2}\right]_{+}
$$

## Intra-view Model

该模型的目的：在两个嵌入空间中分别保留KB的每个视图中的原始结构信息。

### Default Intra-view Model

作者采用三种方式：
$$
\begin{aligned} f_{\text { TransE }}(\mathbf{h}, \mathbf{r}, \mathbf{t}) &=-\|\mathbf{h}+\mathbf{r}-\mathbf{t}\|_{2} \\ f_{\text { Mult }}(\mathbf{h}, \mathbf{r}, \mathbf{t}) &=(\mathbf{h} \circ \mathbf{t}) \cdot \mathbf{r} \\ f_{\text { HolE }}(\mathbf{h}, \mathbf{r}, \mathbf{t}) &=(\mathbf{h} \star \mathbf{t}) \cdot \mathbf{r} \end{aligned}
$$
损失函数：
$$
J_{\text { Intra }}^{G}=\frac{1}{|\mathcal{G}|} \sum_{(h, r, t) \in \mathcal{G}}\left[\gamma^{\mathcal{G}}+f\left(\mathbf{h}^{\prime}, \mathbf{r}, \mathbf{t}^{\prime}\right)-f(\mathbf{h}, \mathbf{r}, \mathbf{t})\right]_{+}
$$
intra损失函数：
$$
J_{\text { Intra }}=J_{\text { Intra }}^{\mathcal{G}_{I}}+\alpha_{1} \cdot J_{\text { Intra }}^{\mathcal{G}_{O}}
$$

### Hierarchy-Aware Intra-view Model for the Ontology

进一步区分了构成本体层次结构的元关系和视图内模型中的规则语义关系(如“related_to”)。

给定概念对（cl，ch），我们将这种层次结构建模为粗略概念和相关更精细概念之间的非线性变换:
$$
g_{\mathrm{HA}}\left(\mathbf{c}_{h}\right)=\sigma\left(\mathbf{W}_{\mathrm{HA}} \cdot \mathbf{c}_{l}+\mathbf{b}_{\mathrm{HA}}\right)
$$
损失函数为：
$$
J_{\text { Intra }}^{\mathrm{HA}}=\frac{1}{|\mathcal{T}|} \sum_{\left(c_{l}, c_{h}\right) \in \mathcal{T}}\left[\gamma^{\mathrm{HA}}+\left\|\mathbf{c}_{h}-g\left(\mathbf{c}_{l}\right)\right\|_{2}-\left\|\mathbf{c}_{\mathrm{h}}^{\prime}-g\left(\mathbf{c}_{1}\right)\right\|_{2}\right]_{+}
$$
故，该部分损失函数为：
$$
J_{\text { Intra }}=J_{\text { Intra }}^{G_{I}}+\alpha_{1} \cdot J_{\text { Intra }}^{\mathcal{G} o \backslash \mathcal{T}}+\alpha_{2} \cdot J_{\text { Intra }}^{\mathrm{HA}}
$$

- $J_{\text { Intra }}^{\mathcal{G} o} \backslash \mathcal{T}$: 默认的视图内模型的丢失，该模型仅在具有规则语义关系的三元组上训练
- $J_{\text { Intra }}^{\mathrm{HA}}$明确训练三元组与形成本体层次结构的元关系

> 感觉这部分就是传递关系，类似推理性质的。
>
> 没明白两种ontology关系的区分点

## Joint Training on Two-View KBs

联合损失函数：
$$
J=J_{\text { Intra }}+\omega \cdot J_{\text { Cross }}
$$
作者并不直接更新$J$，而是交替更新$J_{\text { Intra }}^{\mathcal{G}_{I}}, J_{\text { Intra }}^{\mathcal{G} O} \text { and } J_{\text { Cross }}$.

# EXPERIMENTS

具体细节直接见论文

## dataset

![20190722156379991044398.png](http://image.nysdy.com/20190722156379991044398.png)数据集是作者自己构建的，信息如上图所示。

## Case Study

### Ontology Population

作者想预测在元关系词表中并不存在的元关系，例如：预测(“Office Holder”, ?r, “Country”)

这里，作者采取的方式是将概念通过之前提到的实体空间到概念空间的映射来进行反映射。然后按照$f_{\mathrm{CT}}^{\mathrm{inv}}\left(\mathbf{c}_{\text { country }}\right)-f_{\mathrm{CT}}^{\mathrm{inv}}\left(\mathbf{c}_{\text { office }}\right)$来在实体嵌入空间进行搜索与之相近的实体间关系。

![20190722156380029962081.png](http://image.nysdy.com/20190722156380029962081.png)

### Long-tail entity typing

![2019072215638007155223.png](http://image.nysdy.com/2019072215638007155223.png)

In KGs, the frequency of entities and relations often follow a long-tail distribution (Zipf’s law)

作者抽取了低频次实体进行了训练，发现JOIE模型的效果虽然有下降，但尚在可以接受的程度内。

# FUTURE WORK

- Particularly, instead of optimizing structure loss with triples (first-order neighborhood) locally, we plan to adopt more complex embedding models which leverage information from higher order neighborhood, logic paths or even global knowledge graph structures. 
- We also plan to explore the alignment on relations and meta-relations like entity-concept.
- exploring different triple encoding techniques
- Note that we are also aware of the fact that there are more comprehensive properties of relations and meta-relations in the two views such as logical rules of relations and entity types. Incorporating such properties into the learning process is left as future work.

# 思考

这篇论文和之前跟张老师定的我的论文的思路基本一致，额，有点感觉有点受打击。这篇文章也是该作者博士毕业论文中的一部分，所以应该是这个作者早就有这个思路了，所以也没什么好纠结的。这篇文章也是走的trans的路线，和刘的论文又不一样的思路，但是都是概念本体这类的。其中有一点不一样，就是is_a关系可能两篇论文用的不一样。这篇论文中提到了数据集开源，可是github的链接中并没有数据集。虽然轮文中说他结合了概念和实例的视图，但是其实像刘的论文就已经提出结合了概念和实例的角度了。