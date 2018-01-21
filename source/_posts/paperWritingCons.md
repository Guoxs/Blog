---
title: How to Write Research Papers (Part II)
date: 2018-01-21 11:53:32
tags: 论文写作
---
典型的科技论文结构如下：

- Title/Abstract
- Introduction
- Optional: Background
- Optional: Formal Problem Definition
- Related Work (alternatively put before conclusion)
- Example
- Approach/Framework
- Implementation
- Evaluation
    - Experiment/Case Studies/Experiences/Examples
- Discussion
- Conclusions (and Future work)

下面将论文分解，按点一一介绍其结构以及写作时的注意事项。
<!--more -->
## Title and Abstract
标题和概述就相当于一个人的脸，一个让人印象深刻的标题和简明扼要的概述是十分重要的，这直接影响了审稿人或读者是否有继续看下去的意愿。

**Title writing**

- 不要使用不常用的时髦词 (uncommon buzzwords)
>这有利于读者通过阅读标题了解文章大致内容

- 要具体，但不要太具体 (Be specific enough but not too specific)
- 给你的方法一个好名字 (Name your approach with a cute name)
>这方便别人记住和引用

**Abstract structure**

- Short motivation (problem)
- Proposed solution
- Evaluation
- Evaluation results

>介绍你研究的问题，说明针对这个问题你提出了什么解决方法，你是怎么评估你的方法的好坏，评估的结果怎么样...

**Abstract writing**

- Don't put unexplained or undefined terms whose meanings are not well known
- Solutions: explain them; rephrase them using plain words; not get into too much detail (without mentioning them).

>不要使用无法解释或者未定义的、意义不明的术语，可以组织语言重新措辞，避开这些模棱两可的词语

## Introduction Part

### Introduction Structure
- 领域长期的动向，需要解决的问题，为什么现有的解决方案不行，存在那些不足...(可以举例说明)
- 说明你研究的问题的重要性，需要使用具体的统计数据、具体的例子或者引用
- 建议的解决方案（输入/输出）和关键思想（步骤）
- （可选）简短介绍相关的研究并且说明差异
- Evaluation and evaluation results (评估和评估结果)
- （可选）The paper makes the following main contributions: + 分项列出，这方便审稿人抓住重点，哪个是主要贡献
- 页面结构布局，可以向读者展示不同部分的关联，这在论文的其他部分也同样适用。

### Introduction pitfall

- 不要花费过分笔墨叙述 (even throughout the paper)
    - 可以使用流程图/示意图来展示你的工作
    - 但是需要着重强调问题的重要性以及你的解决方案
- 不要过多引用别人的工作 (even throughout the paper)
- 如果想提出一些已经证实的想法(unjustified points)，最好是将其放在 conclusion 或者 discussion 部分
- 注意措辞和单词拼写

>X "Our approach provides a foundation for this new field."
"We believe our approach can provide a foundation…"
"We believe our approach has a good potential for providing a foundation …"

另一个例子：
>X "Our/X’s approach is the only/first one on …."
"With the best of our knowledge, our/X’s approach is the only one/first on …"
"Our/X’s approach is one of the/a few approaches …"
"Our/X’s approach is a major/representative approach …"

另外一个需要注意的问题是，有些审稿人很厌烦你自己说你的方法是 "**novel**" 的，他们认为
你的方法 "novel" 不 "novel" 这个问题是由他们决定的，而不是你说了算。所以不要强调你的方法很
“novel”，或者至少不要将 **“novel”** 这个词放在你的论文中！

>"TestEra: A Novel Framework for Automated Testing of Java Programs" → "TestEra: Specification-based Testing of Java Programs Using SAT"

### Stirewalt's 5-paragraph rule on writing Introduction

[Stirewalt's 5-paragraph rule][1]

- **Introductory paragraph**: Very briefly: What is the problem and why is it relevant to the audience attending *THIS CONFERENCE*? Moreover, why is the problem hard, and what is your solution? You must be brief here. This forces you to boil down your contribution to its bare essence and communicate it directly.
>引言段。非常简短：问题是什么？为什么与这次会议有关？为什么这个问题很难解决？你的解决方案是什么？这里必须做简短介绍，你需要直接说明你的贡献.

- **Background paragraph**: Elaborate on why the problem is hard, critically examining prior work, trying to tease out one or two central shortcomings that your solution overcomes.
>相关背景介绍段。详细说明为什么这个问题是困难的，仔细考究前人的工作，试着找出你的解决方案克服的一两个主要缺点。

- **Transition paragraph**: What keen insight did you apply to overcome the shortcomings of other approaches? Structure this paragraph like a syllogism: Whereas P and P => Q, infer Q.
>过渡段，你的方法能够克服缺点的原因原因的什么？方法的关键思想是什么？这个段的结构类似于三段论：P and P => Q，则 Q。

- **Details paragraph**: What technical challenges did you have to overcome and what kinds of validation did you perform?
>细节段，你需要克服哪些技术难题？你使用的验证方案又是什么？

- **Assessment paragraph**: Assess your results and briefly state the broadly interesting conclusions that these results support. This may only take a couple of sentences. I usually then follow these sentences by an optional overview of the structure of the paper with interleaved section callouts.
>评估段， 评估你的结果，并简要说明这些结果所引出的关键结论。这可能只需要几句话，之后还可以对整篇文章结构做一个概述。

**[The Stanford InfoLab's patented five-point structure for Introductions][2]**

- What is the problem?
- Why is it **interesting** and **important**?
- Why is it **hard**? (E.g., why do **naive approaches** fail?)
- Why hasn't it been **solved** before? (Or, what's wrong with previous proposed solutions? How does mine **differ**?)
- What are the key components of my approach and results? Also include any specific limitations.

### Tell a Good Story in Intro
Abstract 和 introduction 是非常重要的部分，通常审稿人可以通过阅读摘要和简介准确地判断这篇文章是接受还是拒绝。在这种情况下，我们需要在摘要和简介部分讲述一个有趣生动的故事（以正确和角度和恰当的抽象层次），使得读者忍不住往下读，这也能激发读者的求知欲。

>After finishing reading the short description of the target problem, they couldn’t predict what challenges or significant issues real world setting could face.

>After finishing reading the description of your problem, they couldn’t predict what solutions you will provide (e.g., clever, neat ideas to address challenges)  → attract them to read on

**Suggestion Actions Against Intro/Abstract**
- 在小组讨论中反复迭代改进，可以大声朗读，听取小组成员的意见
- 注意 Abstract 部分句子的逻辑过渡以及 Introduction 部分段落的逻辑过渡，可以借助思维导图整理思路。
- 多次确认先前陈诉的特征是否满足
    - 例如：目标问题很重要...
    - 例如：你的解决方案是很有优势的，解决了非平凡的技术难题，并且得到了很好的验证

## Problem Definition (optional)
如果你的论文提出了一个新的问题或者解决了一个 formalizable 的问题，那么最好有一个**问题定义** (problem definition) 的部分。这样一节有助于清晰地描述论文所讨论的问题。

例如：[Mining Exception-Handling Rules as Sequence Association Rules][3]

**Formal Problem Definition**

- 定义你的方法想要解决的问题
- 可以放在 intro/example 部分之后的一个部分，为之后的示例目的部分服务
- 可以在这部分正式定义你的方法中提到的一些重要概念（问题空间或解空间的概念）。
- 将问题规范化于是论文的贡献之一。

以下审稿人关于 formal definition 部分的意见很有借鉴意义：
>1. "Section 3, the formal definition, is not very well organized. A formal definition can be useful and clarifying, but in that case ought to be crisp, clear, and elegant. To my taste your definitions are a bit messy"

>2. "Definition 1 is not really a definition"

>3. "It is also interesting to see that you don't use your formal definition in the rest of the paper."

>4. "I am not sure what the formalization of XXX adds. It seems rather disconnected to the rest of the paper."

## Background and Related Work
**Background** 有时也叫作 **Preliminaries**，包括 notion，teminology 或者你以前的工作等。

 **Related Work**

- 不要简单的列出相关的工作而不涉及你自己的工作
    - keywords to use：whereas、in contrast、but、however
    - "excuses" to use: "does not require specs", "focus on different problems", "complement with each other", …
    - 可以简单描述几种类似的方法，并与你的方法作比较

- 不要仅在解空间上列举你的工作与相关工作之间的差异，也需要考虑问题空间的对比

**related work** 部分应该放在 introduction/example 部分之后，在 conclusion 部分之前。（放在 example 之后也是可以的）

- After the introduction/example section
    - Pros: Immediately clear out reviewers’ wonder on how the work differs from previous work
    - Cons: hard to let readers to know what you are talking about before showing the approach details
- Before the conclusion section
    - Pros: Now reviewers’ know what your approach is about
    - Cons: reviewers keep wondering how the work differs from previous work till this point

>对于非常相关的工作，应该在 introduction 部分指出差异

## Example

- A simple example
    - Include: where it comes from; a figure listing source code; brief description
    - Throughout the paper, it is important to have illustrating examples for those places that contain “dry” descriptions of your approach
    - If you use several examples throughout the paper, you may not need a separate Example section.
- Optional/important part of the section: high level description of applying your approach on the example
    - describe inputs/outputs of your approach without getting into too much detail
    - very important if the later approach description involves heavy hard-to-understand formalisms

## Approach or Framework
- 在抽象层上概括你的工作，比如概括为“框架”、“算法”而不是工具
    - What you develop should be beyond your own implementation
    - A workflow diagram is useful for explaining your framework
- 理念与具体实现分离
- 细节方面最好用示例解释

## Implementation

- What libraries you used in your tool
- Detailed implementations of each step in your framework
- List complications of implementing a certain idea and how you get around them
    - if some complications are important and general, you may move them to the framework section
- Applicable to both approach/implementation

## Evaluation
### 实验部分写作结构

- Hypotheses/Questions to be answered
>Double check your questions. Ex. "**Can** our approach perform better than a previous related approach?" → "**How much better** can our approach perform than …"

- Measures you use to answer these questions (higher better?)
- Experiment setup
>a good number of subjects, some scripts, some third-party tools or reimplemented tools for comparison

- Independent variables+dependent variables → metrics
- Experimental results
    - Illustrate how to read your table/diagrams
    - Explain what does the curve or data mean，e.g. "We observed that …", "The experimental results show"
    - Summarize your findings, remember to get back to answer the hypotheses and questions
    - Optional: discussion subsection
- Sometimes you may not include cost (time/memory) in your experimental results but you need to at least discuss the analysis cost
- Threats to validity: internal, external, and construct, sometimes may not need that fined-grained type classification
- In the experimental results, need describe how the results relate back to which hypotheses and how hypotheses relate back to which research questions

>When using colored figures, make sure you describe both colors and gray-scale in text (since people may read papers in black-white copy)

### 评估部分

- Need explain evaluation results or describe your insights from the observed results rather than just describing the results
>if some subjects’ results are especially favorable or unfavorable, explain the reasons or even your hypothesis ("We suspect that …"  "We hypothesize that …")

- Need describe "Experiment Designs"
- Need hypothesis testing, t-testing especially if you want to say "A result is **significantly** better than B result"; **statistically significant vs. practically significant**
- Using "benchmarks"
- Measure both mean and variance/deviation, not just mean

>Tips: Construct a project web including the evaluation subjects, evaluation results, Building **trust** from reviewers in your work and your results.

## Discussion

- Limitations and issues your approach/implementation currently cannot address
    - Optional: how are you going to address them in future work
- Other caveats (scope of your approach)
- It is often a good idea to list (obvious) limitations and discuss possible solutions for them rather than hiding them
- Possible applications of your approach that you haven’t validated but are convincingly feasible or effective

>Under what situations your proposed solution would achieve the best results and under what situations your proposed solution would achieve the worst results

## Conclusions (and Future Work)
这个部分只需要简单的总结你的工作。
>In the introduction: "We propose a new approach …"
In the conclusions: "We have proposed a new approach …"

也可以描述你的方法的应用场景以及未来的改进方向。
>"We are currently doing X..., and preliminary results are promising."

最后就是 Acknowledgments.






  [1]: http://www.cse.msu.edu/~chengb/Writing/intro-guidelines-stirewalt.txt
  [2]: http://infolab.stanford.edu/~widom/paper-writing.html
  [3]: https://people.engr.ncsu.edu/txie/publications/icse09-carminer.pdf
