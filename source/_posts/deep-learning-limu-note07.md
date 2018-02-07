---
title: MXNet/Gluon 深度学习笔记 (七) —— 样式迁移
date: 2018-02-06 16:54:01
tags: DeepLearning
mathjax: true
---
今天我们要讲的是利用神经网络来“修图”，即我们通过神经网络，将特定的照片风格从一张图片移植到另一张图片，这个过程我们叫做**样式迁移**。如下图所示：
![neural-style][1]
<!--more-->

## 原理
神经网络的这一应用最先由 [Gatys等人](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf) 开创，思路是通过匹配卷积神经网络的中间层输出来训练出合成图片，流程如下所示：

![neural-style2][2]

- 首先挑选一个卷积神经网络来提取特征，选择它的特定层来匹配样式，特定层来匹配内容。示意图中假设选择层 1,2,4 作为样式层，层 3 作为内容层；
- 输入样式图片并保存样式层输出，记第 $i$ 层输出为 $s_i$
- 输入内容图片并保存内容层输出，记第 $i$ 层输出为 $c_i$
- 初始化合成图片 $x$ 为随机值或者其他更好的初始值。然后进行迭代使得用 $x$ 抽取的特征能够匹配上 $s_i$ 和 $c_i$。迭代如下步骤直至收敛：
  - 输入 $x$ 计算样式层和内容层输出，记第 $i$ 层输出为 $y_i$
  - 使用样式损失函数来计算 $y_i$ 和 $s_i$ 的差异
  - 使用内容损失函数来计算 $y_i$ 和 $c_i$ 的差异
  - 对损失求和并对输入 $x$ 求导，记导数为 $g$
  - 更新 $x$， 例如 $x = x - \eta g$

  内容损失函数通常使用回归中用到的均方误差。对于样式，我们可以将它看成是像素点在每个通道的**统计分布**。例如要匹配两张图片的颜色，我们的一个做法是匹配这两张图片在 RGB 这三个通道上的直方图。更一般的，假设卷积层的输出格式是 $c \times h \times w$，既 `channels x height x width`。那么我们可以把它变形成 $c \times hw$ 的 2D 数组，并将它看成是一个维度为 $c$ 的随机变量采样到的 $hw$ 个点。所谓的样式匹配就是使得两个 $c$ 维随机变量统计分布一致。

  匹配统计分布常用的做法是**冲量匹配**，就是说使得他们有一样的均值，协方差，和其他高维的冲量（三阶矩等）。为了计算简单起见，这里假设卷积输出已经是均值为 0 了，而且我们只匹配协方差。也就是说，样式损失函数就是对 $s_i$ 和 $y_i$ 计算 Gram 矩阵然后应用均方误差

  $$ \textrm{styleloss}(s_i, y_i) = \frac{1}{c^2hw} \| s_i s_i^T - y_i y_i^T \|\_F $$

  这里假设 $s_i$ 和 $y_i$ 已经变形成 $c \times hw$ 的 2D 矩阵。

  接下来我们使用 gluon 来实现简单的“样式迁移”。

## 实验
### 数据准备
我们使用如下的水墨风格的橡树图作为样式图片，实拍的松树图作为内容图片，我们最终要的效果是得到一张水墨风格的松树图片。
样式图片：
![autumn_oak][3]

内容图片：
![pine-tree][4]
我们事先定义图像预处理函数与后处理函数，预处理函数将原始图像进行归一化并转换成神经网络接受的输入格式，后处理函数将网络的输出还原成图片格式。

```python
from mxnet import nd

rgb_mean = nd.array([0.485, 0.456, 0.406])
rgb_std = nd.array([0.229, 0.224, 0.225])

def preprocess(img, image_shape):
    img = image.imresize(img, *image_shape)
    img = (img.astype('float32')/255 - rgb_mean) / rgb_std
    return img.transpose((2,0,1)).expand_dims(axis=0)

def postprocess(img):
    img = img[0].as_in_context(rgb_std.context)
    return (img.transpose((1,2,0))*rgb_std + rgb_mean).clip(0,1)
```
>`rgb_std` 和 `rgb_mean` 的数值是由 ImageNet 数据库 统计而得，用这两个数据归一化图片更具有普适性。

### 模型
我们使用原论文使用的VGG 19模型。并下载在Imagenet上训练好的权重。
```python
from mxnet.gluon.model_zoo import vision as models

pretrained_net = models.vgg19(pretrained=True)
```
该网络的结构如下图所示：
![VGG19][5]
上图中，每两个卷积层加 relu 层构成一个卷积块，卷积块与卷积块之间使用 MaxPool2D 层进行分割。

我们有很多种选择来决定使用那些层作为样式层，哪些层作为内容匹配层。通常越靠近输入层越容易匹配内容和样式的细节信息，越靠近输出则越倾向于匹配语义的内容和全局的样式。这里我们按照原论文中使用每个卷积块的第一个卷积层输出来匹配样式，第四个块中的最后一个卷积层来匹配内容，即有如下数据：

```python
style_layers = [0,5,10,19,28]
content_layers = [25]
```
使用我们需要的 VGG 中间层来构建一个新的网络：
```python
from mxnet.gluon import nn

def get_net(pretrained_net, content_layers, style_layers):
    net = nn.Sequential()
    for i in range(max(content_layers+style_layers)+1):
        net.add(pretrained_net.features[i])
    return net

net = get_net(pretrained_net, content_layers, style_layers)
```
给定输入 x，使用 net(x) 只能拿到最后的输出，但是由上分析可知我们还需要net的中间层输出，因此我们逐层计算，并保留需要的输出：
```python
def extract_features(x, content_layers, style_layers):
    contents = []
    styles = []
    for i in range(len(net)):
        x = net[i](x)
        if i in style_layers:
            styles.append(x)
        if i in content_layers:
            contents.append(x)
    return contents, styles
```
### 损失函数
由上分析可知损失函数由两部分组成，一部分是内容匹配的损失，这是一个经典的回归问题，因此我们使用均方误差来计算内容匹配的损失：
```python
def content_loss(yhat, y):
    return (yhat-y).square().mean()
```
另一部分为样式匹配损失，我们通过 Gram 矩阵来计算它：
```python
def gram(x):
    c = x.shape[1]
    n = x.size / x.shape[1]
    y = x.reshape((c, int(n)))
    return nd.dot(y, y.T) / n

def style_loss(yhat, gram_y):
    return (gram(yhat) - gram_y).square().mean()
```

然而在实际实验中，当使用靠近输出层的高层输出来拟合时，经常可以观察到学到的图片里面有大量高频噪音。这个有点类似老式天线电视机经常遇到的白噪音。有多种方法来降噪，例如可以加入模糊滤镜，或者使用[总变差降噪（Total Variation Denoising）](https://en.wikipedia.org/wiki/Total_variation_denoising)。

![](https://upload.wikimedia.org/wikipedia/en/e/e8/ROF_Denoising_Example.png)

假设 $x_{i,j}$ 表示像素 $(i,j)$，那么我们加入下面的损失函数，它使得邻近的像素值相似：

$$
\sum_{i,j} |x_{i,j} - x_{i+1,j}| + |x_{i,j} - x_{i,j+1}|
$$

```python
def tv_loss(yhat):
    return 0.5*((yhat[:,:,1:,:] - yhat[:,:,:-1,:]).abs().mean() +
                (yhat[:,:,:,1:] - yhat[:,:,:,:-1]).abs().mean())
```
因此，总损失函数应该是上诉三种损失的加权和。通过调整权重值我们可以控制学到的图片是否保留更多样式，更多内容，还是更加干净。注意到样式匹配中我们使用了5个层的输出，这里我们对靠近输入的层给予比较大的权重。
```python
channels = [net[l].weight.shape[0] for l in style_layers]
style_weights = [1e4/n**2 for n in channels]
content_weights = [1]
tv_weight = 10

def sum_loss(loss, preds, truths, weights):
    return nd.add_n(*[w*loss(yhat, y) for w, yhat, y in zip(
        weights, preds, truths)])
```
### 训练
首先我们定义两个函数，他们分别对源内容图片和源样式图片提取特征。
```python
def get_contents(image_shape):
    content_x = preprocess(content_img, image_shape).copyto(ctx)
    content_y, _ = extract_features(content_x, content_layers, style_layers)
    return content_x, content_y

def get_styles(image_shape):
    style_x = preprocess(style_img, image_shape).copyto(ctx)
    _, style_y = extract_features(style_x, content_layers, style_layers)
    style_y = [gram(y) for y in style_y]
    return style_x, style_y
```
训练过程跟传统的神经网络训练的主要的主要不同在于

- 损失函数更加复杂。
- 我们只对输入进行更新，这个意味着我们需要对输入 `x` 预先分配了梯度。
- 我们可能会替换匹配内容和样式的层，和调整他们之间的权重，来得到不同风格的输出。这里我们对梯度做了一般化，使得不同参数下的学习率不需要太大变化。
- 仍然使用简单的梯度下降，但每 *n* 次迭代我们会减小一次学习率

```python
from time import time
from mxnet import autograd

def train(x, max_epochs, lr, lr_decay_epoch=200):
    tic = time()
    for i in range(max_epochs):
        with autograd.record():
            content_py, style_py = extract_features(
                x, content_layers, style_layers)
            content_L  = sum_loss(
                content_loss, content_py, content_y, content_weights)
            style_L = sum_loss(
                style_loss, style_py, style_y, style_weights)
            tv_L = tv_weight * tv_loss(x)
            loss = style_L + content_L + tv_L

        loss.backward()
        x.grad[:] /= x.grad.abs().mean()+1e-8
        x[:] -= lr * x.grad
        # add sync to avoid large mem usage
        nd.waitall()

        if i and i % 20 == 0:
            print('batch %3d, content %.2f, style %.2f, '
                  'TV %.2f, time %.1f sec' % (
                i, content_L.asscalar(), style_L.asscalar(),
                tv_L.asscalar(), time()-tic))
            tic = time()

        if i and i % lr_decay_epoch == 0:
            lr *= 0.1
            print('change lr to ', lr)

    plt.imshow(postprocess(x).asnumpy())
    plt.show()
    return x
```
下面开始训练：
```python
import sys
sys.path.append('..')
import utils

image_shape = (1200,800)

ctx = utils.try_gpu()
net.collect_params().reset_ctx(ctx)

content_x, content_y = get_contents(image_shape)
style_x, style_y = get_styles(image_shape)

x = content_x.copyto(ctx)
x.attach_grad()

y = train(x, 500, 0.1,100)
```
最后得到的合成图片如下图所示：
![result][6]

[1]: neural-style.svg
[2]: neural-style2.svg
[3]: autumn_oak.jpg
[4]: pine-tree.jpg
[5]: VGG19.png
[6]: result.png
