---
title: How to Write Research Papers (Part I)
date: 2018-01-21 09:02:50
tags: 论文写作
---
本文基本参考 PPT [How to Write Research Papers][1]，在论文写作套路方面也有导师给的一些私货。

>The key research contributions are the deciding factor for your paper’s acceptance.

-- 尽管论文写作有许多套路，但是论文能被接收的关键还是要看你的研究成果。

>Quality/impact over quantity of papers

-- 论文的质量远胜于质量。

<!--more -->
## 好论文的特征

- **Interesting** research (有趣的研究)
    - 提出有趣的问题
    - 在问题的解决方法上提出了有趣的想法
    - 在方法的评估上有有趣的发现
- **Novel** research (新颖的研究，being the first)
    - 提出新的问题
    - 提出新的解决方案
    - 有新的发现
- **Inspiring** research (研究推广，research generalization)
    - Problem formulation，抽象化一类问题，使其能够描述类似的问题
    - Slution formulation，抽象化一类解，使其能够解决相似的问题
- **Impactful** research (意义深远的研究)
    - Impactful problem
        - high severity level：impact an case seriously
        - large scope level：impact many cases
    - Impactful solution，解决问题的高效（effective/efficient）方法
        - 发现前人研究中未发现的巨大漏洞
        - 比前人的方法节省了 N 倍时间
        - 能拿出实际的证据再好不过
- **Rigorous/accurate** description 严格/准确的描述
    - 准确的问题定义（无论是否形式化）
    - 使用算法和实例详细描述解决方案（**不能只有实例！**），达到可重现（reprodicible）的要求
- **Significant** research (e.g., not easy problem to solve)
    - Technical challenges 技术上的挑战
        - 问题的水平
        - 解得水平
    - 智商上的碾压
- **Validated** research
    - Clear and strong (empirical) evidence to validate/justify the claims. 清晰而有力的证据

## Key Questions to Double Check Your Paper

① Is the research problem significant/important? (研究的问题的重要性)

>**YES** a problem that people care (evidenced by concrete statistics or examples) (大家都关心的问题总是很有意义的)
**NO** a problem created/imagined by you and no one else cares about it (冷门的问题研究比较危险)

② Is your research solution significant or addressing technical challenges? (你的解决方案是否意义重大或者解决了技术上的难题)
>**NO** a solution that is incremental over previous work (比前人方案复杂的解决方案并不受待见)
**NO** a solution that is straightforward/trivial (解决方案过于简单或者性能轻微提升也是不行的)

③ Is your evaluation justifying the claimed contributions or benefits of your solution? (你的评估手段是否能够准确体现你的解决方案对于问题的贡献）
>Double check by making traceability from your claims listed in your contributions to your research questions to investigate in your evaluation

论文中涉及到的术语、观点、主张或者解决方案要有可追溯性。确保每一个 contribution/claim 都被适当地转化为了研究问题，没有未经证实的证据；确保每个问题都得到了适当地度量。

[GQM][2] ： 目标问题度量(Goal Question Metric)

**Traceability Links**

![Traceability Links][3]

## Know What Your Audience is
明确的介绍你的论文与你要投的会议/期刊的相关性以及契合程度。
>E.g., if ICSM, explain clearly in abstract and intro how your work is related to maintenance; if WWW, explain clearly in abstract and intro how your work is related to web; …

详细的解释你的工作中用到的一些基本假设/概念，即使这些在你所从事的领域里是最最基本的，但是你也不能够确定审稿人或者读者能够理解。
>E.g., if your approach is about achieving high structural coverage of code, need to explain why achieving high structural coverage is important

## Justify Your Choices
论文中药明确你做的任何一个选择。为什么你要提出一个新方法而不是用已有的方案？为什么你要选择这个方法解决你的问题而不是另一个可选方案？ 某种技术的使用与否对结果是否由影响，影响有多大？你的结果和前人研究结果相比新能提升多少？

**Pitfall**：In intro sec, you describe that you propose a way of solutions to address your stated problem, BUT you never discuss why alternative way of solutions would not be chosen

**Pitfall**: In approach sec, you describe that you use a technique to address a sub-problem in your approach, BUT you never discuss why alterative way of techniques would not be chosen

**Pitfall**: In your evaluation sec, you don’t compare the results of including or not including an important technique claimed to be a major contribution

**Pitfall**: in your evaluation sec, you don’t justify why you choose the experimental subjects or a subset of subjects used by previous work

## Don’t Write Too Little or More Than Enough
文章的页数是有限制的，要严格按照期刊的要求，太多太少都不好。在某些很具体或者很 low-level 的细节上不要花太多笔墨，而在一些重要的思想、实验结果等方面要着重叙述。

Need balance on reproducibility (可重现性) and new idea/research contributions
>E.g. solution: separation of approach and implementation sections (方法与实现分离，附源码...)

## Formalize Just Enough
适当的形式化。形式化定义，算法...

Formalization helps

- write clearly and force you to think and write rigorously
- help grasp problem/solution essence, avoid shooting moving targets

但是不要过分形式化，形式化的目的是为了让审稿人/读者更容易理解，而不是增加阅读难度。

##  Technical Challenges
### Why list challenges?
论文中列出技术挑战有利于体现论文的成果以及重要性，审稿人/读者不会被一些简单的解决方案打动。

主要可以从两个方面列举

- Problem-level challenges，解决问题有哪些挑战？
- Solution-level challenges，执行解决问题的方案有哪些挑战？

### Simple vs. Sophisticated Solutions
求解复杂问题时不要忽视简单的、最直接的解决方案（例如枚举法），最简单的解决方案可以作为评估中的 baseline。

>"Make things as simple as possible, but not simpler." - Einstein

### Challenges → Contribution Points

contribution list 的一般结构：

- The overall approach
- A list of specific techniques in the approach
- Implementation and evaluation
- Evaluation results

对于贡献列表中的每一项技术，至少应该有一个明确的技术挑战，或者也可以将整个方法表述为技术上的挑战。

## 附：导师关于写论文的小 Tips 【更新中】
- 论文的 Abstract 不要超过 5 句话，介绍论文涉及到的领域现状，你要解决的问题现状，前人的研究进展怎样，你用了什么方法解决问题，取得了怎样的效果……一句话也不宜太长，5句话差不多就10行；
- 每一段要有中心句，很 detail 的解释不宜放在 Abstract 中；
- 论文一般不用主动语态，用被动语句，我们解决的是客观的问题；
- 遇到模棱两可的词一定要查字典，推荐 [Cambridge Dictionary][4]
- 多用连接词，however，on constract，whereas...
- 针对不同的期刊会议，在 Introduction 中有不同的策略；
- 一段不宜太长，太长意味着一段有多个中心句。一段话只说一个事情；
- 注意文章字体，figure 的字体和大小；
- 论文 figure 尽量不要使用 powerpoint/excel 做，推荐使用 [PSTricks][5]
- 注意 figure 使用的语义，特别是流程图，不同的形状的语义要统一；figure 不宜过长，figure 字体要和中文统一；figure 占用空间要仔细斟酌，寸土寸金
- 注意 figure caption 的写法，一般看 caption 就能够使读者了解整幅图要表达的意思
- 能够用表格表示的就不要用文字，能够用图表示的就不要用表格
- 类似的图不要太多
- 多做量化结果
- 提到帧率一定要说要说在哪个硬件上
- 说多少倍 小于10 写数字，大于 10 写文字
- 专有名词如果不是太普遍要在论文中加定义
- several/some 等非定量词最好不要用，very 不要用 替代：significantly
- great amount -> huge amount，最好用中性的词




  [1]: https://people.engr.ncsu.edu/txie/publications/writepapers.pdf
  [2]: http://en.wikipedia.org/wiki/GQM
  [3]: http://static.zybuluo.com/guoxs/yuyo5v7kerlqo1oot154ir3s/001.png
  [4]: https://dictionary.cambridge.org/
  [5]: https://tug.org/PSTricks/main.cgi/
