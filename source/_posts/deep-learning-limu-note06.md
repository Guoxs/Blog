---
title: MXNet/Gluon 深度学习笔记 (六) —— 物体检测总结
date: 2018-02-03 15:02:11
tags: DeepLearning
mathjax: true
---

目前深度学习中关于物体识别的问题总共有四大类，从最简单的 **图像分类** (Image classification) 到 **物体定位** (Object Localization)，再到 **语义分割** (Semantic Segmentation)，最后到难度最高的 **实例分割** (Instance Segmentation)。在这四大类问题中，Object Detection 一般指第二类，也即是物体定位问题。物体定位问题在整个物体识别技术线中处于承前启后的地位，它的难度要比单纯的图片分类问题复杂许多，而其运用却是最广泛的。目前领域内的研究者在 Object Detection 问题上进行了很多探索，也取得了许多阶段性的成果。 (Object Detection 相关研究的整理详见这篇 [**博文**][1]，作者整理的很详细)。本篇博文按时间顺序，介绍了 Object Detection 的几篇有代表性的论文，重点介绍论文的思路与方法。

![Object Detection][2]

上图清楚说明了image classification, object detection, semantic segmentation, instance segmentation之间的关系. 摘自　[COCO dataset][3]
<!--more-->
Object Detection 需要在图片中精确找出物体所在的位置 (一般以矩形框出)，并标注物体的类别。由于物体的尺寸变化范围很大，摆放物体的角度、姿势等也不确定，并且物体间也会有重叠，这等等问题使得物体定位问题不是那么容易解决。Object Detection 的技术演进大致如下：

> R-CNN --> SPP-Net --> Fast-RCNN --> Faster-RCNN

而后的研究也都是基于 Faster-RCNN 进行改进。

## R-CNN
论文链接：[Rich feature hierarchies for accurate object detection and semantic segmentation][4]
GitHub：https://github.com/rbgirshick/rcnn

RCNN (Region-CNN) 可以说是 **利用深度学习进行目标检测的开山之作**。该论文解决了目标检测中的两个关键问题，一个是 **速度**（用 region proposals 代替 滑动窗口），一个是 **训练集**。 论文利用训练的神经网络进行图片的特征提取 (传统方法需要人工设计特征)，使用两个数据库：

- 一个较大的 **识别库** (ImageNet ILSVC 2012，一千万图像，1000类) 标定每张图片中物体的类别；
- 一个较小的 **检测库**（PASCAL VOC 2007，一万图像，20类）来标定物体的类别和位置。

论文使用识别库进行预训练，而后用检测库调优参数，最后在检测库上评测。

### 算法
RCNN 的算法流程如下图所示：

![RCNN][5]

算法主要分为四个步骤：

- 对每张输入图片使用一个基于规则的 “选择性搜索” 算法，生成 1K~2K 个**候选区域**（Region proposals）；
- 对每个候选区域，使用训练的神经网络进行**特征提取**；
- 将网络提取的特征送入每一类的 SVM 分类器进行判别
- 使用回归器**精细修正**候选框位置

每一步骤的细节如下：

#### **Step 1:** 候选框提取（selective search）

给定一张图片，使用 [Selective Search][6] 方法从图片中生成约 2000~3000 个候选区域。

生成候选区域过程包含以下两个基本步骤：

- 使用过分割手段，将图像分割成许多小区域
- 查看现有的小区域，合并 **可能性最高** 的两个区域，重复直到整张图像合并成一个区域位置
- 输出所有曾经存在过的区域，即为候选区域

**合并规则：**

- 颜色（颜色直方图）相近的
- 纹理（梯度直方图）相近的
- 合并后总面积小
- 合并后总面积在 $B_{box}$ 中占的比例大

其中第三条保证合并操作的尺度较为均匀，避免一个大区域陆续“吃掉”其他小区域；

>例：设有区域a-b-c-d-e-f-g-h。较好的合并方式是：ab-cd-ef-gh -> abcd-efgh -> abcdefgh。
不好的合并方法是：ab-c-d-e-f-g-h ->abcd-e-f-g-h ->abcdef-gh -> abcdefgh。

而第四条保证了合并后形状规则。

>例：左图适于合并，右图不适于合并。
![example][7]

以上四条规则只涉及到区域的颜色直方图、纹理直方图、面积和位置，合并后的区域特征可以直接由子区域特征计算而来，故速度较快。

>为了尽量不遗漏候选区域，以上操作还可以多个颜色空间同时进行（RGB、HSV、Lab等），在一个颜色空间中，使用上述四条规则的不同组合进行合并。所有颜色空间与所有规则的全部结果，再去除重复后，都作为候选区域输出。

[Selective Search 算法源码链接][8]

#### **Step 2 :** 特征提取
由于候选框大小不一，而后续的 CNN 要求输入大小统一，故需要将 2000 个候选框全部 resize 到 227x227 分辨率，同时为了避免图像扭曲严重，中间还可采取一些技巧减少图片扭曲，例如直接外扩成 227x227，外扩对框外区域可以直接截取或者补灰。论文采用的网络基本借鉴Hinton 2012年在Image Net上的分类网络，如下图所示：

![network][9]

使用 ILVCR 2012 的全部数据进行训练，输入一张图片，输出 1000 维的类别标号。训练CNN模型时，对训练数据标定要求比较宽松，即 Selective Search 方法提取的 proposal 只包含部分目标区域时，也将该 proposal 标定为特定物体类别。

>当且仅当一个候选框完全包含 ground truth 区域且不属于 ground truth 部分不超过候选框区域的 5% 时认为该候选框标定结果为目标，否则位背景。

在调优阶段，同样使用上述网络，只是最后一层换成了 4096 --> 21 的全连接网络。使用 PASCAL VOC 2007 的训练集，输出 21 维的类别标号，表示 20类 + 背景。考察一个候选框和当前图像上所有标定框重叠面积最大的一个。如果重叠比例大于0.5，则认为此候选框为此标定的类别；否则认为此候选框为背景。

#### **Step 3 :** 类别判断
对每一类目标，使用一个线性 SVM 二分类器进行判别，输入为深度网络输出的4096维特征，输出是否属于此类。 由于负样本很多，训练过程中需要使用 [Hard negative mining][10] 方法。

**正样本：** 本类的真值标定框；
**负样本：** 考察每一个候选框，如果和本类所有标定框的重叠（IoU）都小于 0.3，则认定为负样本。

#### **Step 4：** 位置精修
**目标检测问题的衡量标准是重叠面积**：许多看似准确的检测结果，往往因为候选框不够准确，重叠面积很小。故需要一个位置精修步骤。

**回归器：** 对每一类目标，使用一个线性脊回归器进行精修。正则项 λ=10000。 输入为深度网络 pool5 层的 4096 维特征，输出为 xy 方向的缩放和平移。

**训练样本：** 判定为本类的候选框中，和真值重叠面积大于0.6的候选框。

### 结果
论文深度学习引入检测领域，一举将 PASCAL VOC 上的检测率从 **35.1%** 提升到 **53.7%**。

算法前两个步骤（候选区域提取+特征提取）与待检测类别无关，可以在不同类之间共用。这两步在 GPU 上约需 13 秒。

同时检测多类时，需要倍增的只有后两步骤（判别+精修），都是简单的线性运算，速度很快。这两步对于 1000 类别只需 10 秒。

**不足：** RCNN 需要对 SS 算法提取的每个 proposal 进行一次前向 CNN 实现特征提取，因此计算量很大，无法实时。此外，由于全连接层的存在，需要严格保证输入的 proposal 最终 resize 到相同的尺度大小，这在一定层度上造成了图像畸变，影响最终结果。另外，特征提取 CNN 的训练和 SVMs 分类器的训练在时间上是先后顺序，两者的训练方式独立，因此 SVMs 的训练 Loss 无法更新 SPP-Layer 之前的卷积层参数，因此即使采用更深的 CNN 网络进行特征提取，也无法保证 SVMs 分类器的准确率一定能够提升。
## SPP-Net
　　论文链接：[Spatial Pyramid Pooling in Deep Convolutional Networks for Visual Recognition][11]

　　RCNN 后面的进化中借鉴了许多 SPP Net 的思想，因此在介绍 Fast RCNN 与 Faster RCNN 之前，有必要先了解一下 SPP Net。

![SPP-net VS traditional CNN][12]

### SPP Layer
SPP 即**图像金字塔池化**，SPP-Net 区别于传统 CNN 网络的特点之一就是将金字塔思想加入到 CNN 中，**实现了数据的多尺度输入，网络对输入图像大小不做特别要求**。

传统的 CNN 中，卷积层对输入图像的大小并不做特别要求，对图像尺寸有要求的是全连接层。因此，在 RCNN 中，对于用 SS 方法提取的不同大小的 proposal 需要先通过 **裁剪** 或者 **扩充** 操作来讲 proposal 区域裁剪为统一大小，然后再利用 CNN 提取特征。SPP-Net 中，为了实现多尺度输入，作者在卷积层与全连接层之间加入了 SPP Layer，如下图所示，从而不必让每个 proposal 大小统一。

![SPP Layer][13]

SPP Layer 是怎么实现多尺度输入呢？其实想法很简单，把最后一层的 Pooling 层 改为 SPP-Pooling 层，也就是说，使用不同的池化步长对特征图进行多次池化，使得池化层输出的尺寸固定。

>假设原图输入为 224x224，对于 conv5 出来的输出是 13x13x256，可以理解为 256 个这样的 filter，每个 filter 对应于一张 13x13 的 response map。如果想上图那样将 response map 分为 1x1（金字塔底座），2x2（金字塔中间），4x4（金字塔顶座）三张子图，分别做 max pooling 后，出来的特征就是 （16+4+1）x 256 维度，这样不管输入图像尺寸如何，该池化层的输出永远是 （16+4+1）x 256 维度。

### 一次卷积

![一次卷积][14]

在 RCNN 中，每一个候选框需要先 resize 到统一大小，然后分别作为 CNN 的输入，这样是十分低效的。SPP-Net 就是根据这个缺点做了优化，**只对原图做一次卷积操作** 得到整张图的 feature map，然后找到每个候选框在特征图上的映射区域，将此区域作为每个候选框的卷积特征输入到 SPP Layer 和之后的层（[将原图 ROI 映射到 Feature map 上机制][15]）。这一过程相对于 RCNN 节省了大量的计算时间，从而使得 SPP-Net 比 RCNN 有一百多倍的提速。

SPP-Net 的整体流程还是 `Selective Search得到候选区域 --> CNN提取ROI特征 --> 类别判断 --> 位置精修`，但是由于所有 ROI 的特征直接在 feature map 上提取，大大减少了卷积操作，提高了效率。
**不足：** SPP-Net 的不足之处主要是训练是一个多阶段（multi-stage）的过程，实现多次较复杂。和 RCNN 一样，SPP-Net 首先选用 Selective Search 方法提取 proposals，然后用 CNN 实现特征提取，最后基于 SVMs 算法训练分类器，在此基础上还可以进一步学习检测目标的 boulding box。SPP-Net 的时间成本和空间代价较高，用于训练 SVMs 分类器的特征需要提前保存在磁盘，造成空间代价较高。SPP-Net 训练和测试还是较慢，无法达到实时性要求。

## Fast RCNN
论文链接：[Fast R-CNN][16]
Github：https://github.com/rbgirshick/fast-rcnn

RCNN 使用 SS 算法提取潜在的 bounding box 作为输入，有严重的速度瓶颈，这是因为计算机对所有 region 进行特征提取是会有重复计算。基于 R-CNN 和 SPP-Net 思想，RBG 提出了 Fast-RCNN 算法。 Fast RCNN 就是在 RCNN 的基础上采纳了 SPP Net 方法，对 RCNN 做了改进，使得性能进一步提高。同样使用最大规模的网络，Fast RCNN和RCNN相比，训练时间从84小时减少为9.5小时，**测试时间从47秒减少为0.32秒**。在PASCAL VOC 2007上的准确率相差无几，约在66%-67%之间.

### 思想
具体而言，Fast RCNN 方法解决了 RCNN 方法的三个问题：

**1. 测试时速度慢**
RCNN 测试一张图片时需要提取很多候选框，而候选框之间有大量重叠，提取特征操作冗余。而 Fast RCNN 中将整张图片归一化后直接送入神经网络，在邻接时，才加入候选框信息，在末尾的少数几层处理每个候选框。

**2. 训练时速度慢**
原因同上。 而 Fast RCNN 在训练时，先将一张图像送入网络，紧接着送入从这幅图像上提取出的候选区域。这些候选区域的前几层特征不需要再重复计算。

**3. 训练时所需空间大**
CNN中独立的分类器和回归器需要大量特征作为训练样本。 Fast RCNN 把类别判断和位置精调统一用深度网络实现，不再需要额外存储。

### 实现
![Fast-RCNN][17]

Fast-RCNN 的架构如上图所示。网络的初始输入是一张图像，通过一系列卷积层和Pooling层生成 feature map；在第五阶段结尾，输入由 Selective Search 方法生成的 P 个候选区域（图像序号×1+几何位置×4，序号用于训练），然后用 **ROI**（region of ineterst）层处理 **最后一个卷积层** 得到的 feature map，为每一个 proposal 生成一个 **定长的特征向量 roi_pool5**。ROI 层的输出 roi_pool5 接着输入到全连接层产生最终用于多任务学习的特征并用于计算多任务 Loss。

![网络架构图][18]
![网络架构图][19]

全连接输出包括两个分支：
**1.SoftMax Loss:** 计算 K+1 类的分类 Loss 函数（其中K表示K个目标类别，1表示背景）；
**2.Regression Loss:** 即 K+1 的分类结果相应的 Proposal 的 Bounding Box 四个角点坐标值。最终将所有结果通过**非极大抑制**处理产生最终的目标检测和识别结果。

**ROI Pooling Layer**
事实上，ROI Pooling Layer 是 SPP-Layer 的简化形式。SPP-Layer 是空间金字塔 Pooling 层，包括不同的尺度；ROI Layer 只包含一种尺度，如论文中所述 $7 \times 7$。这样对于 ROI Layer 的输入（r,c,h,w），ROI Layer 首先产生 $7 \times 7$ 个 $r \times c \times (h/7)\times(w/7)$ 的 Block(块)，然后用 Max-Pool 方式求出每一个 Block 的最大值，这样 ROI Layer 的输出是 $r\times c \times 7 \times 7$。

>roi_pool 层将每个候选区域均匀分成 M×N 块，对每块进行 max pooling。将特征图上大小不一的候选区域转变为大小统一的数据，送入下一层。

**将所有模型整合到一个网络**
Fast R-CNN 的另一个创新点是在一个模型中联合训练卷积神经网络、分类器和边界框回归模型。在 R-CNN 中，我们使用了卷积神经网络来提取图像特征，用支持向量机来分类对象和用了回归模型来缩小边界框，但是Fast R-CNN使用单个网络模型来实现以上三个功能。

Fast-RCNN 在网络训练阶段采用了一些 trick，每个 mini-batch 由 N 个图片（N=2）中的 R 个 Proposal（R=128）组成。这种方式比从 128 张不同图片中提取 1 个 Proposal 的方式块 64 倍。当然，这种方式在一定程度会造成收敛速度变慢。

>注意：从 2 张图中选取 128 个 proposals 时，需要保证至少 25% 的 proposals 与 ground truth 的 IoU 超过 0.5，剩下的全部作为背景类。不需要其它任何数据扩增操作。

另外，Fast-RCNN 无需 SVM 分类器，而是通过 Softmax Classifer 和 Bounding-Box Regressors 联合训练的方式更新所有参数。

>在检测阶段，作者使用 truncated SVD 优化较大的 FC 层，这样 ROI 数目较大时检测端速度会得到的加速。

这样，整个模型的输入和输出分别为：

- 输入：带多个区域建议的图像
- 输出：具有更紧密边界框的每个区域的对象类别

### 实验结论
1. 多任务 Loss 学习方式可以提高算法准确率

2. 多尺度图像训练 Fast-RCNN 与单尺度图像训练相比只能提升微小的 mAP，但是时间成本却增加了很多。因此，综合考虑训练时间和 mAP，作者建议**直接用一种尺度的图像训练 Fast-R-CNN**

3. 论文的结果表明 SoftmaxLoss 的方式比 SVMs 分类器的结果略好一点点，虽然这不能绝对性说明 SoftmaxLoss 好到哪儿去，但是至少不用再那么麻烦的去分步训练一个检测和识别网络了。

4. 不是说 Proposal 提取的越多效果会越好，提的太多反而会导致 mAP 下降

>Fast-RCNN 很重要的一个贡献是成功的让人们看到了 **Region Proposal+CNN** 这一框架实时检测的希望，原来多类检测真的可以在保证准确率的同时提升处理速度，也为后来的 Faster-RCNN 做下了铺垫。


**不足**：Fast RCNN 的不足之处主要在于 SS 方法是极耗时的，是测试时的计算瓶颈。

## Faster RCNN
论文链接：[Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks][20]
Github：https://github.com/rbgirshick/py-faster-rcnn

在 Fast RCNN 中，第一步需要先使用 Selective Search 方法提取图像中的 proposals。基于 CPU 实现的 Selective Search 提取一幅图像的所有 Proposals 需要约 **2s** 的时间。在不计入 proposal 提取情况下，Fast RCNN 基本可以实时进行目标检测。但是，如果从端到端的角度考虑，显然 proposal 提取成为影响端到端算法性能的瓶颈。目前最新的 EdgeBoxes 算法虽然在一定程度提高了候选框提取的准确率和效率，但是处理一幅图像仍然需要 **0.2s**。

因此，在 2015 年中期，由任少卿，何恺明，Ross Girshick 和孙剑组成的一个微软研究院团队提出新的 Faster RCNN 算法，该算法引入了 **RPN 网络**（Region Proposal Network）提取 proposals。RPN 网络是一个全卷积神经网络，通过共享卷积层特征可以实现 proposal 的提取，RPN 提取一幅像的 proposal 只需要 **10ms**。

Faster RCNN 算法由两大模块组成：

- PRN 候选框提取模块
- Fast R-CNN 检测模块

其中，RPN 是全卷积神经网络，用于提取候选框；Fast R-CNN 基于 RPN 提取的 proposal 检测并识别 proposal 中的目标。

faster-rcnn网络结构图：
![faster-rcnn网络结构][21]

### RPN 结构
![RPN][22]

RPN 是作者重点介绍的一种网络，如上图所示。在最后一层卷积层生成的 feature map 上用一个 $n \times n$ 的滑窗（论文中 n = 3）生成一个长度为 256（ZF 网络）维长度的全连接特征，提取 256 维的特征后产生两个分支：
1、**reg layer**，用来预测 proposal 的中心锚点对应的 proposal 的坐标 $x,y$ 和宽高 $w,h$；
2、**cls layer**，用来判定该 proposal 是前景还是背景。

滑动窗口的处理方式保证 reg layer 和 cls layer 关联了 feature map 的全部特征空间。

在RPN网络中，需要重点理解其中的 **anchors 概念**，Loss fucntions 计算方式和 **RPN 层训练数据生成的具体细节** 。

### Anchors
Anchors 字面上可以理解为锚点，位于之前提到的 $n \times n$ 的 sliding window 的中心处。在 feature map 上的每个特征点预测多个 region proposals。具体作法是：**把每个特征点映射回原图的感受野的中心点当成一个基准点**，然后围绕这个基准点选取 k 个不同 scale、aspect ratio（纵横比） 的 anchor。论文中 3 个 scale（三种面积 $\{ 128^2, 256^2, 521^2 \}$），3 个 aspect ratio ( $\{1:1,1:2,2:1\}$ )

![Anchors][23]

因此，一个 anchor 可以理解为一个 anchor box 或一个 reference box，论文中定义 k = 9，即 3 种 scales 和 3 种 aspect_ratio 确定出当前 sliding window 位置处对应的 9 个 reference boxes， $4 \times k$ 个 reg layer 的输出和 $2 \times k$ 个 cls layer 的 score 输出。对于一幅 $W \times H$ 的 feature map，对应 $W \times H \times k$ 个锚点。所有的锚点都具有**尺度不变性**。

### Loss Function
在计算 Loss 值之前，作者设置了 anchors 的标定方法。
**正样本标定：**
1. 对每个标定的 ground true box 区域，与其重叠比例最大的 anchor 记为正样本 (保证每个 ground true 至少对应一个正样本 anchor )
2. 对 1 中剩余的 anchor，如果其与某个标定区域重叠比例 (IoU) 大于 **0.7**，记为正样本（每个 ground true box 可能会对应多个正样本 anchor。但每个正样本 anchor 只可能对应一个grand true box）；如果其与任意一个标定的重叠比例都小于 **0.3**，记为负样本。
3. 对 1、2 剩余的 anchor，弃去不用。
4. 跨越图像边界的anchor弃去不用。

**定义损失函数：** 对于每个 anchor，首先后面接着一个二分类 softmax，有 2 个 score 输出用以表示其是一个物体的概率与不是一个物体的概率 $p_i$，然后再接上一个 bounding box 的 regressor 输出代表这个 anchor的 4 个坐标位置 $t_i$，因此 RPN 的总体 Loss 函数可以定义为 ：
$$
L(\{p_i\}\{t_i\}) = \frac{1}{N_{cls}}\sum_iL_{cls}(p_i,p^\*\_i) + \lambda \frac{1}{N_{reg}}\sum_i p_i^\* L_{reg}(t_i,t^\*\_i)
$$

$i$ 表示第 $i$ 个 anchor，当 anchor 是正样本时 $p_i^\* =1$ ，是负样本则为 0 。  $t_i^\*$ 表示 一个与正样本 anchor 相关的 ground true box 坐标 。

$x,y,w,h$ 分别表示 box 的中心坐标和宽高，$x,x_a,x^\*$ 分别表示 predicted box, anchor box, and ground truth box （$y,w,h$ 同理）$t_i$ 表示 predict box 相对于 anchor box 的偏移，$t_i^\*$ 表示 ground true box相对于anchor box 的偏移，学习目标自然就是让前者接近后者的值。

![t_i][24]

$$
t_x = (x - x_a)/\omega_a, t_y = (y-y_a)/h_a,\\
t_w = log(w/w_a),t_h = log(h/h_a),\\
t_x^\* = (x^\* - x_a)/\omega_a, t_y^\* = (y^\*-y_a)/h_a,\\
t_w^\* = log(w^\*/w_a),t_h = log(h^\*/h_a)
$$

其中 $L_{reg}$ 是：
$$
 smooth_{L_1}(x) =
\begin{cases}
 0.5x^2 & |x| \leq 1 \\
|x|-0.5  & \text{otherwise}
\end{cases}
$$

而总的网络有四个损失函数：

- RPN classification(anchor good/bad)
- RPN regression(anchor --> proposal)
- Fast R-CNN classification(over classes)
- Fast R-CNN regression(proposal --> box)

### 训练
在训练 RPN 时，一个 Mini-batch 是由一幅图像中任意选取的 256 个 proposal 组成的，其中正负样本的比例为 1：1。如果正样本不足 128，则多用一些负样本以满足有 256 个 proposal 可以用于训练，反之亦然。训练 RPN 时，与 VGG 共有的层参数可以直接拷贝经 ImageNet 训练得到的模型中的参数；剩下没有的层参数用标准差为 0.01 的高斯分布初始化。

**RPN 与 Fast R-CNN 特征共享**
RPN　在提取得到　proposals　后，作者选择使用　Fast　R-CNN　实现最终目标的检测和识别。RPN　和　Fast　R-CNN　共用了　13　个　VGG　的卷积层，显然将这两个网络完全孤立训练不是明智的选择，论文作者作者采用交替训练阶段卷积层特征共享：

**第一步：** 用 ImageNet 模型初始化，独立训练一个 RPN 网络；
**第二步：** 仍然用 ImageNet 模型初始化，但是使用上一步 RPN 网络产生的 proposal 作为输入，训练一个 Fast-RCNN 网络，至此，两个网络每一层的参数完全不共享；
**第三步：** 使用第二步的 Fast-RCNN 网络参数初始化一个新的 RPN 网络，**但是把 RPN、Fast-RCNN 共享的那些卷积层的learning rate设置为0**，也就是不更新，仅仅更新 RPN 特有的那些网络层，重新训练，此时，两个网络已经共享了所有公共的卷积层；
**第四步：** 仍然固定共享的那些网络层，把 Fast-RCNN 特有的网络层也加入进来，形成一个 **unified network**，继续训练，fine tune Fast-RCNN 特有的网络层，此时，该网络已经实现我们设想的目标，即网络内部预测 proposal 并实现检测的功能。

整个过程图示如下：
![-Step Alternating Training][25]

### 结果
以下列出 R-CNN、Fast R-CNN 与 Faster R-CNN 的速度对比：

|Time & mAP|R-CNN|Fast R-CNN|Faster R-CNN|
|:---:|:---:|:---:|:---:|
|Test time per image (with proposals)|50s|2s|0.2s|
|Speedup|1x|25x|250x|
|mAP(VOC 2007)|66.0|66.9|66.9|

Faster R-CNN的主要贡献是设计了提取候选区域的网络RPN，代替了费时的选择性搜索，使得检测速度大幅提高。

最后，总结以下这三大算法的步骤：
>**RCNN**
　　1.	在图像中确定约 1000-2000 个候选框 (使用选择性搜索)
　　2. 每个候选框内图像块缩放至相同大小，并输入到 CNN 内进行特征提取
　　3.	对候选框中提取出的特征，使用分类器判别是否属于一个特定类
　　4.	对于属于某一特征的候选框，用回归器进一步调整其位置

>**Fast RCNN**
　　1.	在图像中确定约 1000-2000 个候选框 (使用选择性搜索)
　　2.	对整张图片输进 CNN ，得到 feature map
　　3.	将每个候选框在 feature map 上的映射 patch 作为每个候选框的卷积特征输入到 SPP layer 和之后的层
　　4.	对候选框中提取出的特征，使用分类器判别是否属于一个特定类
　　5.	对于属于某一特征的候选框，用回归器进一步调整其位置

>**Faster RCNN**
　　1.	对整张图片输进 CNN，得到 feature map
　　2.	卷积特征输入到 RPN，得到候选框的特征信息
　　3.	对候选框中提取出的特征，使用分类器判别是否属于一个特定类
　　4.	对于属于某一特征的候选框，用回归器进一步调整其位置

## YOLO
论文链接：[You Only Look Once: Unified, Real-Time Object Detection][26]
GitHub：https://github.com/pjreddie/darknet

### 思想
从 R-CNN 到 Fast R-CNN 再到 Faster RCNN 一直采用的思路是 proposal + 分类，精度已经很高，但是速度还不行。 YOLO 提供了另一种更为直接的思路： 直接在输出层回归 bounding box 的位置和 bounding box 所属的类别(整张图作为网络的输入，把 Object Detection 的问题转化成一个 Regression 问题)。

YOLO的主要特点：

- 速度快，能够达到实时的要求。在 Titan X 的 GPU 上 能够达到 45 帧每秒，基本达到实时要求。
- 使用全图作为 Context 信息，背景错误（把背景错认为物体）比较少。
- 泛化能力强。

### YOLO的实现方法
![YOLO 的实现流程][27]

YOLO 的实现流程：

- 将一幅图像分成 SxS 个网格 (grid cell)，**如果某个 object 的中心 落在这个网格中，则这个网格就负责预测这个object**
- 每个网格要预测 B 个 bounding box，每个 bounding box 除了要回归自身的位置之外，还要附带预测一个 confidence 值。
这个 confidence 代表了所预测的 box 中含有 object 的**置信度**和这个 box 预测的**有多准**两重信息，其值是这样计算的：$Pr(Object) \times IoU^{truth}_{pred}$
其中如果有 object 落在一个 grid cell 里，第一项取 1，否则取 0。 第二项是预测的 bounding box 和实际的 groundtruth 之间的 IoU 值。
- 每个 bounding box 要预测 $(x, y, w, h)$ 和 confidence 共 5 个值，每个网格还要预测一个类别信息，记为 C 类。则 $S \times S$ 个网格，每个网格要预测 B 个 bounding box还要预测 C 个 categories。输出就是 $S \times S \times (5 \times B + C)$ 的一个tensor。

>class 信息是针对每个网格的，confidence 信息是针对每个 bounding box 的。

- 举例说明: 在 PASCAL VOC 中，图像输入为 448x448，取 S=7，B=2，一共有 20 个类别(C=20)。则输出就是 7x7x30 的一个tensor。
网络结构借鉴了 GoogLeNet 。24个卷积层，2个全链接层。整个网络结构如下图：

![网络结构图][28]

- 在 test 的时候，每个网格预测的 class 信息和 bounding box 预测的 confidence 信息相乘，就得到每个 bounding box 的 class-specific confidence score:
$Pr(Class_i|Object)\*Pr(Object)\*IOU^{truth}_{pred} = Pr(Class_i)\*IOU^{truth}_{pred}$

 等式左边第一项就是每个网格预测的类别信息，第二三项就是每个 bounding box 预测的 confidence。这个乘积即 encode 了预测的 box 属于某一类的概率，也有该 box 准确度的信息。

- 得到每个 box 的 class-specific confidence score 以后，设置阈值，滤掉得分低的 boxes，对保留的 boxes 进行 NMS（非极大值抑制）处理，就得到最终的检测结果。

### 损失函数设计
![loss][29]

损失函数的设计目标就是让坐标 $(x,y,w,h), confidence，classification$  这个三个方面达到很好的平衡。简单的全部采用了 sum-squared error loss 来做这件事会有以下不足：
>a) 8维的 localization error 和 20 维的 classification error 同等重要显然是不合理的；

>b)如果一个网格中没有object（一幅图中这种网格很多），那么就会将这些网格中的box的confidence push到0，相比于较少的有object的网格，这种做法是overpowering的，这会导致网络不稳定甚至发散。

**解决方案如下：**

1. 更重视 8 维的坐标预测，给这些损失前面赋予更大的 loss weight, 记为 $\lambda_{coord}$ ,在 pascal VOC 训练中取 5。（上图第一部分）
2. 对没有 object 的 bbox 的 confidence loss，赋予小的 loss weight，记为 $\lambda_{noobj}$ ，在 pascal VOC 训练中取 0.5。（上图第三部分）
3. 有 object 的 bbox 的 confidence loss (上图第二部分) 和类别的 loss （上图第四部分）的 loss weight 正常取 1。
4. 对不同大小的 bbox 预测中，相比于大 bbox预测偏一点，小 box 预测偏一点更不能忍受。而 sum-square error loss 中对同样的偏移 loss 是一样。 为了缓和这个问题，作者用了一个比较取巧的办法，就是将 box 的 width 和 height 取平方根代替原本的 height 和 width。
5. 一个网格预测多个 bounding box，在训练时我们希望每个 object（ground true box）只有一个 bounding box 专门负责（一个object 一个bbox）。具体做法是**与ground true box（object）的 IOU 最大的 bounding box 负责该 ground true box(object) 的预测**。这种做法称作 **bounding box predictor 的 specialization (专职化)**。每个预测器会对特定（sizes,aspect ratio or classed of object）的 ground true box 预测的越来越好。


### 不足
- YOLO对相互靠的很近的物体（挨在一起且中点都落在同一个格子上的情况），还有很小的群体 检测效果不好，这是因为一个网格中只预测了两个框，并且只属于一类。

- 测试图像中，当同一类物体出现的不常见的长宽比和其他情况时**泛化能力偏弱**。

- 由于损失函数的问题，定位误差是影响检测效果的主要原因，尤其是大小物体的处理上，还有待加强。

## SSD
论文链接：[SSD: Single Shot MultiBox Detector][30]
GitHub：https://github.com/weiliu89/caffe/tree/ssd

基于“Proposal + Classification” 的 Object Detection 的方法，R-CNN 系列（R-CNN、SPPnet、Fast R-CNN 以及 Faster R-CNN），取得了非常好的结果，但是在速度方面离实时效果还比较远在提高 mAP 的同时兼顾速度，逐渐成为 Object Detection 未来的趋势。 YOLO 虽然能够达到实时的效果，但是其 mAP 与刚面提到的 state of art 的结果有很大的差距。 YOLO 有一些缺陷：每个网格只预测一个物体，容易造成漏检；对于物体的尺度相对比较敏感，对于尺度变化较大的物体泛化能力较差。针对 YOLO 中的这些不足，该论文提出的方法 SSD 在这两方面都有所改进，同时兼顾了 mAP 和实时性的要求。在满足实时性的条件下，接近 state of art 的结果。对于输入图像大小为 300*300 在 VOC2007 test 上能够达到 58 帧每秒( Titan X 的 GPU )，72.1% 的 mAP。输入图像大小为 500 x 500 , mAP 能够达到 75.1%。

作者的思路就是Faster R-CNN+YOLO，利用 YOLO 的思路和 Faster R-CNN 的 anchor box 的思想。

论文采用 VGG16 为基础网络结构，使用前面的前 5 层，然后利用 astrous 算法将 fc6 和 fc7 层转化成两个卷积层。再额外增加了 3 个卷积层，和一个 average pool 层。 **不同层次的 feature map 分别用于 default box 的偏移以及不同类别得分的预测**，最后通过 nms得到最终的检测结果。

![SSD net][31]

这些增加的卷积层的 feature map 的大小变化比较大，允许能够检测出不同尺度下的物体。SSD去掉了全连接层，每一个输出只会感受到目标周围的信息，包括上下文。这样来做就增加了合理性。并且不同的feature map，预测不同宽高比的图像，这样比YOLO增加了预测更多的比例的box。

![SSD net][32]

**多尺度feature map得到 default boxs及其 4个位置偏移和21个类别置信度**
对于不同尺度 feature map（ 上图中 38x38x512，19x19x512, 10x10x512, 5x5x512, 3x3x512, 1x1x256） 的上的所有特征点： 以 5x5x256 为例 它的 defalut_boxes = 6，其 Detector & classifier 4 展开结构如下：

![ssd][33]

首先按照不同的 scale 和 ratio 生成 k 个 default boxes。
## YOLOv2
论文链接：[YOLO9000: Better, Faster, Stronger][44]
GitHub：https://github.com/pjreddie/darknet
     or https://github.com/philipperemy/yolo-9000


不管是 Faster R-CNN 还是 SSD，它们生成的锚框仍然有大量是相互重叠的，从而导致仍然有大量的区域被重复计算了。YOLO 试图来解决这个问题。它将图片特征均匀的切成 $S \times S $ 块，每一块当做一个锚框。每个锚框预测 B 个边框，以及这个锚框主要包含哪个物体。整体架构如下图所示：

<div align = center>
  <img src = "./yolo.svg"/>
  <p> </p>
</div>

这是 YOLO 的解决方案， 而 YOLO v2 在原来的基础上对 YOLO 进行一些地方的改进，其主要包括：

1. 使用更好的卷积神经网络来做特征提取，使用更大输入图片$448\times 448$使得特征输出大小增大到$13\times 13$
2. 不再使用均匀切来的锚框，而是对训练数据里的真实锚框做聚类，然后使用聚类中心作为锚框。相对于SSD和Faster R-CNN来说可以大幅降低锚框的个数。
3. 不再使用YOLO的全连接层来预测，而是同SSD一样使用卷积。例如假设使用5个锚框（聚类为5类），那么物体分类使用通道数是`5*(1+num_classes)`的$1\times 1$卷积，边框回归使用通道数`4*5`.

## mask-RCNN
论文链接：[Mask R-CNN][45]
GitHub：https://github.com/matterport/Mask_RCNN
     or https://github.com/CharlesShang/FastMaskRCNN


Mask R-CNN 在 Faster R-CNN 上加入了一个新的像素级别的预测层，它不仅对一个锚框预测它对应的类和真实的边框，而且它会判断这个锚框类每个像素对应的哪个物体还是只是背景。后者是语义分割要解决的问题。Mask R-CNN 使用了 [全连接网络（FCN）][43] 来完成这个预测。当然这也意味这训练数据必须有像素级别的标注，而不是简单的边框。

<div align = center>
  <img src = "./mask-rcnn.svg"/>
  <p> </p>
</div>

因为 FCN 会精确预测每个像素的类别，就是输入图片中的每个像素都会在标注中对应一个类别。对于输入图片中的一个锚框，我们可以精确的匹配到像素标注中对应的区域。但是 PoI 池化是作用在卷积之后的特征上，其默认是将锚框做了定点化。例如假设选择的锚框是 $(x,y,w,h)$，且特征抽取将图片变小了 16 倍，就是如果原始图片是 $256\times 256$，那么特征大小就是 $16\times 16$ 。这时候在特征上对应的锚框就是变成了 $(\lfloor x/16 \rfloor, \lfloor y/16 \rfloor, \lfloor w/16 \rfloor, \lfloor h/16 \rfloor)$。如果 $x,y,w,h$ 中有任何一个不被 16 整除，那么就可能发生错位。同样道理，在上面的样例中我们看到，如果锚框的长宽不被池化大小整除，那么同样会定点化，从而带来错位。

通常这样的错位只是在几个像素之间，对于分类和边框预测影响不大。但对于像素级别的预测，这样的错位可能会带来大问题。Mask R-CNN 提出一个 RoI Align 层，它类似于 RoI 池化层，但是去除掉了定点化步骤，就是移除了所有 $\lfloor \cdot \rfloor$。如果计算得到的表框不是刚好在像素之间，那么我们就用四周的像素来线性插值得到这个点上的值。

对于一维情况，假设我们要计算 $x$ 点的值 $f(x)$，那么我们可以用 $x$ 左右的整点的值来插值：

$$f(x) = (\lfloor x \rfloor + 1-x)f(\lfloor x \rfloor) + (x-\lfloor x \rfloor)f(\lfloor x \rfloor + 1)$$

我们实际要使用的是二维差值来估计 $f(x,y)$，我们首先在 $x$ 轴上差值得到 $f(x,\lfloor y \rfloor)$ 和 $f(x,\lfloor y \rfloor+1)$ ，然后根据这两个值来差值得到 $f(x, y)$ .

本博文参考：

[【目标检测】RCNN算法详解][34]
[object detection（物体检测）系列论文梳理][35]
[SPPNet-引入空间金字塔池化改进RCNN][36]
[【目标检测】Fast RCNN算法详解][37]
[CNN图像分割进化史：三年从R-CNN到Mask R-CNN][38]
[【目标检测】Faster RCNN算法详解][39]
[Faster R-CNN][40]
[论文阅读：SSD: Single Shot MultiBox Detector][41]
[SSD][42]


  [1]: https://handong1587.github.io/deep_learning/2015/10/09/object-detection.html
  [2]: http://static.zybuluo.com/guoxs/uu2v6jtf728ebyt2puax90g6/1405.jpg
  [3]: https://arxiv.org/pdf/1405.0312.pdf
  [4]: https://arxiv.org/pdf/1311.2524.pdf
  [5]: http://static.zybuluo.com/guoxs/22tav9iv3t2332w4x3byv1ge/1.png
  [6]: https://ivi.fnwi.uva.nl/isis/publications/2013/UijlingsIJCV2013/UijlingsIJCV2013.pdf
  [7]: http://static.zybuluo.com/guoxs/xbzmn9wtj0ubr8tvz0aeowdi/20160405212106908
  [8]: https://www.koen.me/research/selectivesearch/
  [9]: http://static.zybuluo.com/guoxs/lih88fm7564tp9x2bvmmh4mv/%E7%BD%91%E7%BB%9C%E7%BB%93%E6%9E%84
  [10]: http://blog.csdn.net/u011534057/article/details/51222112
  [11]: https://arxiv.org/pdf/1406.4729.pdf
  [12]: http://static.zybuluo.com/guoxs/lxnbgcrp73kwe7l2za7hy4e3/5.png
  [13]: http://static.zybuluo.com/guoxs/5ykm2xgihotfz0esgwmt1lhi/3.png
  [14]: http://static.zybuluo.com/guoxs/6u32kk61jpltyzpm6oszdqcm/4.png
  [15]: https://zhuanlan.zhihu.com/p/24780433
  [16]: https://arxiv.org/pdf/1504.08083.pdf
  [17]: http://static.zybuluo.com/guoxs/epzx9t31ix6kqmmb75oh067r/6.png
  [18]: http://static.zybuluo.com/guoxs/i439nsl3txml8ipuy0gwfhg6/7
  [19]: http://static.zybuluo.com/guoxs/w0aq7xupwubp0bvuvkxtfgo8/8
  [20]: https://arxiv.org/pdf/1506.01497.pdf
  [21]: http://static.zybuluo.com/guoxs/x60zyw03d5kt2p3qo1kpdth4/9.png
  [22]: http://static.zybuluo.com/guoxs/m6j9qna59j4xc9yxwy0lgpwe/10.png
  [23]: http://static.zybuluo.com/guoxs/6kiu95c4r33u7mixnc5c5q5n/11.jpg
  [24]: http://static.zybuluo.com/guoxs/2hubieeboju17q4g7mwsh4u4/12.png
  [25]: http://static.zybuluo.com/guoxs/vs5fus7yrfw51vwt7xnqma95/13.png
  [26]: https://arxiv.org/pdf/1506.02640.pdf
  [27]: http://static.zybuluo.com/guoxs/lisbjatk5vqkawhjqco8iyfz/14.png
  [28]: http://static.zybuluo.com/guoxs/59m2b9ocmiw29o4s5o3bx87m/15.png
  [29]: http://static.zybuluo.com/guoxs/h70xkvhtzazf3ufi69n1vqer/loss
  [30]: https://arxiv.org/pdf/1512.02325.pdf
  [31]: http://static.zybuluo.com/guoxs/wk33gcqhji7g25vx25o9vw7y/17.png
  [32]: http://static.zybuluo.com/guoxs/3rwb6wm3hxeq7xrs1cc8a66u/1.png
  [33]: http://static.zybuluo.com/guoxs/620dyv9vpyfr2tfxsyyliti2/ssd1.jpg
  [34]: http://blog.csdn.net/shenxiaolu1984/article/details/51066975
  [35]: http://blog.csdn.net/zhang_shuai12/article/details/52554604
  [36]: https://zhuanlan.zhihu.com/p/24774302?refer=xiaoleimlnote
  [37]: http://blog.csdn.net/shenxiaolu1984/article/details/51036677
  [38]: https://zhuanlan.zhihu.com/p/26655034
  [39]: http://blog.csdn.net/shenxiaolu1984/article/details/51152614
  [40]: https://zhuanlan.zhihu.com/p/24916624
  [41]: http://blog.csdn.net/u010167269/article/details/52563573
  [42]: https://zhuanlan.zhihu.com/p/24954433?refer=xiaoleimlnote
  [43]: https://arxiv.org/abs/1411.4038
  [44]: https://arxiv.org/abs/1612.08242
  [45]: https://arxiv.org/abs/1703.06870
