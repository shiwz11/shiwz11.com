---
title: proposal_literature_summary_04.Rmd
author: None
date: '2020-09-10'
slug: proposal-literature-summary-04-rmd
categories:
  - Thesis
tags:
  - Academic
subtitle: ''
summary: ''
authors: []
lastmod: '2020-09-10T10:34:13+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---



# The Economics and Psychology of Personality Traits

## Part One

Bowles, Gintis, and Osborne (2001), for a review. Among other determinants of earnings, they summarize evidence on the labor market effects of beauty by Hamermesh and Biddle (1994) and Hamermesh, Meng, and Zhang (2002).

Marxist economists (Bowles and Gintis, 1976) pioneered the analysis of the impact of personality on earnings. Mueser (1979) estimates empirical relationships between personality traits and earnings. 

Mueller and Plug (2006) relate the Big Five personality factors to earnings. Hartog (1980, 2001) draws on the psychology literature to analyze economci preferences. van Praag (1985) and van Praag and van Weeren (1988) also link econonics with psychology.

<!-----------我是分界线----------------->

Defination of personality traits:

We focus our analysis on personality traits, defined as patters of thought, feelings, and behavior. We do not discuss in depth motivation, values interests, and attitudes which give rise to personality traits.

<!------------我是分界线-------------------------->

In this paper, examined the claim that behavior is purely **situation-specific** and show evidence againest it.

**本篇论文主要探讨了以下问题：**

(1) Is it conceptually possible to separate cognitive ability from personality traits?

很多方面的性格特点是认知能力的“果”， 而认知能力又取决于性格特点。但是，我们依然可以通过个体之间的差异将这两者分离。

(2) Is it possible to empirically distinguish cognitive from personality traits?

经济偏好的衡量是受数学能力和智力的影响的。IQ 测试的结果不仅仅受智商的影响，同时也受到如动机或焦虑程度的影响。更进一步，在个体的整个生命周期，个体认知能力的构建是受到如好奇心、野心和毅力这类的性格特点的影响的。

(3) What are the main measurement systems in psychology for intelligence and personality, and how are they validated?

大多数的心理学家依赖于纸面的“自我报告”类型的调查问卷，其他的一些心理学家和经济学家衡量一些常见的经济学偏好变量，例如，时间偏好，风险厌恶（程度）。我们总结了两种类型的研究，表明，心理学文献中还有一大片空白区域：它没有很好的融合这两种测量系统。

一个值得注意的一点是，当代的性格心理学家更偏重于结构效应的测量而不是效果。

(4) What is the evidence on the predictive power of cognitive and presonality traits?

我们总结了认知能力和人格特征两方面的证据，涉及到教育、工资、犯罪、青春期妊娠和寿命长度。
对于某几个被解释变量，个别的（大五人格中的）某几个性格特点会更有解释力度。
认知能力的解释力度在更多的任务中都是有效的，而不同的性格特点只有在特定的任务中才有解释力度。

(5) How stable are personality traits across situations and across the life cycle? Are they more sensitive than cognitive traits to investment and intervention?

我们证明了，认知能力和性格特点在生命周期都是会演化的，知识其演化程度和时间段是不一样的。
认知能力在儿童期陡增，然后在成年期达到峰值，最后再慢慢下降。而某些性格特点，从儿童期到后成年期线性增加。

(6) Do the findings from psychology suggest that conventional economic theory should be enriched? Can conventional models of perferences in economics explain the body of evidence from personality psychology? Does personality syschology merely recast well-known preference parameters into psychological jargon, or is there somthing new for economists to learn?

传统的经济学理论足够灵活以兼容心理学中的很多发现，而我们的分析发现，某些发现不能被现在的传统经济学理论所兼容。

我们构建了一个框架，在其中，可以把心理学变量作为传统经济选择的约束。
最近的试图将经济学和心理学融合起来的努力似乎都集中在将偏好变量上，而我们呈现了一个更家广阔的框架，在其中，可以融合约束、技能获得和学习以及传统经济学的偏好变量。

## Part Two

**A. Factor Analysis**

$T_{i,j}$: 代表个体i在任务j中的表现；

$f_{i}, i = 1, \dots , I$: 一个向量，代表个体i的因子或心理特质，I是个体的数量

$f_{i} = (f_{i, \dots}, f_{i, L})$: 该向量有L个组成部分

$V_{i, j}$: 个体i在任务j中决定生产力的其他变量。

个体i在任务j中的表现可以表示为

\begin{equation}
\begin{aligned}
T_{i, j} = h_{j}(f_{i}, V_{i, j}), i = 1, \dots, I, j = 1, \dots, J.
\end{aligned}
(\#eq:pls04-01)
\end{equation}

不同任务中，不同因子的重要程度是不一样的，如，在一个纯粹认知的任务中，$f_{i}$对于任务结果是无足轻重的。

心理学研究中广泛采用**线性因子模型**：

\begin{equation}
\begin{aligned}
T_{i,j} = \mu_{j} + \lambda_{j} f_{i} + V_{i, j}, i = 1, \dots, I, j = 1, \dots , J.
\end{aligned}
(\#eq:pls04-02)
\end{equation}

$\mu_{j}$: 第j个任务的均值；

$\lambda_{j}$: 因子负荷向量；

公式 \@ref(eq:pls04-01)、\@ref(eq:pls04-02) 意味着：

a. 潜在特质$f_i$产生了不同的产出；

b. 任务产出是对特质($f_i$)的不完美测度；

c. 除了测试，使用任务也可能还好的代理我们的潜在特质。

**B. 认知能力**

美国心理学协会把智力（或者认知能力）定义为：理解复杂事物的能力，适应环境的能力，从过去经历学习的能力，进行推理的能力，通过思考克服困难的能力。(Neisser et al.1996, p.77)

一级的因子在所有类型的任务中都有解释力，对应于方程\@ref(eq:pls04-01)中的$j = 1, \dots, J$。而低级的因子只在部分的任务中有解释力。低级的因子可能与一级因子相关，且可能低级因子之间相互相关。

Carroll (1993) 和 Horn 和 McArdle (2007) 归纳了大量的经验证据，反对以下论述：因子“g”在解释成就测试和智力测试的相关结构上是有效的。

**C. 性格特质**

执行能力不是一项特质，而是一个被脑前额皮质控制的行为的总称。执行能力包括行为抑制、工作记忆、专注和其他所谓的自上而下的控制低级活动的脑力活动。

本文集中在更容易与认知能力区分的人格特质，他们与致力不同，定义为处理抽象问题的能力。大多数对人格特质的度量仅仅与IQ存在弱相关关系(Webb 1915; McCrae and Costa 1994; Stankov 2005; Ackerman and Heggestad 1997). 然而，存在特例，在其中，IQ适度的和大五人格特质中的开放性，和刺激寻求特质，和对时间偏好的度量。

在第三部分我们发现，IQ测试中的表现被人格特质变量影响。

**D. Operationalizing the Concepts**

IQ测试中的工作假设是：一个测试对应于某一特定的$f_{i}$组成部分，而不同的测试对应于不同的组成部分。

Wechsler (1939) 指出了斯坦福-比奈测试中两个局限：（1）它过分依赖于词汇技巧因此导致其取决于正式的教育；（2）针对成年人的心理年龄和实际年龄的比率是不合适的。（Boake 2002）

**E. 性格测试**

性格测试最初是用作描述个体差异的，IQ测试最初是用于预测个别任务上的表现的。

主流的性格理论即设一个类似于智力那样的分层模型。最广为接受的一个人格特质分类法是大五或者成为五因子模型。

Big Five: Conscientiousness, Extraversion, Agreeableness, Neuroticism (also called Emotional Stability).

对五因子模型最尖锐的批评是：它是和理论脱节的.

Block (1995) 不仅质疑了五因子模型本身，更一般地，他质疑了使用因子分析作为工具分析性格特点的有效性。

**F. 气质（性情/性格）的测度**

发展心理学家门使用性情(Temperament)来描述婴儿和儿童期的行为倾向。因为这些差别在生命的一开始就显现出来，因此这些特质传统上被假设为一种生理学差别（以和由环境引致的进行区别）。

然而行为基因学发现，就像成年人的性格特质一样，性情(Temperament)只是部分可继承的，而且，成年人和孩子的（特质）测量都是受环境影响的。

### 测量以及方法学问题

大体上有两种类型的测量方案：(a) 追求测出或引出惯用的经济学偏好变量；(b) 使用自报告或观察人员报告的方法来册来嗯性格特质。

### The Evidence On Preference Parameters

**A. 时间折现**

Frederick et al. 建议说时间偏好是三维度的，包括三个潜在的冲动：

冲动、强迫症、自我抑制

**B. 风险偏好**

Harrison, Lau, and Rutstrom (2007) 在Denmark 253个代表性样本身上使用真实的收益来测量风险偏好。

**C. 闲暇偏好**

**D. 利他主义和社会偏好**

### 性格特质的解释力度

Hogan and Holland (2003) 曾经展示了感情稳定度可以成为一个有效的、普遍的对工作表现测度的变量。

**A. 现存的预测效率的局限性**

1. 鉴于IQ的收益是单调递增的，大多数性格特质的最优点可能位于两个极端点之间。(Benson and Campbell 2007; LaHuis, Martin, and Avis 2005).

2. 第二，短的性格测试对大五人格的每一个领域产生一个分数，而把这作为捕捉性格特质和产出之间关系的预测变量未免太过生硬。

3. 第三，大样本研究中的性格测量基本上都是通过简略的调查问卷度量的，而这种方式导致对性格特质对产出的影响的估计更不可信也更不精确。

4. 第四，这些估计方法并没有抓住交互效应。

5. 标准的对预测效力的度量方法是效果大小以及被解释的方差的大小。然而，$R^2$，或者称之为拟合优度只是衡量变量重要性的其中一个方式。

### Changing Preference Parameters and Psychological Variables

性格的可煅性可以通过以下几种方式来定义并测量：

a. 某个特质在某段时间的平均变化程度，通过其在某段时间的得分变化来度量。

b. 等级顺序变化，是指群体中特质的序数等级变化，并通过测试-在测试等级相关性来衡量


\cite{Hejingtong-Huoyan-2007}

