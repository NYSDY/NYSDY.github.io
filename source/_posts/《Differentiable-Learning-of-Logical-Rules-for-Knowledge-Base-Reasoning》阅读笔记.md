---
title: 《Differentiable Learning of Logical Rules for Knowledge Base Reasoning》阅读笔记
date: 2019-03-05 15:34:05
tags:
- 知识图谱
- 知识图谱推理
- 规则学习
english_title: Differentiable Learning of Logical Rules for Knowledge Base Reasoning
categories:
- 论文阅读笔记

---

> 论文[下载地址](http://papers.nips.cc/paper/6826-differentiable-learning-of-logical-rules-for-knowledge-base-reasoning.pdf)，本文研究用于于知识图谱推理的学习概率一阶逻辑规则的问题，提出了Neural Logic Programming（Neural-LP）框架，它结合了端到端可微分模型中一阶逻辑规则的参数和结构学习。为了在可微分的框架中同时学习参数和结构，作者设计了一个具有注意机制和记忆的神经控制器系统，以学习顺序组成TensorLog使用的原始可微操作。作者采用的注意机制是作为逻辑规则的置信度并且有寓意含义的。

<!-- more -->

下图展示了一个使用逻辑规则进行知识图谱推理的例子

![](https://i.loli.net/2019/03/05/5c7dc90edacd1.jpg)

使用概率逻辑的优点是通过为逻辑规则配备概率，可以更好地模拟统计复杂和噪声数据。

statistical relational learning（统计关系学习）：学习关系规则的集合

==inductive logic programming（归纳逻辑规划）：==当学习涉及提出新的逻辑规则时。（这应该和我正在做的方向是相关的，都是带有归纳性质的，有新的东西产生）。

# Framework

## Knowledge base reasoning

为了推理知识库，对于每个查询我们都有兴趣学习以下形式的加权链式逻辑规则，类似于**==随机逻辑程序==**：

![](https://i.loli.net/2019/03/05/5c7dcd6e1a5ac.jpg)

其中$\alpha$是和规则有关的置信度，R是知识库中的关系，query(Y,X) 表示一个三元组，query 表示一个关系。

## TensorLog for KB reasoning

将实体转换成one-hot变量；并用一个矩阵$M_R$表示关系，该矩阵只在（i，j）处为1，i、j为第i、j个实体。

结合两个操作，逻辑规则推理$R(Y,X) \gets P(Y,X) \bigwedge Q(Z,X)$可以被表示为：$M_P \cdot M_P \cdot v_x \doteq s$，向量s中为1的位置就是Y的答案。

对于一条查询，所有的逻辑规则的右边部分被表示为以下形式：

![](https://i.loli.net/2019/03/05/5c7dda071bcef.jpg)

其中，l表示所有的可能规则的个数，$\alpha_l$是规则l的置信度，$\beta_l$是某特定关系里的有序关系列表，所以在inference时，给定实体$v_x​$，实体y的score等于向量s中的对应y的位置的值。对于推理，给定实体x，实体y的score等于向量s中的对应y的位置的值。

![](https://i.loli.net/2019/03/10/5c85104e2a53b.jpg)

所以总结本文关心的优化问题如下：

![](https://i.loli.net/2019/03/05/5c7e1568d9ec7.jpg)

## Learning the logical rules

在上式的优化问题中，算法需要学习的部分分为两个：一个是规则的结构，即一个规则是由哪些条件组合而成的；另一个是规则的置信度。由于每一条规则的置信度都是依赖于具体的规则形式，而规则结构的组成也是一个离散化的过程，因此上式整体是不可微的。因此作者对前面的式子做了以下更改：![](https://i.loli.net/2019/03/05/5c7e160404436.jpg)

对比与式（2）：主要交换了连乘和累加的计算顺序，对预一个关系的相关的规则，为每个关系在每个步骤都学习了一个权重，即上式的 $a_t^k$。

由于上式固定了每个规则的长度都为 T，这显然是不合适的。为了能够学习到变长的规则，Neural LP中设计了记忆向量 $u_t$,表示每个步骤输出的答案--每个实体作为答案的概率分布，还设计了两个注意力向量：一个为记忆注意力向量 $b_t$ ——表示在步骤 t 时对于之前每个步骤的注意力；一个为算子注意力向量 $a_t$ ——表示在步骤 t 时对于每个关系算子的注意力。每个步骤的输出由下面三个式子生成：![](https://i.loli.net/2019/03/05/5c7e18ac09662.jpg)

其中$b_t$和$a_t$由以下公式通过RNN获得：

![](https://i.loli.net/2019/03/05/5c7e1a015f0ac.jpg)

推理机的整体框架是：

![](https://i.loli.net/2019/03/05/5c7e1aec1aea9.jpg)

其中memory存的就是每步的推理结果（实体），最后的输出（例如$u_{T+1}$，目标就是最大化 $logv_y^Tu$，加log是因为非线性能让效果变好。

整个算法如下：![](https://i.loli.net/2019/03/05/5c7e1c0d5cf32.jpg)

# 实验

## （1） 两个标准数据集上的统计关系学习相关的实验

- Unified Medical Language System (UMLS)：The entities are biomedical concepts (e.g. disease, antibiotic) and relations are like treats and diagnoses.
- Kinship：contains kinship relationships among members of the Alyawarra tribe from Central Australia [

![](https://i.loli.net/2019/03/05/5c7e2270289d0.jpg)

## （2） 在$16*16$的网格上的路径寻找的实验

![](https://i.loli.net/2019/03/05/5c7e235b430cb.jpg)

## （3） 知识库补全实验

实验所用数据集信息：

FB15KSelected：这是通过从FB15K中去除近似重复和反向关系而构造的

![](https://i.loli.net/2019/03/05/5c7e2378e1c64.jpg)

实验结果：![](https://i.loli.net/2019/03/05/5c7e23c43db48.jpg)

为了证明Neural LP的归纳推理的能力，本文还特别设计了一个实验，在训练数据集中去掉所有涉及测试集中包含的实体的三元组，然后训练并预测，得到结果如下：![](https://i.loli.net/2019/03/05/5c7e23ebbca69.jpg)

## （4） 知识库问答的实验

![](https://i.loli.net/2019/03/05/5c7e24aa6a427.jpg)

# **总结**

本文提出了一个可微的规则学习模型，并强调了知识库中的规则应该是实体无关的，对于我目前在做的方向，本体论也是与实体无关的，这种规则学习有一定的借鉴性，但是好像所区别。这个规则推理也可以看成某些关系之间的包含关系3.1中举的HasOfficeInCity(New York,Uber) and CityInCountry(USA,New York)的例子，可以看作是2对于1有包含关系。并且可以看到本篇论文中，作者设计了丰富的实验。

# 参考链接

- https://toutiao.io/posts/wrxf4z/preview
- https://zhuanlan.zhihu.com/p/46024825

