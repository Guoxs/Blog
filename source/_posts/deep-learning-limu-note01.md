---
title: MXNet/Gluon 深度学习笔记（第一课）
date: 2018-01-20 13:03:20
tags: DeepLearning
mathjax: true
---

如何在远端开启 jupyter notebook 服务，而在本地访问？

可以把远端的端口映射到本地，让浏览器能够在本地打开 notebook。
先在远端运行 jupyter notebook，然后使用 `ssh` 将远端的jupyter notebook 端口映射到本地的未使用的端端口

```
ssh -L8008:localhost:8888 remote-ip
```
其中 `8008` 为本地未使用的一个端口号，`8888` 为在远端开启 jupyter notebook 时的默认端口号。

<!--more-->
## NDArray 处理数据
NDArray 是 MXNet 存储和变换数据的主要工具，它和 Numpy 的多维数组非常相似。

创建数组，它的元素服从均值0标准差1的正态分布：
```
y = nd.random_normal(0, 1, shape=(3, 4))
```
**广播（Broadcasting）**
当二元操作符左右两边ndarray形状不一样时，系统会尝试将其复制到一个共同的形状。例如a的第0维是3, b的第0维是1，那么a+b时会将b沿着第0维复制3遍：
```
a = nd.arange(3).reshape((3,1))
b = nd.arange(2).reshape((1,2))
print('a:', a)
print('b:', b)
print('a+b:', a+b)

a: 3x1
[[ 0.]
 [ 1.]
 [ 2.]]

 b: 1x2
[[ 0.  1.]]

a+b: 3x2
[[ 0.  1.]
 [ 1.  2.]
 [ 2.  3.]]
```
**原位操作**
减少运算消耗的内存
```
nd.elemwise_add(x, y, out=z)
```
## autograd 自动求导
mxnet 中进行求导的时候，需要一个地方来存 x 的导数，可以通过 NDArray 的方法 attach_grad() 来要求系统申请对应的空间。
```
x.attach_grad()
```
默认条件下，MXNet 不会自动记录和构建用于求导的计算图，我们需要使用 autograd 里的 record() 函数来显式的要求 MXNet 记录我们需要求导的程序。譬如 $f = 2 \times x^2$
```
with ag.record():
  y = x * 2
  z = y * x
```

>一定要先为待求导的变量分配存储导数的空间 x.attach_grad()，再定义求导函数 with ag.record()，不然会报错。

接下来可以通过 `z.backward()` 来进行求导。如果 z 不是一个标量，那么 `z.backward()` 等价于 `nd.sum(z).backward()`.

## Linear Regression
线性模型

$$y = X \cdot w + b + \eta, \quad \text{for } \eta \sim \mathcal{N}(0,\sigma^2)$$


训练神经网络的时候，网络需要不断读取数据块。可以定义一个函数，每次返回 batch_size 个随机的样本和对应的目标。这个功能可以使用 python 中的 yield 来构造一个迭代器实现：
```
import random
batch_size = 10
def data_iter():
    # 产生一个随机索引
    idx = list(range(num_examples))
    random.shuffle(idx)
    for i in range(0, num_examples, batch_size):
        j = nd.array(idx[i:min(i+batch_size,num_examples)])
        yield nd.take(X, j), nd.take(y, j)

#读取数据
for data, label in data_iter():
  ......
```

gluon 中提供了封装好的函数
```
batch_size = 10
dataset = gluon.data.ArrayDataset(X, y)
data_iter = gluon.data.DataLoader(dataset, batch_size, shuffle=True)

#读取
for data, label in data_iter:
    print(data, label)
```
使用 gluon 训练模型
```
#定义一个空的模型
net = gluon.nn.Sequential()
#加入一个全连接层
net.add(gluon.nn.Dense(1))
#初始化模型参数
net.initialize()
#损失函数
square_loss = gluon.loss.L2Loss()
#优化
trainer = gluon.Trainer(net.collect_params(),'sgd',{learning_rate:0.2})
```

>这里我们不需要定义层的输入节点是多少，节点数在读取数据的时候系统会自动赋值

```
#训练
epochs = 10
batch_size = 15
for e in range(epochs):
    total_loss = 0
    for data, label in data_iter:
        with autograd.record():
            output = net(data)
            loss = square_loss(output, label)
        loss.backward()
        #更新模型，因为拿到的是一个 batch_size 的梯度和，故最后还需除 batch_size
        trainer.step(batch_size)
        total_loss += nd.sum(loss).asscalar()
    print("Epoch %d, average loss: %f" % (e, total_loss/num_examples))
```
可从 net 中拿到需要的层，然后访问其权重和位移
```
dense = net[0]
true_w, dense.weight.data()
true_b, dense.bias.data()
#拿到梯度
dense.weight.grad()
```

>Tips: 可通过 `help(functionName)` 来从 jupyter notebook 中查看函数的文档，通过 `functionName??` 可直接调出函数的源码

## Softmax Regression
Softmax 函数
```
def softmax(X):
    exp = nd.exp(X)
    # 假设exp是矩阵，这里对行进行求和，并要求保留axis 1，
    # 就是返回 (nrows, 1) 形状的矩阵
    partition = exp.sum(axis=1, keepdims=True)
    return exp / partition
```
>注意，这样实现的 softmax 在后面求损失值时可能出现数值越界问题，解决方法详见博文末尾。


定义模型：
```
def net(X):
    return softmax(nd.dot(X.reshape((-1,num_inputs)), W) + b)
```
>这里 X.reshape() 第一个参数为 `-1` 表示该值可以由已知条件（这里是 num_inputs）推导出来

**交叉熵损失函数**
这是针对概率值得损失函数，它将两个概率分布的负交叉熵作为目标值，最小化这个值等价于最大化这两个概率的相似度。

$$J(\theta) = - \frac{1}{m} \sum^m_{i=1} y^{(i)}log(h_{\theta}(x^{(i)})) + (1 - y^{(i)})log(1 - h_{\theta}(x^{(i)}))$$

具体来说，我们先将真实标号表示成一个概率分布，例如如果 y=1，那么其对应的分布就是一个除了第二个元素为1其他全为 0 的长为 10 的向量，也就是 yvec = [0, 1, 0, 0, 0, 0, 0, 0, 0, 0]。那么交叉熵就是 yvec[0]\*log(yhat[0])+...+yvec[n]\*log(yhat[n])。注意到 yvec 里面只有一个 1，那么前面等价于 log(yhat[y])。所以我们可以定义这个损失函数了

```
def cross_entropy(yhat, y):
    return - nd.pick(nd.log(yhat), y)
```

gluon提供一个将这两个函数合起来的数值更稳定的版本
```
softmax_cross_entropy = gluon.loss.SoftmaxCrossEntropyLoss()
```


**预测概率最高的类**
```
def accuracy(output, label):
    return nd.mean(output.argmax(axis=1)==label).asscalar()

    def evaluate_accuracy(data_iterator, net):
        acc = 0.
        for data, label in data_iterator:
            output = net(data)
            acc += accuracy(output, label)
        return acc / len(data_iterator)
```

## Softmax 与数值稳定性
首先，Softmax 函数 $\sigma(z) = (\sigma_1(z),...,\sigma_m(z))$ 定义如下：
$$
\sigma_i(z) = \frac{e^{z_i}}{\sum^m_{j=1}e^{z_j}}, i = 1,...,m
$$

假设 $z_i = \omega^T_ix + b_i$ 是第 $i$ 类别的线性预测结果，则 Softmax 的结果其实就是先对每一个  $z_i$ 取 exponential 变成非负，然后除以所有项之和进行归一化。$\sigma_i(z)$ 可以解释为观察到的数据 $x$ 属于类别 $i$ 的概率， 或者称为似然(Likelihood)。

对这个函数求导的过程是这样的：
当 $i = j$ 时
$$
\frac{\partial y_i}{\partial z_j} = \frac{\partial \frac{e^{z_i}}{\sum^m_{j=1}e^{z_j}}}{\partial z_j} = \frac{e^{z_i}\sum - e^{z_i} e^{z_j}}{\sum^2} = \frac{e^{z_i}}{\sum} \frac{\sum - e^{z_j}}{\sum} = y_i(1-y_j)
$$

当 $i \neq j$ 时
$$
\frac{\partial y_i}{\partial z_j} = \frac{\partial \frac{e^{z_i}}{\sum^m_{j=1}e^{z_j}}}{\partial z_j} = \frac{0 - e^{z_i} e^{z_j}}{\sum^2} = \frac{e^{z_i}}{\sum} \frac{e^{z_j}}{\sum} = y_iy_j
$$

其中 $\sum = \sum^m_{j=1} e^{z_j}$

上面我们用 python 实现的 softmax 函数为：
```
def softmax(X):
  exp = nd.exp(X)
  partition = exp.sum(axis=1, keepdims=True)
  return exp / partition
```
注意到，当 x 很大时，exp(x) 会出现溢出的现象。一个简单的方法就是 x 乘以一个小的常数，将其缩放到一个合适的值。
$$
y_i = \frac{e^{z_i}}{\sum^m_{j=1}e^{z_j}} = \frac{Ee^{z_i}}{\sum^m_{j=1}Ee^{z_j}} = \frac{e^{z_i + log(E)}}{\sum^m_{j=1}e^{z_j +  log(E)}} = \frac{e^{z_i + F}}{\sum^m_{j=1}e^{z_j +  F}}
$$
其中，常数 $ F = -max(z_1,...,z_m)$ 可将所有值放缩在 0 附近。

但即使解决了 exp(x) 的数值溢出，在求导数的阶段还是有可能出现数值溢出的情况，一个更好的方法是使用 softmax-loss，详见 [Softmax vs. Softmax-Loss: Numerical Stability][2]

参考：

[Softmax函数与交叉熵][1]

[Softmax vs. Softmax-Loss: Numerical Stability][2]

[1]: https://zhuanlan.zhihu.com/p/27223959
[2]: http://freemind.pluskid.org/machine-learning/softmax-vs-softmax-loss-numerical-stability/
