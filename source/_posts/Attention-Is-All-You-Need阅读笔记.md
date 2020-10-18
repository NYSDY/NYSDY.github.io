---
title: Attention Is All You Need阅读笔记
date: 2019-05-25 13:57:35
tags:
- attention
- transformer
- translation
- classical
english_title: Attention Is All You Need
categories:
- 论文阅读笔记
- classical
---



> transformer 是一个完全由注意力机制组成的搭建的模型，模型复杂度低，并可以进行并行计算，使得计算速度快。在翻译模型上取得了较好的效果。本篇论文属于经典必读论文，阅读笔记中对一些不清楚的地方进行了汉语解释，读完论文后阅读参考链接以加深理解。
>
> [论文下载地址](https://arxiv.org/pdf/1706.03762)

<!-- more -->

# research objective

based solely on attention mechanisms, increase parallezable computation and decrease train time

# Problem Statement

recurrent models hidden states depended on previous hidden state and the input for position precludes parallelization

# contribution

- Transformer,
  - eschewing recurrence and instead relying entirely on an attention mechanism, solve the long dependency problem.
  - draw global dependecies between input and output 
  - allow for significantly more parallelization

# Model Architecture

The Transformer uses stacked self-attention and point-wise, fully connected layers for both the encoder and decoder.![2019052515587656321218.jpg](http://image.nysdy.com/2019052515587656321218.jpg)

## Encoder and Decoder Stacks 

### Encoder

- compose of a stack of N identical layers
- each layers has two sub-layers
  1. multi-head self-attention mechanism
  2. position-wise fully connected feed forward network
- employ a residual connection around each of the two sub-layers, followed by layer normalization
- the output of each sub-layer is $\text { LayerNorm }(x+\text { Sublayer }(x))$
- encoder中的Q，K，V都是学出来的

### Decoder

- composed of a stack of N identical layers
- has the same two sub-layers as the encoder
- the third sub-layer between the two sub-layers
  - perform multi-head attention over the output of the encoder stack
- add a mask to modify the self-attention sub-layer to ensure that the predictions for position $i$ can depend only the known outputs at positions less than $i$
- 除了第一子层中Q，K，V是自己学出来的，第二个子层利用了encoder中的K，V。

## Attention

![20190526155885488441950.jpg](http://image.nysdy.com/20190526155885488441950.jpg)

### Scaled Dot-Product Attention

the calculation process as the left at the figure 2. **formula：**
$$
\text { Attention }(Q, K, V)=\operatorname{softmax}\left(\frac{Q K^{T}}{\sqrt{d_{k}}}\right) V
$$

- where $\sqrt{d_{k}}$ is to  prevent value from getting too large, which will push the softmax function into regions where it has extremely small gradients. 因为量级太大，softmax后就非0即1了，不够“soft”了。也会导致softmax的梯度非常小。也就是让softmax结果**不稀疏**(问号脸，通常人们希望得到更稀疏的attention吧)。
- $Q, K,V$ is a matrix needed to learn from input.

### Multi-Head Attention

**helps the encoder look at other words in the input sentence as it encodes a specific word**

in the figure 2 right. 

- it's beneficial to **lineraly project** the quries, keys and values $h$ times with different, learned projections to $d_k, d_k, d_v$ dimensions, respectively
- concatenate the output 

$$
\begin{aligned} \text { MultiHead }(Q, K, V) &=\text { Concat (head }_{1}, \ldots, \text { head }_{h} ) W^{O} \\ \text { where head }_{i} &=\text { Attention }\left(Q W_{i}^{Q}, K W_{i}^{K}, V W_{i}^{V}\right) \end{aligned}
$$

where $W_{i}^{Q} \in \mathbb{R}^{d_{\text { model }} \times d_{k}}, W_{i}^{K} \in \mathbb{R}^{d_{\text { model }} \times d_{k}}, W_{i}^{V} \in \mathbb{R}^{d_{\text { model }} \times d_{v}}, W^{O} \in \mathbb{R}^{h d_{v} \times d_{\mathrm{model}}}$

### Applications of Attention in our Model

- the queries come from the previous decoder layer, and the memory keys and values come from the output of the encoder. mimicing the seq-to-seq
- self -attention can make that each position in the encoder can attend to all positions in the previous layer of the encoder
- **We need to prevent leftward information flow in the decoder to preserve the auto-regressive property. We implement this inside of scaled dot-product attention by masking out (setting to −∞) all values in the input of the softmax which correspond to illegal connections. See Figure 2**。即我们只能attend到前面已经翻译过的输出的词语，因为翻译过程我们当前还并不知道下一个输出词语，这是我们之后才会推测到的。即将$QK^T$中每行该单词之后的数值做处理，使得前面的单词看不到后面单词所占的重要性程度。

## Position-wise Feed-Forward Networks



- applied to each position separately and identically
- feed-forward network consists of tow linear transformations with a ReLU activation. formula:

$$
\mathrm{FFN}(x)=\max \left(0, x W_{1}+b_{1}\right) W_{2}+b_{2}
$$

> **小结**
>
> 1. 为什么叫强调`position-wise`?
>    - 解释一: 这里FFN层是每个position进行相同且独立的操作，所以叫position-wise。对每个position独立做FFN。
>    - 解释二：从卷积的角度解释，这里的FFN等价于kernel_size=1的卷积，这样每个position都是独立运算的。如果kernel_size=2，或者其他，position之间就具有依赖性了，貌似就不能叫做position-wise了
> 2. 为什么要采用全连接层？
>    - 目的: 增加非线性变换
>    - 如果不采用FFN呢？有什么替代的设计？
> 3. 为什么采用2层全连接，而且中间升维？
>    - 这也是所谓的bottle neck，只不过低维在IO上，中间采用high rank

## Embeddings and Softmax

Sharing the same weight maatrix between the two embedding layers and the pre-softmax linear transformation

## Positional Encoding

Using sine and xosine functions of different frequencies:
$$
P E_{(p o s, 2 i)}=\sin \left(p o s / 10000^{2 i / d_{\text { model }}}\right)
\\
P E_{(p o s, 2 i+1)}=\cos \left(p o s / 10000^{2 i / d_{\mathrm{model}}}\right)
$$

- where $pos$ is the postiiton and $i$ is the dimension

- **Authors hypothesized it would allow the model to easily learn to attend by relative positions, since for any fixed offset $k$, $PE_{pos+k}$can be represented as a linear function of $PE_{pos}$**

  但在语言中，`相对位置`也很重要，Google选择前述的位置向量公式的一个重要原因是：由于我们有$\sin (\alpha+\beta)=\sin \alpha \cos \beta+\cos \alpha \sin \beta$以及$\cos (\alpha+\beta)=\cos \alpha \cos \beta-\sin \alpha \sin \beta$，这表明位置$p+k$的向量可以表示成位置$p$的向量的线性变换，这提供了表达相对位置信息的可能性。

- Compared with using learned positional embeddings, the sinusoidal version may allow the model to extrapolate to sequence lengths longer than the ones encountered during training.

注意由于该模型没有recurrence或convolution操作，所以没有明确的关于单词在源句子中位置的相对或绝对的信息，为了更好的让模型学习位置信息，所以添加了position encoding并将其叠加在word embedding上。

# Why Self-Attention

- total computational complexity per layer
- the amount of computation that can be parallelized
- the path between long-range dependencies in the network

![20190527155892126287777.jpg](http://image.nysdy.com/20190527155892126287777.jpg)

![20190528155902465937774.jpg](http://image.nysdy.com/20190528155902465937774.jpg)

self-attention|：

- $QK^TV$相乘，根据矩阵大小（分别为$n*d, n*d, n*d$需要的复杂度为$O(n^2d*2)$（忽略softmax）
- maximum path length：图说明了， 对于self-attention, target node (生成的那个点) 实际上和 输入中的任意一点的距离是相同的

convolutional:  

- 每层有k个卷积和，对于input matix（$n*d$)矩阵执行卷积需要运算复杂度是$n*d*(d-m)$, m为卷积和宽度是一个比较小的常数，所以总复杂度为$O\left(k \cdot n \cdot d^{2}\right)$,作者提到可分离的卷基层暂时还不了解，可以以后查阅。

- maximum path length: 正常卷积和的距离是$O(n/k)$, 但如果是堆叠卷积如图：　　![2019052815590251436862.jpg](http://image.nysdy.com/2019052815590251436862.jpg)

  就可以减小到$O\left(\log _{k}(n)\right)$

recurrent:

- 计算是每个词向量乘隐藏权重($d*d$)，所以易得计算复杂度：$O\left(n \cdot d^{2}\right)$
- maximum path length: 长度就是n。
- 操作步骤要从第一个到第n个为n步，是有顺序的。其他的都没有顺序要求

self-attentin(restricted)

- 相当于只输入r邻近的句子长度，自然可以得到如图结果

# Train

## Optimizer

$$
\text { lrate }=d_{\text { model }}^{-0.5} \cdot \min \left(\text {step}_{-} n u m^{-0.5}, \text { step }_{-} n u m \cdot \text { warmup steps }^{-1.5}\right)
$$

- increasing the learning rate linearly for the first warmup_steps training steps
-  decreasing it thereafter proportionally to the inverse square root of the step number

## Regularization

### Residual Dropout

- apply dropout to the output of each sub-layer, before it is added to the sub-layer input and normalized
- apply dropout to the sums of the embeddings and the positional encodings in both the encoder and decoder stacks

### Label Smoothing

- This hurts perplexity, as the model learns to be more unsure, but improves accuracy and BLEU score

# Result

## machine Translation

Even our base model surpasses all previously published models and ensembles, at a fraction of the training cost of any of the competitive models

![2019052715589406009133.jpg](http://image.nysdy.com/2019052715589406009133.jpg)

## Model Variations

![20190527155894062794865.jpg](http://image.nysdy.com/20190527155894062794865.jpg)

# **缺点**

缺点在原文中没有提到，是后来在Universal Transformers中指出的，在这里加一下吧，主要是两点：

1. 实践上：有些rnn轻易可以解决的问题transformer没做到，比如复制string，尤其是碰到比训练时的sequence更长的时
2. 理论上：transformers非computationally universal（[图灵完备](https://www.zhihu.com/question/20115374/answer/288346717)），（我认为）因为无法实现“while”循环

# **总结**

Transformer是第一个用纯attention搭建的模型，不仅计算速度更快，在翻译任务上也获得了更好的结果。

Google现在的翻译应该是在此基础上做的，但是数据量大可能用transformer好一些，小的话还是继续用rnn-based model。

花了不少时间，算是理解了attention和transformer，对其中不是很清楚的点如attention的内部中Q，K，V具体是什么在self-attention和multi-head attention中大小是不同的，如何mask，如何计算复杂，等进行查阅资料弄懂了。总体来说还是收获很大的。准备在看一些代码讲解。

# reference

- Attention机制详解（二）——Self-Attention与Transformer - 川陀学者的文章 - 知乎
  https://zhuanlan.zhihu.com/p/47282410
- **https://jalammar.github.io/illustrated-transformer/（这个讲的比较详细，建议看完论文后再看一遍这个会加深理解）**
- 【NLP】Transformer详解 - 李如的文章 - 知乎
  https://zhuanlan.zhihu.com/p/44121378
- https://blog.eson.org/pub/664e9bad/
- https://mp.weixin.qq.com/s/J-anyCuwLd5UYjTsUFNT1g