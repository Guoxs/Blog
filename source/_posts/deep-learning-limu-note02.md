---
title: MXNet/Gluon 深度学习笔记 (二)
date: 2018-01-22 15:33:38
tags: DeepLearning
mathjax: true
---
本篇学习笔记主要介绍了正则化的贝叶斯解释、过拟合、Dropout、K 折交叉验证以及 MXNet 中 GPU 的使用。

## 正则化的贝叶斯解释
计算损失函数时加入　$L_2$ 范数正则化，那么最小化损失函数时实际上是在最小化：

$$loss + \lambda \sum_{p \in params} ||p||^2_2$$

然而这个正则化项也可以在贝叶斯方法中得到解释。

统计学中有两个学派，一派叫做　Frequentiet （频率派），一派叫做　Bayesian （贝叶斯学派）。以线性回归为例，假设　$y_i = \omega x_i + noise$ ，noise 服从正态分布，均值 0 方差为 $\sigma^2$.

从贝叶斯的角度：
假设 $\omega$ 的 prior 是高斯 prior: $\omega \sim N(0, 1/\lambda)$，这里的 N 为高斯（正态）分布，因为有 MAP = ML * Proir (MAP: 最大先验概率，ML: 似然函数最大值)，所以由最大先验概率估计有：
$$
\omega = argmax_{\omega} \\,\\, \mathcal{ln}  \\,\\, \Pi^n_i \frac{1}{\sigma \sqrt(2\pi)} exp(-\frac{1}{2}(y_i-\omega^TX_i)^2)
  \\,\\, \Pi^n_j \frac{1}{\tau \sqrt(2\pi)} exp(-\frac{1}{2}(\frac{\omega_j}{\tau})^2)\\
 = -\frac{1}{2\sigma^2}\sum^n_i(y_i-\omega^TX_i)^2 - \frac{1}{2\tau^2}\sum^n_i\omega^2 - nln\sigma\sqrt{2\pi} - nln\tau\sqrt{2\pi}
$$
<!--more-->
去掉不影响估计 $\omega$ 的常数项，得：

$$
\omega = argmax_{\omega}  \\,\\, \sum^n_i-(y_i-\omega^TX_i)^2 - \frac{\tau^2}{\sigma^2}\sum^n_j\omega^2
$$

把负号去掉，即求 $\omega$ 也就是最小化 $\sum^n_i (y_i-\omega^TX_i)^2 + \lambda||\omega||^2$ (其中 $\lambda = \frac{\tau^2}{\sigma^2}$)

这就和频率学派说的直接最小化损失函数（最大似然估计）$\sum^n_i(y_i-\omega^T X_i)^2$ 然后再在后面加个 L2 范数正则化 $\lambda \omega^2$ 一样了。

故 $\lambda$ 来自贝叶斯先验，如果为 0 则没有先验。

>L2 正则化对应着高斯分布下的后验最大估计，L1 正则化对应着先验服从拉普拉斯分布下的后验最大估计。详见 [Regularized Regression: A Bayesian point of view][1]

## 如何应对过拟合现象
过拟合（overfitting）是指在模型参数拟合过程中的问题，由于训练数据包含**抽样误差**，训练时，复杂的模型将抽样误差也考虑在内，将抽样误差也进行了很好的拟合。 具体表现就是最终模型在训练集上效果好，在测试集上效果差，模型泛化能力弱。

可以通过以下方法防止过拟合：

- 获取更多数据
  - 从数据源头获取更多数据
  - 根据当前数据集估计数据分布参数，使用该分布生成更多的数据
  - 数据增强
- 使用合适的模型
  - 降低网络复杂度，比如减少网络层数，神经元个数等限制网络的拟合能力
  - 限制训练时间 （Early stoping）
  - 正则化
  - 增加噪音，可以在输入、权值和网络的响应上增加噪音
- 结合多种模型
  - Bagging, 用不同模型拟合不同部分的训练集
  - Boosting, 通过训练一系列简单的神经网络，加权平均其输出
  - Dropout, 训练是每次随机忽略隐层的默写节点，类似于集成了多个小模型
- 贝叶斯方法

参考：[机器学习中用来防止过拟合的方法有哪些][2]

## Dropout
Dropout 是一种常见的应对过拟合的方法，通常是对输入层或者隐含层做以下操作：

- 随机选择一部分该层的输出作为丢弃元素
- 把丢弃元素乘以0
- 把非丢弃元素拉伸

实现如下：
```python
from mxnet import nd
def dropout(X, drop_probability):
    keep_probability = 1 - drop_probability
    assert 0 <= keep_probability <= 1
    # 这种情况下把全部元素都丢弃。
    if keep_probability == 0:
        return X.zeros_like()

    # 随机选择一部分该层的输出作为丢弃元素。
    mask = nd.random.uniform(
        0, 1.0, X.shape, ctx=X.context) < keep_probability
    # 保证 E[dropout(X)] == X
    scale =  1 / keep_probability
    return mask * X * scale
```
Dropout 实际上是在模拟集成学习。我们在训练神经网络模型时一般随机采样一个批量的训练数据。Dropout 实质上是对每一个这样的数据集分别训练一个原神经网络子集的分类器。与一般的集成学习不同，这里每个原神经网络子集的分类器用的是同一套参数。因此丢弃法只是在模拟集成学习。

Dropout 神经网络子集的分类器在不同的训练数据批量上训练并使用同一套参数，因此，使用丢弃法的神经网络实质上是对输入层和隐含层的参数做了正则化：学到的参数使得原神经网络不同子集在训练数据上都尽可能表现良好。

注意， Dropout 只在训练的时候使用，在测试的时候不需要随机失活，但是对于两个隐层的输出都要乘以 p，调整其数值范围。因为在测试时所有的神经元都能看见它们的输入，因此我们想要神经元的输出与训练时的预期输出是一致的。基于这一点，实际上推荐使用 **反向随机失活（invert dropout）**，在训练时就进行数值范围调整，从而让前向传播在测试时保持不变，这也是上面实现中保证 E[dropout(X)] == X 之后代码的作用。

## K折交叉验证
过度依赖训练数据集的误差来推断测试数据集的误差容易导致过拟合。事实上，当我们调参时，往往需要基于K折交叉验证。

在 K 折交叉验证中，我们把初始采样分割成 K 个子样本，一个单独的子样本被保留作为验证模型的数据，其他 K−1 个样本用来训练，K 种不同验证子样本共训练 K 次，然后取 K 次验证模型的测试结果的平均值和训练误差的平均值。

实现如下：
```python
def k_fold_cross_valid(k, epochs, verbose_epoch, X_train, y_train,
                       learning_rate, weight_decay):
    assert k > 1
    fold_size = X_train.shape[0] // k
    train_loss_sum = 0.0
    test_loss_sum = 0.0
    for test_i in range(k):
        X_val_test = X_train[test_i * fold_size: (test_i + 1) * fold_size, :]
        y_val_test = y_train[test_i * fold_size: (test_i + 1) * fold_size]

        val_train_defined = False
        for i in range(k):
            if i != test_i:
                X_cur_fold = X_train[i * fold_size: (i + 1) * fold_size, :]
                y_cur_fold = y_train[i * fold_size: (i + 1) * fold_size]
                if not val_train_defined:
                    X_val_train = X_cur_fold
                    y_val_train = y_cur_fold
                    val_train_defined = True
                else:
                    X_val_train = nd.concat(X_val_train, X_cur_fold, dim=0)
                    y_val_train = nd.concat(y_val_train, y_cur_fold, dim=0)
        net = get_net()
        train_loss, test_loss = train(
            net, X_val_train, y_val_train, X_val_test, y_val_test,
            epochs, verbose_epoch, learning_rate, weight_decay)
        train_loss_sum += train_loss
        print("Test loss: %f" % test_loss)
        test_loss_sum += test_loss
    return train_loss_sum / k, test_loss_sum / k
```
## 使用 GPU
Linux 下使用 `!nvidia-smi` 可查看 GPU状态。

**Context**
MXNet 使用 Context 来指定使用哪个设备来存储和计算。默认会将数据开在主内存，然后利用 CPU 来计算，这个由 `mx.cpu()` 来表示。GPU 则由 `mx.gpu()` 来表示。注意 `mx.cpu()` 表示所有的物理 CPU 和内存，意味着计算上会尽量使用多有的 CPU 核。但 `mx.gpu()` 只代表一块显卡和其对应的显卡内存。如果有多块 GPU，用 `mx.gpu(i)` 来表示第 i 块 GPU（ i 从 0 开始）。

**创建内存**
```python
x = nd.array([1,2,3], ctx=mx.gpu())
```

通过 `copyto` 和 `as_in_context` 来在设备直接传输数据。
```python
y = x.copyto(mx.gpu())
z = x.as_in_context(mx.gpu())
```
>这两个函数的主要区别是，如果源和目标的 context 一致，`as_in_context` 不复制，而 `copyto` 总是会新建内存。

**计算**

计算会在数据的 context 上执行。所以为了使用 GPU，只需要事先将数据放在上面就行了。结果会自动保存在对应的设备上。注意 MXNet 中所有计算要求输入数据在同一个设备上，不一致的时候系统不进行自动复制。这个设计的目的是因为设备之间的数据交互通常比较昂贵，作者希望用户确切的知道数据放在哪里，而不是隐藏这个细节。

如果某个操作需要将 GPU 里面的内容转出来，例如打印或变成 numpy 格式，如果需要的话系统都会自动将数据 copy 到主内存。

Gluon的大部分函数可以通过ctx指定设备。下面代码将模型参数初始化在GPU上：
```python
from mxnet import gluon
net = gluon.nn.Sequential()
net.add(gluon.nn.Dense(1))
net.initialize(ctx=mx.gpu())
```
### 多 GPU 的使用
**数据并行**

数据并行目前是深度学习里面使用最广泛的用来将任务划分到多设备的办法。它是这样工作的：假设这里有 k 个 GPU，每个 GPU 将维护一个模型参数的复制。然后每次我们将一个批量里面的样本划分成 k 块并分每个 GPU 一块。每个 GPU 使用分到的数据计算梯度。然后我们将所有 GPU 上梯度相加得到这个批量上的完整梯度。之后每个 GPU 使用这个完整梯度对自己维护的模型做更新。

**在多GPU之间同步数据**
用一个实例介绍

将模型参数复制到某个特定设备并初始化梯度：
```python
from mxnet import gpu

def get_params(params, ctx):
    new_params = [p.copyto(ctx) for p in params]
    for p in new_params:
        p.attach_grad()
    return new_params
```
给定分布在多个 GPU 之间数据，定义一个函数它将这些数据加起来，然后再广播到所有 GPU 上：
```python
def allreduce(data):
    # sum on data[0].context, and then broadcast
    for i in range(1, len(data)):
        data[0][:] += data[i].copyto(data[0].context)
    for i in range(1, len(data)):
        data[0].copyto(data[i])
```
最后给定一个批量，我们划分它并复制到各个GPU上：
```python
def split_and_load(data, ctx):
    n, k = data.shape[0], len(ctx)
    m = n // k
    assert m * k == n, '# examples is not divided by # devices'
    return [data[i*m:(i+1)*m].as_in_context(ctx[i]) for i in range(k)]
```
现在我们可以实现如何使用数据并行在多个GPU上训练一个批量了:
```python
from mxnet import autograd
import sys
sys.path.append('..')
import utils

def train_batch(data, label, params, ctx, lr):
    # split the data batch and load them on GPUs
    data_list = split_and_load(data, ctx)
    label_list = split_and_load(label, ctx)
    # run forward on each GPU
    with autograd.record():
        losses = [loss(lenet(X, W), Y)
                  for X, Y, W in zip(data_list, label_list, params)]
    # run backward on each gpu
    for l in losses:
        l.backward()
    # aggregate gradient over GPUs
    for i in range(len(params[0])):
        allreduce([params[c][i].grad for c in range(len(ctx))])
    # update parameters with SGD on each GPU
    for p in params:
        utils.SGD(p, lr/data.shape[0])
```
训练函数：
```python
from time import time

def train(num_gpus, batch_size, lr):
    train_data, test_data = utils.load_data_fashion_mnist(batch_size)

    ctx = [gpu(i) for i in range(num_gpus)]
    print('Running on', ctx)

    # copy parameters to all GPUs
    dev_params = [get_params(params, c) for c in ctx]

    for epoch in range(5):
        # train
        start = time()
        for data, label in train_data:
            train_batch(data, label, dev_params, ctx, lr)
        nd.waitall()
        print('Epoch %d, training time = %.1f sec'%(
            epoch, time()-start))

        # validating on GPU 0
        net = lambda data: lenet(data, dev_params[0])
        test_acc = utils.evaluate_accuracy(test_data, net, ctx[0])
        print('validation accuracy = %.4f'%(test_acc))
```
## 多机器分布式训练

详见：[Set Up a Stack for Distributed Deep Learning Using Apache MXNet][3]



[1]: http://charleshm.github.io/2016/03/Regularized-Regression/
[2]: https://www.zhihu.com/question/59201590/answer/167392763
[3]: https://docs.aws.amazon.com/mxnet/latest/dg/mxnet-on-ec2-cluster.html
