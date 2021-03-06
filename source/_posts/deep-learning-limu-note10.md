---
title:  MXNet/Gluon 深度学习笔记 (十) —— 词向量与语言模型
date: 2018-02-26 10:38:42
tags: DeepLearning
mathjax: true
---

自然语言是一套用来表达含义的复杂系统. 在这套系统中，词是表义的基本单元. 在机器学习中，使用词向量来表示词. 顾名思义，词向量是用来表示词的向量，通常也被认为是词的特征向量. 近年来，词向量已逐渐成为自然语言处理的基础知识.

![word_scatter][1]

一个很自然的想法就是使用 one-hot 向量表示词, 假设词典中不同词的数量为 $N$ ，每个词可以和从 0 到 $N-1$ 的连续整数一一对应。假设一个词的相应整数表示为 $i$ ，为了得到该词的 one-hot 向量表示，我们创建一个全 0 的长为 $N$ 的向量，并将其第 $i$ 位设成 1 。然而，使用 one-hot 词向量并不是一个好选择。一个主要的原因是，one-hot 词向量无法表达不同词之间的相似度。例如，任何一对词的 one-hot 向量的余弦相似度都为 0 。
<!--more-->
## word2vec
2013年，Google团队发表了 [word2vec](https://code.google.com/archive/p/word2vec/) 工具。word2vec 工具主要包含两个模型：跳字模型（skip-gram）和连续词袋模型（continuous bag of words，简称 CBOW ），以及两种高效训练的方法：负采样（negative sampling）和层序 softmax（hierarchical softmax）。值得一提的是，word2vec 词向量可以较好地表达不同词之间的相似和类比关系。

word2vec 自提出后被广泛应用在自然语言处理任务中。它的模型和训练方法也启发了很多后续的词向量模型。本节将重点介绍 word2vec 的模型和训练方法。
### 模型
#### 跳字模型 (skip-gram)
在跳字模型中，我们用一个词来预测它在文本序列周围的词。例如，给定文本序列 "the", "man", "hit", "his", 和 "son"，跳字模型所关心的是，给定 "hit"，生成它邻近词 “the”, "man", "his", 和 "son" 的概率。在这个例子中，"hit" 叫中心词，“the”, "man", "his", 和 "son" 叫背景词。由于 "hit" 只生成与它距离不超过 2 的背景词，该时间窗口的大小为 2。


我们来描述一下跳字模型。


假设词典大小为 $|\mathcal{V}|$ ，我们将词典中的每个词与从 0 到 $|\mathcal{V}|-1$ 的整数一一对应：词典索引集 $\mathcal{V} = \{0, 1, \ldots, |\mathcal{V}|-1\}$ 。一个词在该词典中所对应的整数称为词的索引。给定一个长度为 $T$ 的文本序列中，$t$ 时刻的词为 $w^{(t)}$ 。当时间窗口大小为 $m$ 时，跳字模型需要最大化给定任一中心词生成背景词的概率：

$$ \prod_{t=1}^T \prod_{-m \leq j \leq m, j \neq 0} \mathbb{P}(w^{(t+j)} \mid w^{(t)})$$

上式的最大似然估计与最小化以下损失函数等价

$$ -\frac{1}{T} \sum_{t=1}^T \sum_{-m \leq j \leq m, j \neq 0} \text{log} \mathbb{P}(w^{(t+j)} \mid w^{(t)})$$


我们可以用 $\mathbf{v}$ 和 $\mathbf{u}$ 分别代表中心词和背景词的向量。换言之，对于词典中一个索引为$i$的词，它在作为中心词和背景词时的向量表示分别是 $\mathbf{v}_i$ 和 $\mathbf{u}_i$ 。而词典中所有词的这两种向量正是跳字模型所要学习的模型参数。为了将模型参数植入损失函数，我们需要使用模型参数表达损失函数中的中心词生成背景词的概率。假设中心词生成各个背景词的概率是相互独立的。给定中心词 $w_c$ 在词典中索引为 $c$ ，背景词 $w_o$ 在词典中索引为 $o$ ，损失函数中的中心词生成背景词的概率可以使用 softmax 函数定义为

$$\mathbb{P}(w_o \mid w_c) = \frac{\text{exp}(\mathbf{u}_o^\top \mathbf{v}_c)}{ \sum_{i \in \mathcal{V}} \text{exp}(\mathbf{u}_i^\top \mathbf{v}_c)}$$

当序列长度 $T$ 较大时，我们通常随机采样一个较小的子序列来计算损失函数并使用随机梯度下降优化该损失函数。通过微分，我们可以计算出上式生成概率的对数关于中心词向量 $\mathbf{v}_c$ 的梯度为：

$$\frac{\partial \text{log} \mathbb{P}(w_o \mid w_c)}{\partial \mathbf{v}_c} = \mathbf{u}_o - \sum_{j \in \mathcal{V}} \frac{\text{exp}(\mathbf{u}_j^\top \mathbf{v}_c)}{ \sum_{i \in \mathcal{V}} \text{exp}(\mathbf{u}_i^\top \mathbf{v}_c)} \mathbf{u}_j$$

而上式与下式等价：

$$\frac{\partial \text{log} \mathbb{P}(w_o \mid w_c)}{\partial \mathbf{v}_c} = \mathbf{u}_o - \sum_{j \in \mathcal{V}} \mathbb{P}(w_j \mid w_c) \mathbf{u}_j$$

通过上面计算得到梯度后，我们可以使用随机梯度下降来不断迭代模型参数 $\mathbf{v}_c$ 。其他模型参数 $\mathbf{u}_o$ 的迭代方式同理可得。最终，对于词典中的任一索引为 $i$ 的词，我们均得到该词作为中心词和背景词的两组词向量 $\mathbf{v}_i$ 和 $\mathbf{u}_i$ 。
#### 连续词袋模型 (CBOW)
连续词袋模型与跳字模型类似。与跳字模型最大的不同是，连续词袋模型中用一个中心词在文本序列周围的词来预测该中心词。例如，给定文本序列 "the", "man", "hit", "his", 和 "son"，连续词袋模型所关心的是，邻近词 “the”, "man", "his", 和 "son" 一起生成中心词 "hit" 的概率。

假设词典大小为 $|\mathcal{V}|$ ，我们将词典中的每个词与从 0 到 $|\mathcal{V}|-1$ 的整数一一对应：词典索引集 $\mathcal{V} = \{0, 1, \ldots, |\mathcal{V}|-1\}$ 。一个词在该词典中所对应的整数称为词的索引。给定一个长度为 $T$ 的文本序列中，$t$ 时刻的词为 $w^{(t)}$ 。当时间窗口大小为 $m$ 时，连续词袋模型需要最大化由背景词生成任一中心词的概率：

$$ \prod_{t=1}^T  \mathbb{P}(w^{(t)} \mid  w^{(t-m)}, \ldots,  w^{(t-1)},  w^{(t+1)}, \ldots,  w^{(t+m)})$$

上式的最大似然估计与最小化以下损失函数等价

$$  -\sum_{t=1}^T  \text{log} \mathbb{P}(w^{(t)} \mid  w^{(t-m)}, \ldots,  w^{(t-1)},  w^{(t+1)}, \ldots,  w^{(t+m)})$$

我们可以用 $\mathbf{v}$ 和 $\mathbf{u}$ 分别代表背景词和中心词的向量（注意符号和跳字模型中的不同）。换言之，对于词典中一个索引为$i$的词，它在作为背景词和中心词时的向量表示分别是 $\mathbf{v}_i$ 和 $\mathbf{u}_i$ 。而词典中所有词的这两种向量正是连续词袋模型所要学习的模型参数。为了将模型参数植入损失函数，我们需要使用模型参数表达损失函数中的中心词生成背景词的概率。给定中心词 $w_c$ 在词典中索引为 $c$ ，背景词 $w_{o_1}, \ldots, w_{o_{2m}}$ 在词典中索引为 $o_1, \ldots, o_{2m}$ ，损失函数中的背景词生成中心词的概率可以使用 softmax 函数定义为

$$\mathbb{P}(w_c \mid w_{o_1}, \ldots, w_{o_{2m}}) = \frac{\text{exp}[\mathbf{u}_c^\top (\mathbf{v}_{o_1} + \ldots + \mathbf{v}_{o_{2m}}) /(2m) ]}{ \sum_{i \in \mathcal{V}} \text{exp}[\mathbf{u}_i^\top (\mathbf{v}_{o_1} + \ldots + \mathbf{v}_{o_{2m}}) /(2m)]}$$

当序列长度 $T$ 较大时，我们通常随机采样一个较小的子序列来计算损失函数并使用随机梯度下降优化该损失函数。通过微分，我们可以计算出上式生成概率的对数关于任一背景词向量 $\mathbf{v}_{o_i}$($i = 1, \ldots, 2m$) 的梯度为：

$$\frac{\partial \text{log} \mathbb{P}(w_c \mid w_{o_1}, \ldots, w_{o_{2m}})}{\partial \mathbf{v}_{o_i}} = \frac{1}{2m}(\mathbf{u}_c - \sum_{j \in \mathcal{V}} \frac{\text{exp}(\mathbf{u}_j^\top \mathbf{v}_c)}{ \sum_{i \in \mathcal{V}} \text{exp}(\mathbf{u}_i^\top \mathbf{v}_c)} \mathbf{u}_j)$$

而上式与下式等价：

$$\frac{\partial \text{log} \mathbb{P}(w_c \mid w_{o_1}, \ldots, w_{o_{2m}})}{\partial \mathbf{v}_{o_i}} = \frac{1}{2m}(\mathbf{u}_c - \sum_{j \in \mathcal{V}} \mathbb{P}(w_j \mid w_c) \mathbf{u}_j)$$


通过上面计算得到梯度后，我们可以使用随机梯度下降)来不断迭代各个模型数 $\mathbf{v}_{o_i}$($i = 1, \ldots, 2m$)。其他模型参数 $\mathbf{u}_c$ 的迭代方式同理可得。最终，对于词典中的任一索引为 $i$ 的词，我们均得到该词作为背景词和中心词的两组词向量 $\mathbf{v}_i$ 和 $\mathbf{u}_i$。
### 近似训练法
我们可以看到，无论是跳字模型还是连续词袋模型，每一步梯度计算的开销与词典 $\mathcal{V}$ 的大小相关。显然，当词典较大时，例如几十万到上百万，这种训练方法的计算开销会较大。所以，使用上述训练方法在实践中是有难度的。

我们将使用近似的方法来计算这些梯度，从而减小计算开销。常用的近似训练法包括负采样和层序 softmax。
#### 负采样
我们以跳字模型为例讨论负采样。

词典 $\mathcal{V}$ 大小之所以会在目标函数中出现，是因为中心词 $w_c$ 生成背景词 $w_o$ 的概率 $\mathbb{P}(w_o \mid w_c)$ 使用了 softmax，而 softmax 正是考虑了背景词可能是词典中的任一词，并体现在 softmax 的分母上。

我们不妨换个角度，假设中心词 $w_c$ 生成背景词 $w_o$ 由以下相互独立事件联合组成来近似

* 中心词 $w_c$ 和背景词 $w_o$ 同时出现在该训练数据窗口
* 中心词 $w_c$ 和第 1 个噪声词 $w_1$ 不同时出现在该训练数据窗口（噪声词 $w_1$ 按噪声词分布 $\mathbb{P}(w)$ 随机生成，假设一定和 $w_c$ 不同时出现在该训练数据窗口）
* ...
* 中心词 $w_c$ 和第 $K$ 个噪声词 $w_K$ 不同时出现在该训练数据窗口（噪声词 $w_K$ 按噪声词分布 $\mathbb{P}(w)$ 随机生成，假设一定和 $w_c$ 不同时出现在该训练数据窗口）

我们可以使用 $\sigma(x) = 1/(1+\text{exp}(-x))$ 函数来表达中心词 $w_c$ 和背景词 $w_o$ 同时出现在该训练数据窗口的概率：

$$\mathbb{P}(D = 1 \mid w_o, w_c) = \sigma(\mathbf{u}_o^\top \mathbf{v}_c)$$

那么，中心词 $w_c$ 生成背景词 $w_o$ 的对数概率可以近似为

$$ \text{log} \mathbb{P} (w_o \mid w_c) = \text{log} [\mathbb{P}(D = 1 \mid w_o, w_c) \prod_{k=1, w_k \sim \mathbb{P}(w)}^K \mathbb{P}(D = 0 \mid w_k, w_c) ]$$

假设噪声词 $w_k$ 在词典中的索引为 $i_k$ ，上式可改写为

$$ \text{log} \mathbb{P} (w_o \mid w_c) = \text{log} \frac{1}{1+\text{exp}(-\mathbf{u}_o^\top \mathbf{v}_c)}  + \sum_{k=1, w_k \sim \mathbb{P}(w)}^K \text{log} [1-\frac{1}{1+\text{exp}(-\mathbf{u}_{i_k}^\top \mathbf{v}_c)}] $$

因此，有关中心词 $w_c$ 生成背景词 $w_o$ 的损失函数是

$$ - \text{log} \mathbb{P} (w_o \mid w_c) = -\text{log} \frac{1}{1+\text{exp}(-\mathbf{u}_o^\top \mathbf{v}_c)}  - \sum_{k=1, w_k \sim \mathbb{P}(w)}^K \text{log} \frac{1}{1+\text{exp}(\mathbf{u}_{i_k}^\top \mathbf{v}_c)} $$


当我们把 $K$ 取较小值时，每次随机梯度下降的梯度计算开销将由 $\mathcal{O}(|\mathcal{V}|)$ 降为 $\mathcal{O}(K)$ 。

我们也可以对连续词袋模型进行负采样。有关背景词 $w^{(t-m)}, \ldots,  w^{(t-1)},  w^{(t+1)}, \ldots,  w^{(t+m)}$ 生成中心词 $w_c$ 的损失函数

$$-\text{log} \mathbb{P}(w^{(t)} \mid  w^{(t-m)}, \ldots,  w^{(t-1)},  w^{(t+1)}, \ldots,  w^{(t+m)})$$

在负采样中可以近似为

$$-\text{log} \frac{1}{1+\text{exp}[-\mathbf{u}_c^\top (\mathbf{v}_{o_1} + \ldots + \mathbf{v}_{o_{2m}}) /(2m)]}  - \sum_{k=1, w_k \sim \mathbb{P}(w)}^K \text{log} \frac{1}{1+\text{exp}[(\mathbf{u}_{i_k}^\top (\mathbf{v}_{o_1} + \ldots + \mathbf{v}_{o_{2m}}) /(2m)]}$$

同样地，当我们把 $K$ 取较小值时，每次随机梯度下降的梯度计算开销将由 $\mathcal{O}(|\mathcal{V}|)$ 降为 $\mathcal{O}(K)$ 。
#### 层序 softmax
层序 softmax 利用了二叉树。树的每个叶子节点代表着词典 $\mathcal{V}$ 中的每个词。每个词 $w_i$ 相应的词向量为 $\mathbf{v}_i$ 。我们以下图为例，来描述层序 softmax 的工作机制。


![hierarchical_softmax][2]


假设 $L(w)$ 为从二叉树的根到代表词 $w$ 的叶子节点的路径上的节点数，并设 $n(w,i)$ 为该路径上第 $i$ 个节点，该节点的向量为 $\mathbf{u}_{n(w,i)}$ 。以上图为例，$L(w_3) = 4$ 。那么，跳字模型和连续词袋模型所需要计算的任意词 $w_i$ 生成词 $w$ 的概率为：

$$\mathbb{P}(w \mid w_i) = \prod_{j=1}^{L(w)-1} \sigma([n(w, j+1) = \text{leftChild}(n(w,j))] \cdot \mathbf{u}_{n(w,j)}^\top \mathbf{v}_i)$$

其中 $\sigma(x) = 1/(1+\text{exp}(-x))$，如果 $x$ 为真，$[x] = 1$；反之 $[x] = -1$。

由于 $\sigma(x)+\sigma(-x) = 1$，$w_i$ 生成词典中任何词的概率之和为 1：

$$\sum_{w=1}^{\mathcal{V}} \mathbb{P}(w \mid w_i) = 1$$


让我们计算 $w_i$ 生成 $w_3$ 的概率，由于在二叉树中由根到 $w_3$ 的路径上需要向左、向右、再向左地遍历，我们得到

$$\mathbb{P}(w_3 \mid w_i) = \sigma(\mathbf{u}_{n(w_3,1)}^\top \mathbf{v}_i)) \cdot \sigma(-\mathbf{u}_{n(w_3,2)}^\top \mathbf{v}_i)) \cdot \sigma(\mathbf{u}_{n(w_3,3)}^\top \mathbf{v}_i))$$

我们可以使用随机梯度下降在跳字模型和连续词袋模型中不断迭代计算字典中所有词向量 $\mathbf{v}$ 和非叶子节点的向量 $\mathbf{u}$ 。每次迭代的计算开销由 $\mathcal{O}(|\mathcal{V}|)$ 降为二叉树的高度 $\mathcal{O}(\text{log}|\mathcal{V}|)$。
## GloVe
[GloVe](https://nlp.stanford.edu/pubs/glove.pdf) 是 Standford 团队在 2014 年发表的. GloVe 使用了词与词之间的共现（co-occurrence）信息。我们定义 $\mathbf{X}$ 为共现词频矩阵，其中元素 $x_{ij}$ 为词 $j$ 出现在词 $i$ 的环境（context）的次数。这里的“环境”有多种可能的定义。举个例子，在一段文本序列中，如果词 $j$ 出现在词 $i$ 左边或者右边不超过 10 个词的距离，我们可以认为词 $j$ 出现在词 $i$ 的环境一次。令 $x_i = \sum_k x_{ik}$ 为任意词出现在词 $i$ 的环境的次数。那么，

$$P_{ij} = \mathbb{P}(j \mid i) = \frac{x_{ij}}{x_i}$$

为词 $j$ 出现在词 $i$ 的环境的概率。这一概率也称词 $i$ 和词 $j$ 的共现概率。
### 共现概率比值
[GloVe论文](https://nlp.stanford.edu/pubs/glove.pdf) 里展示了以下一组词对的共现概率与比值：

* $\mathbb{P}(k \mid \text{ice})$：0.00019（$k$= solid），0.000066（$k$= gas），0.003（$k$= water），0.000017（$k$= fashion）
* $\mathbb{P}(k \mid \text{steam})$：0.000022（$k$= solid），0.00078（$k$= gas），0.0022（$k$= water），0.000018（$k$= fashion）
* $\mathbb{P}(k \mid \text{ice}) / \mathbb{P}(k \mid \text{steam})$：8.9（$k$= solid），0.085（$k$= gas），1.36（$k$= water），0.96（$k$= fashion）


我们通过上表可以观察到以下现象：

* 对于与 ice 相关而与 steam 不相关的词 $k$ ，例如 $k=$solid ，我们期望共现概率比值 $P_{ik}/P_{jk}$ 较大，例如上面最后一栏的 8.9。
* 对于与 ice 不相关而与 steam 相关的词 $k$ ，例如 $k=$gas ，我们期望共现概率比值 $P_{ik}/P_{jk}$ 较小，例如上面最后一栏的 0.085。
* 对于与 ice 和 steam 都相关的词 $k$ ，例如 $k=$water ，我们期望共现概率比值 $P_{ik}/P_{jk}$ 接近 1，例如上面最后一栏的 1.36。
* 对于与 ice 和 steam 都不相关的词 $k$ ，例如 $k=$fashion，我们期望共现概率比值 $P_{ik}/P_{jk}$ 接近 1，例如上面最后一栏的 0.96。

由此可见，共现概率比值能比较直观地表达词之间的关系。GloVe 试图用有关词向量的函数来表达共现概率比值。

### 用词向量表达共现概率比值

GloVe 的核心在于使用词向量表达共现概率比值。而任意一个这样的比值需要三个词 $i$、$j$ 和 $k$ 的词向量。对于共现概率 $P_{ij} = \mathbb{P}(j \mid i)$，我们称词 $i$ 和词 $j$ 分别为中心词和背景词。我们使用 $\mathbf{v}$ 和 $\tilde{\mathbf{v}}$ 分别表示中心词和背景词的词向量。

我们可以用有关词向量的函数 $f$ 来表达共现概率比值：

$$f(\mathbf{v}_i, \mathbf{v}_j, \tilde{\mathbf{v}}_k) = \frac{P_{ik}}{P_{jk}}$$

需要注意的是，函数 $f$ 可能的设计并不唯一。首先，我们用向量之差来表达共现概率的比值，并将上式改写成

$$f(\mathbf{v}_i - \mathbf{v}_j, \tilde{\mathbf{v}}_k) = \frac{P_{ik}}{P_{jk}}$$

由于共现概率比值是一个标量，我们可以使用向量之间的内积把函数 $f$ 的自变量进一步改写。我们可以得到

$$f((\mathbf{v}_i - \mathbf{v}_j)^\top \tilde{\mathbf{v}}_k) = \frac{P_{ik}}{P_{jk}}$$

由于任意一对词共现的对称性，我们希望以下两个性质可以同时被满足：

* 任意词作为中心词和背景词的词向量应该相等：对任意词 $i$ ，$\mathbf{v}_i = \tilde{\mathbf{v}}_i$
* 词与词之间共现次数矩阵 $\mathbf{X}$ 应该对称：对任意词 $i$ 和 $j$ ，$x_{ij} = x_{ji}$

为了满足以上两个性质，一方面，我们令

$$f((\mathbf{v}_i - \mathbf{v}_j)^\top \tilde{\mathbf{v}}_k) = \frac{f(\mathbf{v}_i^\top \tilde{\mathbf{v}}_k)}{f(\mathbf{v}_j^\top \tilde{\mathbf{v}}_k)}$$

并得到 $f(x) = \text{exp}(x)$ 。以上两式的右边联立，


$$\exp(\mathbf{v}_i^\top \tilde{\mathbf{v}}_k) = P_{ik} = \frac{x_{ik}}{x_i}$$

由上式可得

$$\mathbf{v}_i^\top \tilde{\mathbf{v}}_k = \log(x_{ik}) - \log(x_i)$$

另一方面，我们可以把上式中 $\log(x_i)$ 替换成两个偏移项之和 $b_i + b_k$，得到

$$\mathbf{v}_i^\top \tilde{\mathbf{v}}_k = \log(x_{ik}) - b_i - b_k$$

将索引 $i$ 和 $k$ 互换，我们可验证对称性的两个性质可以同时被上式满足。

因此，对于任意一对词 $i$ 和 $j$ ，用它们词向量表达共现概率比值最终可以被简化为表达它们共现词频的对数：

$$\mathbf{v}_i^\top \tilde{\mathbf{v}}_j + b_i + b_j = \log(x_{ij})$$


### 损失函数

上式中的共现词频是直接在训练数据上统计得到的，为了学习词向量和相应的偏移项，我们希望上式中的左边与右边越接近越好。给定词典大小 $V$ 和权重函数 $f(x_{ij})$ ，我们定义损失函数为

$$\sum_{i, j = 1}^V f(x_{ij}) (\mathbf{v}_i^\top \tilde{\mathbf{v}}_j + b_i + b_j - \log(x_{ij}))^2$$

对于权重函数 $f(x)$ ，一个建议的选择是，当 $x < c$（例如 $c = 100$ ），令 $f(x) = (x/c)^\alpha$（例如 $\alpha = 0.75$ ），反之令 $f(x) = 1$ 。需要注意的是，损失函数的计算复杂度与共现词频矩阵 $\mathbf{X}$ 中非零元素的数目呈线性关系。我们可以从 $\mathbf{X}$ 中随机采样小批量非零元素，使用随机梯度下降迭代词向量和偏移项。

需要注意的是，对于任意一对 $i, j$，损失函数中存在以下两项之和

$$f(x_{ij}) (\mathbf{v}_i^\top \tilde{\mathbf{v}}_j + b_i + b_j - \log(x_{ij}))^2 + f(x_{ji}) (\mathbf{v}_j^\top \tilde{\mathbf{v}}_i + b_j + b_i - \log(x_{ji}))^2$$

由于 $x_{ij} = x_{ji}$，对调 $\mathbf{v}$ 和 $\tilde{\mathbf{v}}$ 并不改变损失函数中这两项之和的值。也就是说，在损失函数所有项上对调 $\mathbf{v}$ 和 $\tilde{\mathbf{v}}$ 也不改变整个损失函数的值。因此，任意词的中心词向量和背景词向量是等价的。只是由于初始化值的不同，同一个词最终学习到的两组词向量可能不同。当所有词向量学习得到后，GloVe 使用一个词的中心词向量与背景词向量之和作为该词的最终词向量。

## fastText
fastText在使用负采样的跳字模型基础上，将每个中心词视为子词（subword）的集合，并学习子词的词向量。


以 where 这个词为例，设子词为 3 个字符，它的子词包括 “&lt;wh”、“whe”、“her”、“ere”、“re&gt;” 和特殊子词（整词）“&lt;where&gt;”。其中的 “&lt;” 和 “&gt;” 是为了将作为前后缀的子词区分出来。而且，这里的子词 “her” 与整词 “&lt;her&gt;” 也可被区分。给定一个词 $w$ ，我们通常可以把字符长度在 3 到 6 之间的所有子词和特殊子词的并集 $\mathcal{G}_w$ 取出。假设词典中任意子词 $g$ 的子词向量为 $\mathbf{z}_g$，我们可以把使用负采样的跳字模型的损失函数


$$ - \text{log} \mathbb{P} (w_o \mid w_c) = -\text{log} \frac{1}{1+\text{exp}(-\mathbf{u}_o^\top \mathbf{v}_c)}  - \sum_{k=1, w_k \sim \mathbb{P}(w)}^K \text{log} \frac{1}{1+\text{exp}(\mathbf{u}_{i_k}^\top \mathbf{v}_c)} $$

直接替换成

$$ - \text{log} \mathbb{P} (w_o \mid w_c) = -\text{log} \frac{1}{1+\text{exp}(-\mathbf{u}_o^\top \sum_{g \in \mathcal{G}_{w_c}} \mathbf{z}_g)}  - \sum_{k=1, w_k \sim \mathbb{P}(w)}^K \text{log} \frac{1}{1+\text{exp}(\mathbf{u}_{i_k}^\top \sum_{g \in \mathcal{G}_{w_c}} \mathbf{z}_g)} $$

我们可以看到，原中心词向量被替换成了中心词的子词向量的和。与整词学习（word2vec 和 GloVe）不同，词典以外的新词的词向量可以使用 fastText 中相应的子词向量之和。

fastText 对于一些语言较重要，例如阿拉伯语、德语和俄语。例如，德语中有很多复合词，例如乒乓球（英文 table tennis）在德语中叫 “Tischtennis”。fastText 可以通过子词表达两个词的相关性，例如 “Tischtennis” 和 “Tennis”。

## 困惑度（Perplexity）
文章最后替一下语言模型的损失函数。在语言模型中，损失函数即被预测字符的对数似然平均值的相反数：

$$\text{loss} = -\frac{1}{N} \sum_{i=1}^N \log p_{\text{target}_i}$$

其中 $N$ 是预测的字符总数，$p_{\text{target}_i}$ 是在第 $i$ 个预测中真实的下个字符被预测的概率。

而这里的困惑度可以简单的认为就是对交叉熵做 exp 运算使得数值更好读。

为了解释困惑度的意义，我们先考虑一个完美结果：模型总是把真实的下个字符的概率预测为 1 。也就是说，对任意的 $i$ 来说，$p_{\text{target}_i} = 1$ 。这种完美情况下，困惑度值为 **1** 。

我们再考虑一个基线结果：给定不重复的字符集合 $W$ 及其字符总数 $|W|$，模型总是预测下个字符为集合 $W$ 中任一字符的概率都相同。也就是说，对任意的 $i$ 来说，$p_{\text{target}_i} = 1/|W|$ 。这种基线情况下，困惑度值为 **$|W|$** 。

最后，我们可以考虑一个最坏结果：模型总是把真实的下个字符的概率预测为 0 。也就是说，对任意的 $i$ 来说，$p_{\text{target}_i} = 0$ 。这种最坏情况下，困惑度值为**正无穷**。

任何一个有效模型的困惑度值必须小于预测集中元素的数量。在本例中，困惑度必须小于字典中的字符数 $|W|$ 。如果一个模型可以取得较低的困惑度的值（更靠近 1 ），通常情况下，该模型预测更加准确。

[1]: wordscatter.png
[2]: hierarchical_softmax.svg
