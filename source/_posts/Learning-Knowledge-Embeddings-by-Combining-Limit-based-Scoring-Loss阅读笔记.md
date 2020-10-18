---
title: Learning Knowledge Embeddings by Combining Limit-based Scoring Loss阅读笔记
date: 2019-06-03 10:57:20
tags:
- margin loss
- transE
- transH
- KG
english_title: Learning_Knowledge_Embeddings_by_Combining_Limit-based_Scoring_Loss
categories:
- 论文阅读笔记
- KG
---

> 此篇文章最为重要的就是作者设计的 margin-based ranking loss 的改进，对两个超参数$\lambda$和$\gamma$的实验，对于实验结果有很多值得分析与思考的地方。
>
> [论文下载地址](https://dl.acm.org/ft_gateway.cfm?id=3132939&ftid=1920664&dwn=1&CFID=135630312&CFTOKEN=71536805165d7c9d-10D2074A-AD9B-A596-5CA31DB63C36A322)

<!-- more -->

# Problem Statement

The margin-based ranking loss function cannot ensure the fact that the scoring of correct triplets must be low enough to fulfill the translation.

# research objective

reduce the scoring of correct triplets to fulfill the translation by mending the margin-based ranking loss function

# Contributions

- proposing a limit-based ranking loss item combined with margin-based ranking loss 
- extending TransE and TransH to TransE-RS and TransH-RS

# Model

## Margin-based Tanking Loss

formula:
$$
L_{R}=\sum_{(h, r, t) \in \Delta} \sum_{\left(h^{\prime}, r, t^{\prime}\right) \in \Delta^{\prime}}\left[\gamma_{1}+f_{r}(h, t)-f_{r}\left(h^{\prime}, t^{\prime}\right)\right) ]_{+}
$$

- The margin-based ranking loss function aims to make the score $f_{r}\left(h^{\prime}, t^{\prime}\right)$ of corrupted triplet higher by at least $\gamma_{1}$ than  of positive triplet.

- cannot be proved $f_{r}(h, t)<\varepsilon$ 

## Limit-based Scoring Loss

formula:
$$
L_{S}=\sum_{(h, r, t) \in \Delta}\left[f_{r}(h, t)-\gamma_{2}\right]_{+}
$$

## Finally loss

formula:
$$
L_{R S}=L_{R}+\lambda L_{S}, \quad(\lambda>0)
$$
detail is :
$$
\begin{array}{c}{L_{R S}=\sum_{(h, r, t) \in \Delta} \sum_{\left(h^{\prime}, r, t^{\prime}\right) \in \Delta^{\prime}}\left\{\left[\gamma_{1}+f_{r}(h, t)-f_{r}\left(h^{\prime}, t^{\prime}\right)\right]_{+}\right.} \\ {+\lambda\left[f_{r}(h, t)-\gamma_{2}\right]_{+} \}}\end{array}
$$

# Experiments

## dataset

![20190603155952634332494.jpg](http://image.nysdy.com/20190603155952634332494.jpg)

## Link prediction

> 思考
>
> 作者只是对表格的数据进行了陈述，有一些问题并没有进行分析解释
>
> - 并没有分析比如说为什么改进loss后的transE为什么会比TransH（R、D）效果要好？
> - 为什么在n-to-1中的表现效果没有达到最好（其他的都达到了最好）？
> - 通过这种改进可以发现，transH相比于TransE并没有显著提升，原因是什么？

## Triple Classification

- TransE-RS and TransH-RS have same parameter and operation complexities as TransE and TransH, which is lower than TransR and TransD.
- Our models randomly initial the entities, not use the learned embeddings by TransE as TransR and TransD.
  - It means that our models have much better ability to overcome the problem of overfitting

## Distributions of Triplets’ Scores

### aim

analyze the difference between $L_R$ Loss and our $L_RS$ Loss

### Parameters

![20190603155952784132260.jpg](http://image.nysdy.com/20190603155952784132260.jpg)

> 思考：
>
> - 对于我自己正在做的实验：是不是我自己用的间隔太小了

### result

![20190603155952821850690.jpg](http://image.nysdy.com/20190603155952821850690.jpg)

![20190603155952848841252.jpg](http://image.nysdy.com/20190603155952848841252.jpg)

> 思考
>
> - 这部分的实验值得借鉴，它可以相对于直观的可以展示出为什么效果会好。
> - 比如对于上述为什么改进后的transE的效果会更好
>   - 看到最后的分数分布transE-RS的分布效果和Trans-H的十分接近，
>   - 而transE的模型较为简单，可能最终loss最小化会使得模型充分表达，而其他模型引入了更多的假设可能会带来更多的噪声
>   - 也可能当loss很小时，其他的假设条件发挥作用的很小（至少从实验结果来看是的，但是还有待于进一步设计实验验证）

## Discussion of Parameters

### Discussion on γ1 and γ2.

![20190603155952911595962.jpg](http://image.nysdy.com/20190603155952911595962.jpg)

- We find that γ2 = 3γ1 or γ2 = 4γ1 is better for link prediction, but for triplet classification there are not obvious characteristics on γ1 and γ2.
- a lower γ2 is expected to ensure the golden condition $\mathbf{h}+\mathbf{r} \approx \mathbf{t}$ for positive triplets, but an entity needs to satisfy many golden coditions at the same time.

> 思考
>
> - 既然如作者说，那么理论上transH的效果应该很好才对，但是结果并不是这样的，这又产生矛盾。

### Discussion on λ

![20190603155953002076189.jpg](http://image.nysdy.com/20190603155953002076189.jpg)

> 思考
>
> - 看到λ并没有对模型影响并没很大
> - λ在1左右是效果会比较好
> - λ和margin会不会产生关联？