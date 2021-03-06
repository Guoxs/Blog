---
title: 《Deep Forest》文献阅读
date: 2017-11-06 00:10:54
tags: DeepLearning
---
近日，西瓜书《机器学习》作者、国内机器学习大牛周志华教授发表了一篇论文，提出了一种基于树的方法——gcForest，挑战深度神经网络。本篇学习笔记为对其论文的解读。

## 深度神经网络的缺陷
深度神经网络的巨大成功掀起了一股深度学习热潮，然而，深度学习取得巨大成就的同时，其不可避免的缺陷也渐渐暴露出来。
深度学习主要的缺陷有以下几个方面：

- 深度神经网络训练时需要大量的数据，无法被运用到小规模的数据任务中；
- 深度神经网络是个非常复杂的模型（complicated models），训练过程中通常需要强大的计算设施(powerful computational facilities)，这导致身处大公司外的个人无法充分发挥其学习的潜力。
- 深度神经网络超参数（hyper-parameters）太多，其学习性能严重依赖于调参的过程。
- 深度神经网络的理论分析很困难，学习的过程就像一个黑箱子。
<!--more-->
## gcForest 相对于深度神经网络的优点
文章中提出了 **gcForest**（multi-Grained Cascade forest，**多粒度级联森林**），以及一种全新的决策树集成方法。这种方法生成一个**深度树集成方法**（deep forest ensemble method），使用级联结构让 gcForest 做表征学习。当输入带有高维度时，通过多粒度扫描，其表征学习能力还能得到进一步的提升，而这有望使 gcForest 能注意到**上下文或结构**（contextual or structural aware）。级联的数量能够根据情况进行调节，从而使 gcForest 在只有小数据的情况下也表现出优异的性能。
相对于深度神经网络，gcForest 有以下优点：

- gcForest 的超参数比深度神经网络少得多；且超参数设定性能鲁棒性相当高；
- 因为基于树模型，故 gcForest 理论分析更加简单；
- gcForest 天然适合用于并行部署，因此效率更高。

## gcForest 方法介绍
gcForest 的整体流程如下：

![整体流程][1]

整个流程包含两大步，第一步是数据的多粒度扫描，第二步是级联森林学习。
假设原始数据有400个原始特征，图示使用三种不同的窗口对其进行扫描，每次向下移动一个特征。对于每个训练样本，100特征的窗口可以产生301 个100维的特征向量，如果有 M 个训练样本，则共有 301*M 个100维的训练样本。这些数据被用来训练一个完全随机森林和一个随机森林，每个森林包含了30棵决策树。
假设结果分为三类，则每个森林会为301个实例中每个实例都生成一个3维向量。 和等和200个特征和300个特征的滑动窗口数据相结合后，原始的400维特征向量被表示成了3618维特征向量，达到了特征向量的增强。

以下为使用滑动窗口扫描的特征重新表示的图示。对于图片等二维数据，用一个 10*10的窗口将产生121个特征向量，其具体实现和多尺寸的滑动窗口和一维数据类似。

![滑动窗口扫描原始数据][4]

随后，M 个 3618维特征向量被传入级联森林。级联森林结构如下图所示：

![级联深林结构图][2]

其中级联中的每一级接收到由前一级处理的特征信息，并将该级的处理结果输出给下一级。每个级是决策树森林的一个集合，即集成的集成（ensemble of ensembles）。使用不同类型的森林是为了增加多样性，因为多样性是集合结构的关键。论文中使用了两个**完全随机的树森林**（complete-random tree forests）和两个**随机森林**，每个完全随机的树森林包含1000个**完全随机树**，通过随机选择一个特征在树的每个节点进行分割实现生成，树一直生长，直到每个叶节点只包含相同类的实例或不超过10个实例。类似地，每个随机森林也包含1000棵树，通过随机选择 $\sqrt{d}$ 数量的特征作为候选（d是输入特征的数量），然后选择具有最佳 **gini 值**的特征作为分割。

给定一个实例，每个森林会通过计算在相关实例落入的叶节点处的不同类的训练样本的百分比，然后对森林中的所有树计平均值，以生成对类的分布的估计。如下图所示，其中红色部分突出了每个实例遍历到叶节点的路径。

![类向量生成图示][3]

被估计的类分布形成**类向量**（class vector），该类向量接着与输入到级联的下一级的原始特征向量相连接。例如，假设有三个类，则四个森林每一个都将产生一个三维的类向量，因此，级联的下一级将接收12 = 3×4个增强特征（augmented feature）。

为了降低过拟合风险，每个森林产生的类向量由**k折交叉验证**（k-fold cross validation）产生。每个实例都将被用作 k − 1 次训练数据，产生 k − 1 个类向量，然后对其取平均值以产生作为级联中下一级的增强特征的最终类向量。在扩展一个新的级后，整个级联的性能将在验证集上进行估计，如果没有显着的性能增益，训练过程将终止；因此，级联中级的数量是自动确定的。与模型的复杂性固定的大多数深度神经网络相反，gcForest 能够适当地通过终止训练来决定其模型的复杂度。这使得 gcForest 能够适用于不同规模的训练数据，而不局限于大规模训练数据。

## 总结
正如文字中最后所说的：
>There are other possibilities to construct deep forest. As a seminar study, we have only explored a little in this direction. If we had stronger computational facilities, we would like to try big data and deeper forest, which is left for future work. In principle, deep forest should be able to exhibit other powers of deep neural networks, such as serving as feature extractor or pre-trained model. **It is worth mentioning that in order to tackle complicated tasks, it is likely that learning models have to go deep.** Current deep models, however, are always neural networks. This paper illustrates how to construct deep forest, and we believe it may open a door towards alternative to deep neural networks for many tasks.

要解决复杂的问题，学习模型需要往深了走。然而当前的深度模型全部都是神经网络，其实，除了神经网络，还有很多深度学习的方法等待着去发掘。

[1]: http://7xjzhz.com1.z0.glb.clouddn.com/Deep%20Forest4.jpg
[2]: http://7xjzhz.com1.z0.glb.clouddn.com/Deep%20Forest1.jpg
[3]: http://7xjzhz.com1.z0.glb.clouddn.com/Deep%20Forest2.jpg
[4]: http://7xjzhz.com1.z0.glb.clouddn.com/Deep%20Forest3.jpg
