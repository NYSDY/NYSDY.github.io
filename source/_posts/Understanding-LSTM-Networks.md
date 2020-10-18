---
title: Understanding LSTM Networks
date: 2019-01-06 14:44:13
tags:
- LSTM
- RNN
english_title: Understanding LSTM Networks
categories:
- 神经网络
---

> [原文链接](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)。这篇文章很好很细的一步一步的分解讲解了LSTM，之前看过一篇翻译的博客，现在自己翻译一遍，感觉对LSTM的认识加深了许多，虽然还是对LSTM中存有一些问题，比如为什么用tanh，sigmoid，为什不采用其他的？，但是看过之后至少对LSTM没有那么畏惧，不觉得过于复杂了。

<!-- more -->

# LSTM

## Recurrent Neural Networks

![](https://i.loli.net/2019/01/06/5c31826c20c5f.jpg)

循环允许信息从一个网络传入下一个。

![](https://i.loli.net/2019/01/06/5c3189b3b43e8.jpg)

一个循环网络可以被认为是相同网络的多个复制，每一个网络都将信息传递给后继者。这种类似链的性质表明，递归神经网络与序列和列表密切相关。

## The Problem of Long-Term Dependencies

RNN的一个吸引力是他们可能能够将先前信息连接到当前任务。

- 有时，我们只需要查看最近的信息来执行当前任务。例如：

  > If we are trying to predict the last word in “the clouds are in the *sky*,” we don’t need any further context – it’s pretty obvious the next word is going to be sky.

   在这种情况下，如果相关信息与待预测地方之间的差距很小，RNN可以学习使用过去的信息。

- 但是，对于一些情况，我们需要更多的上下文信息。

  > Consider trying to predict the last word in the text “I grew up in France… I speak fluent *French*.”

  这时，相关信息与需要变得非常大的点之间的差距完全有可能。不幸的是，随着差距的扩大，RNN无法学会连接信息。

  > The problem was explored in depth by [Hochreiter (1991) [German\]](http://people.idsia.ch/~juergen/SeppHochreiter1991ThesisAdvisorSchmidhuber.pdf) and [Bengio, et al. (1994)](http://www-dsi.ing.unifi.it/~paolo/ps/tnn-94-gradient.pdf), who found some pretty fundamental reasons why it might be difficult.

## LSTM Networks

> LSTMs are explicitly designed to avoid the long-term dependency problem. Remembering information for long periods of time is practically their default behavior, not something they struggle to learn!

标准RNNs的重复模块只有一个简单的结构，比如tanh层。

![](https://i.loli.net/2019/01/06/5c31942fa0852.jpg)



LSTMs也有像这种链式结构，但是它的重复模块具有不同的结构。 有四个，而不是一个神经网络层，以一种非常特殊的方式进行交互。

![](https://i.loli.net/2019/01/06/5c3195013e06a.jpg)

基本符号如下：

![](https://i.loli.net/2019/01/06/5c319541ec43b.jpg)

在上图中，每一行都携带一个完整的向量，从一个节点的输出到其他节点的输入。 粉色圆圈表示逐点运算，如矢量加法，而黄色框表示神经网络层。 行合并表示连接，而行分叉表示其内容被复制，副本将转移到不同的位置。

## The Core Idea Behind LSTMs

**LSTM的关键是单元状态**，水平线贯穿图的顶部。

单元状态有点像传送带。 它直接沿着整个链运行，只有一些微小的线性相互作用。 信息很容易沿着它不变地流动。

![](https://i.loli.net/2019/01/06/5c31965a2ea97.jpg)

LSTM确实能够移除或添加信息到细胞状态，由称为门的结构精心调节。

门是一种可选择通过信息的方式。 它们由Sigmoid神经网络层和逐点乘法运算组成。

![](https://i.loli.net/2019/01/06/5c3196e0eb9e6.jpg)

sigmoid层输出0到1之间的数字，描述每个组件应该通过多少。 值为零意味着“不让任何东西通过”，而值为1则意味着“让一切都通过！”

LSTM具有三个这样的门，用于保护和控制单元状态。

## Step-by-Step LSTM Walk Through

我们的第一步就是确定我们将从单元状态中丢弃的信息。这个决定是由一个称为“遗忘门层”的sigmoid层决定的。它查看$h_{t-1}$和$x_t$，并为单元状态$C_{t-1}$中的每一个数字输出一个介于0和1之间的数字。1代表“完全保留这个”，而0代表“完全舍弃这个”。

让我们回到我们的语言模型示例，试图根据以前的所有单词预测下一个单词。 在这样的问题中，单元状态可能包括当前受试者的性别，因此可以使用正确的代词。 当我们看到一个新主题时，我们想要忘记旧主题的性别。

![](https://i.loli.net/2019/01/06/5c31997a2ecec.jpg)

下一步是确定我们将在单元状态中存储哪些新信息。 这有两个部分。 首先，称为“输入门层”的sigmoid层决定我们将更新哪些值。 接下来，tanh层创建可以添加到状态的新候选值$\tilde{C}_t$的向量。 在下一步中，我们将结合这两个来创建状态更新。

在我们的语言模型的例子中，我们想要将新主题的性别添加到单元格状态，以替换我们忘记的旧主题。

![](https://i.loli.net/2019/01/06/5c319afb31868.jpg)

现在是时候将旧的单元状态$C_{T-1}$更新为新的单元状态$C_t$。 前面的步骤已经决定要做什么，我们只需要实际做到这一点。

我们将旧状态乘以$f_t$，忘记我们之前决定忘记的事情。 然后我们添加$i_t * \tilde{C}_t$。 这是新的候选值，根据我们决定更新每个状态的值来缩放。

在语言模型的情况下，我们实际上放弃了关于旧主题的性别的信息并添加新信息，正如我们在前面的步骤中所做的那样。

![](https://i.loli.net/2019/01/06/5c319c9f37f63.jpg)

最后，我们需要决定我们要输出的内容。 此输出将基于我们的单元状态，但将是过滤版本。 首先，我们运行一个sigmoid层，它决定我们要输出的单元状态的哪些部分。 然后，我们将单元格状态设置为tanh（将值推到介于-1和1之间）并将其乘以sigmoid门的输出，以便我们只输出我们决定的部分。

对于语言模型示例，由于它只是看到一个主题，它可能想要输出与动词相关的信息，以防接下来会发生什么。 例如，它可能输出主语是单数还是复数，以便我们知道动词应该与什么形式共轭，如果接下来的话。

![](https://i.loli.net/2019/01/06/5c319da831938.jpg)

## Variants on Long Short Term Memory

到目前为止我所描述的是一个非常正常的LSTM。 但并非所有LSTM都与上述相同。 事实上，似乎几乎所有涉及LSTM的论文都使用略有不同的版本。 差异很小，但值得一提的是其中一些。

> One popular LSTM variant, introduced by [Gers & Schmidhuber (2000)](ftp://ftp.idsia.ch/pub/juergen/TimeCount-IJCNN2000.pdf), is adding “peephole connections.” This means that we let the gate layers look at the cell state.

![](https://i.loli.net/2019/01/06/5c319ec053e96.jpg)

上面的图表为所有门增加了窥视孔（peephole），但是许多论文会给一些窥视孔而不是其他的。

另一种变化是使用耦合的遗忘和输入门。 我们不是单独决定忘记什么以及应该添加新信息，而是一起做出这些决定。**我们仅仅会当我们在当前位置将要输入时忘记。我们仅仅输入新的值到那些我们已经忘记旧的信息的那些状态** 。

![](https://i.loli.net/2019/01/06/5c31a06e327f5.jpg)

另一个改动较大的变体是 Gated Recurrent Unit (GRU)，这是由 [Cho, et al. (2014)](http://arxiv.org/pdf/1406.1078v3.pdf) 提出。它将遗忘和输入门组合成一个“更新门”。它还合并了单元状态和隐藏状态，并进行了一些其他更改。 由此产生的模型比标准LSTM模型简单，并且越来越受欢迎。

![](https://i.loli.net/2019/01/06/5c31a1285f207.jpg)

这些只是最着名的LSTM变种中的一小部分。 还有很多其他的东西，如 [Yao, et al. (2015)](http://arxiv.org/pdf/1508.03790v2.pdf) 提出的 Depth Gated RNN。 还有一些完全不同的解决长期依赖关系的方法，如 [Koutnik, et al. (2014)](http://arxiv.org/pdf/1402.3511v1.pdf) 提出的 Clockwork RNN。

哪种变体最好？ 差异是否重要？  [Greff, et al. (2015)](http://arxiv.org/pdf/1503.04069.pdf) 对流行变体进行了很好的比较，发现它们几乎完全相同。[Jozefowicz, et al. (2015)](http://jmlr.org/proceedings/papers/v37/jozefowicz15.pdf) 测试了超过一万个RNN架构，找到了一些在某些任务上比LSTM更好的架构。

## Conclusion

早些时候，我提到了人们用RNN取得的显着成果。基本上所有这些都是使用LSTM实现的。对于大多数任务来说，它们确实工作得更好！

作为一组方程写下来，LSTM看起来非常令人生畏。希望，在这篇文章中逐步走过它们使他们更加平易近人。

LSTM是我们用RNN实现的重要一步。很自然地想知道：还有另一个重要的一步吗？研究人员的共同观点是：“是的！下一步是它的注意！“我们的想法是让RNN的每一步都从一些更大的信息集中选择信息。例如，如果您使用RNN创建描述图像的标题，则可能会选择图像的一部分来查看其输出的每个单词。

事实上， [Xu, *et al.*(2015)](http://arxiv.org/pdf/1502.03044v2.pdf) 做到这一点 - 如果你想探索注意力，这可能是一个有趣的起点！使用注意力已经取得了许多非常令人兴奋的结果，似乎还有更多的事情即将来临......

注意力不是RNN研究中唯一令人兴奋的问题。例如，[Kalchbrenner, *et al.* (2015)](http://arxiv.org/pdf/1507.01526v1.pdf) 的Grid LSTMs似乎非常有希望。在生成模型中使用RNN工作 - 例如[Gregor, *et al.* (2015)](http://arxiv.org/pdf/1502.04623.pdf), [Chung, *et al.* (2015)](http://arxiv.org/pdf/1506.02216v3.pdf), 或者 [Bayer & Osendorfer (2015)](http://arxiv.org/pdf/1411.7610v3.pdf)  似乎也很有趣。过去几年对于反复出现的神经网络来说是一个激动人心的时刻，即将到来的那些承诺只会更加如此！