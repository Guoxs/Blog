---
title: MXNet/Gluon 深度学习笔记 (九) —— 循环神经网络
date: 2018-02-16 18:37:06
tags: DeepLearning
mathjax: true
---

传统的前馈神经网络的输入是时间无关的，它可以很方便地提取图像的特征，但是却无法处理时间序列数据，因为它无法捕捉前一个输入与后一个输入之间的联系，而这种联系在时间序列数据中至关重要. 为了处理序列数据，循环神经网络 (RNN) 应运而生．RNN 于 1980 年诞生, 我们知道，一个三层的前馈神经网络可以学到任何的函数，而RNN则是 “turing-complete” 的，它可以逼近任何算法. RNN 每一层不仅输出给下一层，同时还输出一个隐含状态，给当前层在处理下一个样本时使用, 理论上，RNNs能够对任何长度的序列数据进行处理. 下图展示了前馈神经网络与循环神经网络的区别.

![rnn-1][1]

## 循环神经网络 (RNN)
首先回忆一下单隐含层的前馈神经网络的定义，例如**多层感知机**。假设隐含层的激活函数是 $\phi$，对于一个样本数为 $n$ 特征向量维度为 $x$ 的批量数据 $\mathbf{X} \in \mathbb{R}^{n \times x}$（ $\mathbf{X}$ 是一个 $n$ 行 $x$ 列的实数矩阵）来说，那么这个隐含层的输出就是

$$\mathbf{H} = \phi(\mathbf{X} \mathbf{W}_{xh} + \mathbf{b}_h)$$

假定隐含层长度为 $h$，其中的 $\mathbf{W}_{xh} \in \mathbb{R}^{x \times h}$ 是权重参数。偏移参数 $\mathbf{b}_h \in \mathbb{R}^{1 \times h}$ 在与前一项 $\mathbf{X} \mathbf{W}_{xh} \in \mathbb{R}^{n \times h}$ 相加时使用了广播。这个隐含层的输出的尺寸为 $\mathbf{H} \in \mathbb{R}^{n \times h}$。

把隐含层的输出 $\mathbf{H}$ 作为输出层的输入，最终的输出

$$\hat{\mathbf{Y}} = \text{softmax}(\mathbf{H} \mathbf{W}_{hy} + \mathbf{b}_y)$$

假定每个样本对应的输出向量维度为 $y$，其中 $\hat{\mathbf{Y}} \in \mathbb{R}^{n \times y}, \mathbf{W}_{hy} \in \mathbb{R}^{h \times y}, \mathbf{b}_y \in \mathbb{R}^{1 \times y}$ 且两项相加使用了广播。


将上面网络改成循环神经网络，首先对输入输出加上时间戳 $t$。假设 $\mathbf{X}_t \in \mathbb{R}^{n \times x}$ 是序列中的第 $t$ 个批量输入（样本数为 $n$，每个样本的特征向量维度为 $x$ ），对应的隐含层输出是隐含状态 $\mathbf{H}_t  \in \mathbb{R}^{n \times h}$（隐含层长度为 $h$ ），而对应的最终输出是 $\hat{\mathbf{Y}}_t \in \mathbb{R}^{n \times y}$（每个样本对应的输出向量维度为 $y$ ）。在计算隐含层的输出的时候，循环神经网络只需要在前馈神经网络基础上加上跟前一时间 $t-1$ 输入隐含层 $\mathbf{H}_{t-1} \in \mathbb{R}^{n \times h}$ 的加权和。为此，我们引入一个新的可学习的权重 $\mathbf{W}_{hh} \in \mathbb{R}^{h \times h}$：

$$\mathbf{H}_t = \phi(\mathbf{X}_t \mathbf{W}_{xh} + \mathbf{H}_{t-1} \mathbf{W}_{hh}  + \mathbf{b}_h)$$

输出的计算跟前面一致：

$$\hat{\mathbf{Y}}_t = \text{softmax}(\mathbf{H}_t \mathbf{W}_{hy}  + \mathbf{b}_y)$$

隐含状态可以认为是这个网络的记忆。该网络中，时刻 $t$ 的隐含状态就是该时刻的隐含层变量 $\mathbf{H}_t$, 它存储前面时间里面的信息, 我们的输出是只基于这个状态, 最开始的隐含状态里的元素通常会被初始化为 0。

## 通过时间的反向传播
在循环神经网络的训练中，当每个时序训练数据样本的时序长度 `num_steps` 较大或者时刻$t$较小，目标函数有关 $t$ 时刻的隐含层变量梯度较容易出现衰减（valishing）或爆炸（explosion）.

为了应对梯度爆炸，一个常用的做法是如果梯度特别大，那么就投影到一个比较小的尺度上。假设我们把所有梯度接成一个向量 $\boldsymbol{g}$，假设剪裁的阈值是 $\theta$，那么我们这样剪裁使得 $\|\boldsymbol{g}\|$不会超过$\theta$：

$$ \boldsymbol{g} = \min\left(\frac{\theta}{\|\boldsymbol{g}\|}, 1\right)\boldsymbol{g}$$

梯度衰减（valishing）或爆炸（explosion）产生的原因可以由通过时间的反向传播解释.

### 模型定义
给定一个输入为 $\mathbf{x}_t \in \mathbb{R}^x$（每个样本输入向量长度为 $x$ ）和对应真实值为 $y_t \in \mathbb{R}$ 的时序数据训练样本（ $t = 1, 2, \ldots, T$ 为时刻），不考虑偏差项，我们可以得到隐含层变量的表达式

$$\mathbf{h}_t = \phi(\mathbf{W}_{hx} \mathbf{x}_t + \mathbf{W}_{hh} \mathbf{h}_{t-1})$$

其中 $\mathbf{h}_t \in \mathbb{R}^h$ 是向量长度为 $h$ 的隐含层变量，$\mathbf{W}_{hx} \in \mathbb{R}^{h \times x}$ 和 $\mathbf{W}_{hh} \in \mathbb{R}^{h \times h}$ 是隐含层模型参数。使用隐含层变量和输出层模型参数 $\mathbf{W}_{yh} \in \mathbb{R}^{y \times h}$，我们可以得到相应时刻的输出层变量 $\mathbf{o}_t \in \mathbb{R}^y$。不考虑偏差项，

$$\mathbf{o}_t = \mathbf{W}_{yh} \mathbf{h}_{t}$$

给定每个时刻损失函数计算公式 $\ell$，长度为$T$的整个时序数据的损失函数 $L$ 定义为

$$L = \frac{1}{T} \sum_{t=1}^T \ell (\mathbf{o}_t, y_t)$$

这也是模型最终需要被优化的目标函数。

### 计算图
为了可视化模型变量和参数之间在计算中的依赖关系，我们可以绘制计算图。我们以时序长度 T=3 为例。

![rnn-bptt][2]

### 梯度的计算与存储
在上图中，模型的参数是 $\mathbf{W}_{hx}$、$\mathbf{W}_{hh}$ 和 $\mathbf{W}_{yh}$。为了在模型训练中学习这三个参数，以随机梯度下降为例，假设学习率为 $\eta$，我们可以通过

$$\mathbf{W}_{hx} = \mathbf{W}_{hx} - \eta \frac{\partial L}{\partial \mathbf{W}_{hx}}$$

$$\mathbf{W}_{hh} = \mathbf{W}_{hh} - \eta \frac{\partial L}{\partial \mathbf{W}_{hh}}$$

$$\mathbf{W}_{yh} = \mathbf{W}_{yh} - \eta \frac{\partial L}{\partial \mathbf{W}_{yh}}$$


来不断迭代模型参数的值。因此我们需要模型参数梯度 $\partial L/\partial \mathbf{W}_{hx}$、$\partial L/\partial \mathbf{W}_{hh}$ 和 $\partial L/\partial \mathbf{W}_{yh}$。为此，我们可以按照反向传播的次序依次计算并存储梯度。

为了表述方便，对输入输出 $\mathsf{X}, \mathsf{Y}, \mathsf{Z}$ 为任意形状张量的函数 $\mathsf{Y}=f(\mathsf{X})$ 和 $\mathsf{Z}=g(\mathsf{Y})$，我们使用

$$\frac{\partial \mathsf{Z}}{\partial \mathsf{X}} = \text{prod}(\frac{\partial \mathsf{Z}}{\partial \mathsf{Y}}, \frac{\partial \mathsf{Y}}{\partial \mathsf{X}})$$

来表达链式法则, 以下依次计算得到的梯度将依次被存储。

首先，目标函数有关各时刻输出层变量的梯度 $\partial L/\partial \mathbf{o}_t \in \mathbb{R}^y$ 可以很容易地计算

$$\frac{\partial L}{\partial \mathbf{o}_t} =  \frac{\partial \ell (\mathbf{o}_t, y_t)}{T \cdot \partial \mathbf{o}_t} $$

事实上，这时我们已经可以计算目标函数有关模型参数 $\mathbf{W}_{yh}$ 的梯度 $\partial L/\partial \mathbf{W}_{yh} \in \mathbb{R}^{y \times h}$。需要注意的是，在计算图中，
$\mathbf{W}_{yh}$ 可以经过 $\mathbf{o}_1, \ldots, \mathbf{o}_T$ 通向 $L$，依据链式法则，

$$\frac{\partial L}{\partial \mathbf{W}_{yh}}
= \sum_{t=1}^T \text{prod}(\frac{\partial L}{\partial \mathbf{o}_t}, \frac{\partial \mathbf{o}_t}{\partial \mathbf{W}_{yh}})
= \sum_{t=1}^T \frac{\partial L}{\partial \mathbf{o}_t} \mathbf{h}_t^\top
$$


其次，我们注意到隐含层变量之间也有依赖关系, 对于最终时刻 $T$，在计算图中，隐含层变量 $\mathbf{h}_T$ 只经过 $\mathbf{o}_T$ 通向 $L$。因此我们先计算目标函数有关最终时刻隐含层变量的梯度 $\partial L/\partial \mathbf{h}_T \in \mathbb{R}^h$。依据链式法则，我们得到

$$\frac{\partial L}{\partial \mathbf{h}_T} = \text{prod}(\frac{\partial L}{\partial \mathbf{o}_T}, \frac{\partial \mathbf{o}_T}{\partial \mathbf{h}_T} ) = \mathbf{W}_{yh}^\top \frac{\partial L}{\partial \mathbf{o}_T}
$$


为了简化计算，我们假设激活函数 $\phi(x) = x$ 。接下来，对于时刻 $t < T$，在计算图中，
由于 $\mathbf{h}_t$ 可以经过 $\mathbf{h}_{t+1}$ 和 $\mathbf{o}_t$ 通向 $L$ ，依据链式法则，目标函数有关隐含层变量的梯度 $\partial L/\partial \mathbf{h}_t \in \mathbb{R}^h$ 需要按照时刻从晚到早依次计算：


$$\frac{\partial L}{\partial \mathbf{h}_t}
= \text{prod}(\frac{\partial L}{\partial \mathbf{h}_{t+1}}, \frac{\partial \mathbf{h}_{t+1}}{\partial \mathbf{h}_t} )
+ \text{prod}(\frac{\partial L}{\partial \mathbf{o}_t}, \frac{\partial \mathbf{o}_t}{\partial \mathbf{h}_t} )
= \mathbf{W}_{hh}^\top \frac{\partial L}{\partial \mathbf{h}_{t+1}} + \mathbf{W}_{yh}^\top \frac{\partial L}{\partial \mathbf{o}_t}
$$

将递归公式展开，对任意 $1 \leq t \leq T$，我们可以得到目标函数有关隐含层变量梯度的通项公式

$$\frac{\partial L}{\partial \mathbf{h}_t}
= \sum_{i=t}^T {(\mathbf{W}_{hh}^\top)}^{T-i} \mathbf{W}_{yh}^\top \frac{\partial L}{\partial \mathbf{o}_{T+t-i}}
$$

由此可见，当每个时序训练数据样本的时序长度$T$较大或者时刻 $t$ 较小，目标函数有关隐含层变量梯度较容易出现**衰减**（valishing）和**爆炸**（explosion）。想象一下 $2^{30}$ 和 $0.5^{30}$ 会有多大。


有了各时刻隐含层变量的梯度之后，我们可以计算隐含层中模型参数的梯度 $\partial L/\partial \mathbf{W}_{hx} \in \mathbb{R}^{h \times x}$ 和 $\partial L/\partial \mathbf{W}_{hh} \in \mathbb{R}^{h \times h}$ 。在计算图中，它们都可以经过 $\mathbf{h}_1, \ldots, \mathbf{h}_T$ 通向 $L$ 。依据链式法则，我们有

$$\frac{\partial L}{\partial \mathbf{W}_{hx}}
= \sum_{t=1}^T \text{prod}(\frac{\partial L}{\partial \mathbf{h}_t}, \frac{\partial \mathbf{h}_t}{\partial \mathbf{W}_{hx}})
= \sum_{t=1}^T \frac{\partial L}{\partial \mathbf{h}_t} \mathbf{x}_t^\top
$$

$$\frac{\partial L}{\partial \mathbf{W}_{hh}}
= \sum_{t=1}^T \text{prod}(\frac{\partial L}{\partial \mathbf{h}_t}, \frac{\partial \mathbf{h}_t}{\partial \mathbf{W}_{hh}})
= \sum_{t=1}^T \frac{\partial L}{\partial \mathbf{h}_t} \mathbf{h}_{t-1}^\top
$$


在每次迭代中，上述各个依次计算出的梯度会被依次存储或更新, 这是为了避免重复计算。例如，由于输出层变量梯度 $\partial L/\partial \mathbf{h}_t$ 被计算存储，反向传播稍后的参数梯度 $\partial L/\partial  \mathbf{W}_{hx}$ 和隐含层变量梯度 $\partial L/\partial \mathbf{W}_{hh}$ 的计算可以直接读取输出层变量梯度的值，而无需重复计算。

还有需要注意的是，反向传播对于各层中变量和参数的梯度计算可能会依赖通过正向传播计算出的各层变量和参数的当前值。举例来说，参数梯度 $\partial L/\partial \mathbf{W}_{hh}$ 的计算需要依赖隐含层变量在时刻 $t = 1, \ldots, T-1$ 的当前值 $\mathbf{h}_t$（$\mathbf{h}_0$ 是初始化得到的）。这个当前值是通过从输入层到输出层的正向传播计算并存储得到的。

>* 所谓通过时间反向传播只是反向传播在循环神经网络的具体应用。
>* 当每个时序训练数据样本的时序长度$T$较大或者时刻$t$较小，目标函数有关隐含层变量梯度较容易出现衰减和爆炸。


## 长短期记忆 (LSTM)
长短期记忆（Long Short-Term Memory, LSTM）是一种时间递归神经网络，适合于处理和预测时间序列中间隔和延迟相对较长的事件，LSTM 区别于传统前馈神经网络的地方在于网络内部有机制可以实现保留对输入的记忆，并且还通过加入遗忘机制和保留机制等以非常精确的方式改变记忆，应用专门的学习机制来记住、更新、聚焦于信息。这有助于实现更长时间内的信息跟踪，并且也可以有效处理循环神经网络中梯度消失的问题。

LSTM 网络结构图如下图所示:

![RNN-unrolled][3]

其大体结构是基于循环神经网络，加入了一个判断信息是否有用的“处理器”，这个处理器作用的结构被称为元胞（cell）, 即图中的 A，A 的详细结构如下图所示:

![LSTM3-chain][4]

一个 cell 中被放置了三扇门，分别叫做输入门、遗忘门和输出门。一个信息进入 LSTM 的网络当中，可以根据规则来判断是否有用。只有符合算法认证的信息才会留下，不符的信息则通过遗忘门被遗忘。

LSTM 第一步是决定我们将要从元胞状态中扔掉哪些信息。该决定由“遗忘门（Forget  Gate）”的 Sigmoid 层控制，遗忘门观察上一个隐藏状态 $h_{t-1}$ 和当前输入 $x_t$, 对于上一个元胞状态 $C_{t-1}$ 中的每一个元素，输出 $f_t$ 中相应的元素为 0~1 之间的数，表示对信息的保留程度。

![LSTM3-focus-f][5]

下一步决定我们会把哪些新信息存储到元胞状态中，这分为两部分。首先，”输入门（Input Gate）” 的 Sigmoid 层决定我们要更新哪些信息; 接下来，一个 $tanh$ 层创造了一个元胞状态新的候选值 $\tilde{C}_t$, 该值可能被加到元胞状态中。

![LSTM3-focus-i][6]

下一步就是更新旧元胞状态 $C_{t-1}$ 到新状态 $C_t$ 了。我们把 $C_{t-1}$ 乘以 $f_t$，忘掉没用的信息，然后再加上 $i_t \times \tilde{C}_t$，这个值是侯选值 $\tilde{C}_t$ 乘以侯选值的每一个状态的更新权重 $i_t$ 构成，决定元胞中当前输入的影响。

![LSTM3-focus-C][7]

最后，最终的输出状态将基于目前的元胞状态，并且会加入一些过滤。首先建立一个 Sigmoid 层的输出门（Output Gate）来决定我们将输出元胞的哪些部分，然后我们将元胞状态通过 $tanh$ 函数后使输出值在 -1~1 之间，之后与输出门相乘，得到最终的输出。

![LSTM3-focus-o][8]

LSTM 模型代码实现如下:
```python
def lstm_rnn(inputs, state_h, state_c, *params):
    # inputs: num_steps 个尺寸为 batch_size * vocab_size 矩阵
    # H: 尺寸为 batch_size * hidden_dim 矩阵
    # outputs: num_steps 个尺寸为 batch_size * vocab_size 矩阵
    [W_xi, W_hi, b_i, W_xf, W_hf, b_f, W_xo, W_ho, b_o, W_xc, W_hc, b_c,
     W_hy, b_y] = params

    H = state_h
    C = state_c
    outputs = []
    for X in inputs:
        I = nd.sigmoid(nd.dot(X, W_xi) + nd.dot(H, W_hi) + b_i)
        F = nd.sigmoid(nd.dot(X, W_xf) + nd.dot(H, W_hf) + b_f)
        O = nd.sigmoid(nd.dot(X, W_xo) + nd.dot(H, W_ho) + b_o)
        C_tilda = nd.tanh(nd.dot(X, W_xc) + nd.dot(H, W_hc) + b_c)
        C = F * C + I * C_tilda
        H = O * nd.tanh(C)
        Y = nd.dot(H, W_hy) + b_y
        outputs.append(Y)
    return (outputs, H, C)
```
### LSTM 的变体
以上介绍的是正常的 LSTM, 当并不是所有的 LSTM 都长这样. 事实上, 几乎所有包含 LSTM 的论文都采用了微小的变体。差异非常小，但是也值得拿出来讲一下。
其中一个流形的 LSTM 变体，就是由 [Gers & Schmidhuber (2000)](ftp://ftp.idsia.ch/pub/juergen/TimeCount-IJCNN2000.pdf) 提出的，增加了 “peephole connection”, 让门层也会接受细胞状态的输入。

![LSTM3-var-peepholes][9]

上面的图例增加了 peephole 到每个门上，但是许多论文会加入部分的 peephole 而非所有都加。

另一个变体是通过使用 coupled 忘记和输入门。不同于之前的分开确定忘记什么和添加什么新的信息，这里是一同做出决定。变体仅仅在将要输入时选择遗忘, 仅仅当忘记旧值后才会输入新值.

![LSTM3-var-tied][10]

另一个改动较大的变体是 Gated Recurrent Unit (GRU)，这是由 [Cho, et al. (2014)](https://arxiv.org/pdf/1406.1078v3.pdf) 提出。它将忘记门和输入门合成了一个单一的 更新门。同时还混合了细胞状态和隐藏状态，和其他一些改动。最终的模型比标准的 LSTM 模型要简单，也是非常流行的变体。

![LSTM3-var-GRU][11]

这里只是部分流行的 LSTM 变体。当然还有很多其他的，如 [Yao, et al. (2015)](https://arxiv.org/pdf/1503.04069.pdf) 提出的 Depth Gated RNN。还有用一些完全不同的观点来解决长期依赖的问题，如 [Koutnik, et al. (2014)](https://arxiv.org/pdf/1402.3511v1.pdf) 提出的 Clockwork RNN。

[1]: rnn-1.png
[2]: rnn-bptt.svg
[3]: RNN-unrolled.png
[4]: LSTM3-chain.png
[5]: LSTM3-focus-f.png
[6]: LSTM3-focus-i.png
[7]: LSTM3-focus-C.png
[8]: LSTM3-focus-o.png
[9]: LSTM3-var-peepholes.png
[10]: LSTM3-var-tied.png
[11]: LSTM3-var-GRU.png
