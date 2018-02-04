---
title: MXNet/Gluon 深度学习笔记 (五) —— 图像增强与迁移学习
date: 2018-01-30 20:47:56
tags: DeepLearning
mathjax: true
---
## 图像增强
图像增强是通过一系列的随机变化生成大量“新”样本，从而减低过拟合的可能的技术。现在在深度神经网络中，特别是当训练数据量不充足时，图像增强是必不可少的一部分。

常用的图像增强方法有以下几种：

- **变形**：水平方向翻转图片是最早也是最广泛使用的一种增广
- **随机裁剪**：卷积层对目标位置敏感
- **颜色变化**：一般有改变亮度，调整色调等


>随机截取一般会缩小输入的形状，如果原始输入图片过小，导致没有太多空间进行随机裁剪，通常做法是先将其放大的足够大的尺寸。所以如果你的原始图片足够大，建议不要事先将它们裁到网络需要的大小。

实验时通常会将数个增强方法一起用，图像增强通常只增对训练数据，对于测试数据则用得较小。后者常用的是做5次随机剪裁，然后将5张图片的预测结果做均值。
<!--more-->
用 gluon 做数据增强, 可以先定义一个辅助函数:
```python
def apply_aug_list(img, augs):
    for f in augs:
        img = f(img)
    return img
```
然后该定义图像增强内容:
```python
train_augs = [
    image.HorizontalFlipAug(flip_prob),
    image.RandomCropAug((height,width))
]

test_augs = [
    image.CenterCropAug((height,width))
]
```
最后应用到图像:
```python
## apply to each sample one-by-one and then stack
data = nd.stack(*[apply_aug_list(d, augs) for d in data])
```
## 迁移学习
总所周知,训练神经网络需要庞大的数据集, 而数据集的制作绝非易事. 对于我们大多数人而言, 需要在自己的数据集上训练数据集, 而通常我们所能获得的只是相对而言中等规模的数据, 几百张图片很正常，找到几千张图片也有可能，但很难同 Imagenet 一样获得上百万张图片。

于是一个很自然的问题就是, 如何使用在百万张图片上训练出来的强大的模型来帮助提升在小数据集上的精度呢？这种在源数据上训练，然后将学到的知识应用到目标数据集上的技术通常被叫做**迁移学习**。

对于深度神经网络来首，最为流行的一个方法叫做微调（fine-tuning）。它的想法很简单但有效：


* 在源数据 $S$ 上训练一个神经网络。
* 砍掉它的头，将它的输出层改成适合目标数据 $S$ 的大小
* 将输出层的权重初始化成随机值，但其它层保持跟原先训练好的权重一致
* 然后开始在目标数据集开始训练

该过程如下图所示:

<div align = center>
  <img src = "./fine-tuning.svg"/>
  <p>fine-tuning</p>
</div>

在 gluon 中, 使用迁移学习也很简单.首先, 我们从模型库中获取改良过的 ResNet, 使用 `pretrained = True` 将会自动下载并加载从 ImageNet 数据集上训练而来的权重。

```python
from mxnet.gluon.model_zoo import vision as models

pretrained_net = models.resnet18_v2(pretrained=True)
```
通常预训练好的模型由两块构成，一是 `features`，二是 `output`。后者主要包括最后一层全连接层，前者包含从输入开始的大部分层。这样的划分的一个主要目的是为了更方便做微调。
output 的内容如下：
```python
pretrained_net.output
>> Dense(512 -> 1000, linear)
```
卷积层的部分权重可以通过如下方式查看:
```python
pretrained_net.features[1].weight.data()[0][0]
```
微调时, 一般新建一个网络，它的定义跟之前训练好的网络一样，除了最后的输出数等于当前数据的类别数。新网络的 `features` 被初始化前面训练好网络的权重，而 `output` 则是从头开始训练。

```python
from mxnet import init

finetune_net = models.resnet18_v2(classes=2)
finetune_net.features = pretrained_net.features
finetune_net.output.initialize(init.Xavier())
```

通过一个预先训练好的模型，我们可以在即使较小的数据集上训练得到很好的分类器, 这是因为这两个任务里面的数据表示有很多**共通性**，例如都需要如何识别纹理、形状、边等等, 而这些特征通常能被靠近数据的层有效的捕捉。因此，如果我们有一个相对较小的数据，而且担心它可能不够训练出很好的模型，那么我们可以寻找与我们数据类似的大数据集来先预先训练网络模型，然后再使用小数据集进行微调。
