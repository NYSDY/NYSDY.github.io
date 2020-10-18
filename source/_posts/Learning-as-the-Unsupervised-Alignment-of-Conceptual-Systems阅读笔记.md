---
title: Learning as the Unsupervised Alignment of Conceptual Systems阅读笔记
date: 2019-06-25 14:42:57
tags:
english_title: Learning_as_the_Unsupervised_Alignment_of_Conceptual_Systems
categories:
- 论文阅读笔记
---



> 这篇文章没怎么看懂，主要思想应该是代表同时概念的不同形式（文本，图像，语音等）应该具有相似的分布，以此来进行无监督的概念对齐。这种思路挺不错的，不过还没有深入的想法，算是拓展视野吧！

<!-- more -->

# KEY

The key insight is that each concept has a unique signature within one conceptual system (e.g., images) that is recapitulated in other systems (e.g., text or audio)

# Problem Statement

- For supervised approaches, as the number of concepts grows, so does the number of required training examples
- V. W. Quine argued, even supervised instruction contains a substantial amount of ambiguity (Quine, 1960).Quine suggested that meaning may derive from something’s place within a conceptual system.

# Model

In order to solve Quinne’s problem, we align a system of word labels and a system of visual semantics that both refer to the same underlying reality and therefore have related structure that can be discovered by unsupervised means (Figure 1）