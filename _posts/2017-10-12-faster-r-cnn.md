---
layout: post
mathjax: true
category: deep_learning
title: Faster R-CNN
tags: faster_r-cnn deep_learning object_detection
author: Alvin Zhu
date: 2017-10-12
---

* content
{:toc}

Faster R-CNN全文翻译。





# Faster R-CNN：利用区域提案网络实现实时目标检测

## 摘要

目前最先进的目标检测网络需要先用区域提案算法推测目标位置，像SPPnet[^1]和Fast R-CNN[^2]这些网络已经减少了检测网络的运行时间，这时计算区域提案就成了瓶颈问题。本文中，我们介绍一种**区域提案网络（Region Proposal Network, RPN）**，它和检测网络共享全图的卷积特征，使得区域提案几乎不花时间。RPN是一个全卷积网络，在每个位置同时预测目标边界和objectness得分。RPN是端到端训练的，生成高质量区域提案框，用于Fast R-CNN来检测。我们通过共享其卷积特征进一步将RPN和Fast R-CNN合并到一个网络中。使用最近流行的神经网络术语“注意力”机制，RPN模块告诉统一网络需要看哪里。对于非常深的VGG-16模型[^3]，我们的检测系统在GPU上的帧率为5fps（包含所有步骤），在PASCAL VOC 2007、PASCAL VOC 2012和MS COCO数据集上实现了最先进的目标检测准确率，每个图像用了300个提案框。在ILSVRC和COCO 2015比赛中，Faster R-CNN和RPN是几个比赛的第一名方法的基础。[代码](https://github.com/ShaoqingRen/faster_rcnn)已公开。

## 1.引言

最近在目标检测中取得的进步都是由区域提案方法（例如[^4]）和基于区域的卷积神经网络（R-CNN）[^5]取得的成功来推动的。基于区域的CNN在[^5]中刚提出时在计算上消耗很大，幸好后来这个消耗通过提案框之间共享卷积[^1] [^2]大大降低了。最近的Fast R-CNN[^2]用非常深的网络[^3]实现了近实时检测的速率，注意它忽略了生成区域提案框的时间。现在，**提案框是最先进的检测系统中的计算瓶颈**。

区域提案方法典型地依赖于消耗小的特征和经济的获取方案。选择性搜索（Selective Search, SS）[^4]是最流行的方法之一，它基于设计好的低级特征贪心地融合超级像素。与高效检测网络[^2]相比，SS要慢一个数量级，CPU应用中大约每个图像2s。EdgeBoxes[^6]在提案框质量和速度之间做出了目前最好的权衡，大约每个图像0.2s。但无论如何，区域提案步骤花费了和检测网络差不多的时间。
Fast R-CNN利用了GPU，而区域提案方法是在CPU上实现的，这个运行时间的比较是不公平的。一种明显提速生成提案框的方法是在GPU上实现它，这是一种工程上很有效的解决方案，但这个方法忽略了其后的检测网络，因而也错失了共享计算的重要机会。

本文中，我们改变了算法——**用深度网络计算提案框**——这是一种简洁有效的解决方案，提案框计算几乎不会给检测网络的计算带来消耗。为了这个目的，我们介绍新颖的区域提案网络（Region Proposal Networks, RPN），它与最先进的目标检测网络[^1] [^2]共享卷积层。在测试时，通过共享卷积，计算提案框的边际成本是很小的（例如每个图像10ms）。

我们观察发现，基于区域的检测器例如Fast R-CNN使用的卷积（conv）特征映射，同样可以用于生成区域提案。我们紧接着这些卷积特征增加一些额外的卷积层来构造RPN：这些层在每个卷积映射网格上同时预测objectness得分和回归边界。 我们的RPN是一种**全卷积网络**（fully-convolutional network, FCN）[^7]，可以针对生成检测提案框的任务端到端地训练。

![Figure 1](/assets/2017-10-12-faster-r-cnn/figure1.png)

图1. 用于解决多种尺度和尺寸的不同方案。（a）构建了金字塔的图像和特征图，分类器在所有尺度上运行。 （b）在特征图上运行具有多个刻度/尺寸的卷积的金字塔。 （c）我们在回归函数中使用参考框的金字塔。

RPN旨在有效地预测具有广泛尺度和纵横比的区域分布。与使用图像的金字塔（图1，a）或卷积的金字塔（图1，b）的流行方法[^8] [^9] [^1] [^2]相比，我们引入了新的“锚点”作为多尺度和纵横比的参考。我们的方案可以被认为是一个回归参考金字塔（图1，c），它避免了枚举多个尺度或纵横比的图像或卷积。当使用单尺度图像进行训练和测试时，该模型表现良好，从而有利于运行速度。

为了统一RPN和Fast R-CNN[^2]目标检测网络，我们提出一种简单的训练方案，即**保持提案框固定，微调区域提案和微调目标检测之间交替进行**。这个方案收敛很快，最后形成可让两个任务共享卷积特征的标准网络。

我们在PASCAL VOC检测标准集[^11]上评估我们的方法， Fast R-CNN结合RPN的检测准确率超过了作为强大基准的Fast R-CNN结合SS的方法。同时，我们的方法没有了SS测试时的计算负担，对于生成提案框的有效运行时间只有10毫秒。利用[^3]中网络非常深的深度模型，我们的检测方法在GPU上依然有5fps的帧率（包括所有步骤），因此就速度和准确率而言，这是一个实用的目标检测系统。我们还评估了MS COCO数据集[^12]的结果，并使用COCO数据对PASCAL VOC的改进进行了评估。[MATLAB版本](https://github.com/shaoqingren/faster_rcnn)和[Python版本](https://github.com/rbgirshick/py-faster-rcnn)的代码已经公开提供。

以前，这份手稿的初步版本已经公布[^10]。从那时起，RPN和Faster R-CNN的框架已被采用并通用于其他方法，如3D目标检测[^13]，基于部分的检测[^14]，目标分割[^15]和图像字幕[^16]。我们的快速有效的物体检测系统也已经在诸如Pinterest[^17]等商业系统中使用，有了用户的参与与改进。

在ILSVRC和COCO 2015比赛中，Faster R-CNN和RPN是ImageNet检测，ImageNet定位，COCO检测和COCO分割的第一名所采用的方法的基础。 RPN完全从数据中学习提出区域，从而可以从更深层次和更具表现力的特征（如[^18]中采用的101层残差网络）中轻松获益。Faster R-CNN和RPN也被这些比赛的其他几个主要参赛作品使用([http://image-net.org/challenges/LSVRC/2015/results](http://image-net.org/challenges/LSVRC/2015/results))。这些结果表明，我们的方法不仅实用，而且是提高目标检测精度的有效方法。

## 2.相关工作

**目标提案。**有关于目标提案方法的大量文献。目标提案方法的综合调查和比较可以在[^19]，[^20]，[^21]中找到。广泛使用的目标提案方法包括基于分组超像素（例如，选择性搜索[^4]，CPMC[^22]，MCG[^23]）和基于滑动窗口的目标提案方法（例如，窗口中的目标[^24]，EdgeBoxes[^6]）。目标提案方法被采用为独立于检测器的外部模块（例如，选择性搜索[4]目标检测器，R-CNN[^5]和Fast R-CNN[^2]）。

**深度网络目标检测。**R-CNN方法[^5]使用CNN端到端地将提案区域分类为目标类别或背景。 R-CNN主要作为分类器，它不预测目标边界（除了通过边界框回归进行细化）。其准确性取决于区域提案模块的性能（参见[^20]中的比较）。几篇论文提出了使用深层网络预测检测框的方法[^25] [^9] [^26] [^27]。在OverFeat方法[^9]中，训练全连接层以预测假定单目标定位任务的框坐标。全连接层然后被变成用于检测多种类别目标的卷积层。MultiBox方法[^26] [^27]的网络从最后一个全连接层同时预测多个类别无关框，是对OverFeat的但目标模式的推广。这些类别无关框被用作R-CNN的提案[^5]。与我们的全卷积方案相比，MultiBox提案网络应用于单个图像块或多个大图像块（例如，$224 \times 224$）。 MultiBox不共享提案和检测网络之间的特征。我们在后文中讲我们的方法时会更深层次地讨论OverFeat和MultiBox。与我们的工作同时进行的DeepMask方法[^28]被用于学习分割提案。

卷积的共享计算[^9] [^1] [^29] [^7] [^2]高效、精确，已经在视觉识别方面吸引了越来越多的注意。OverFeat论文[^9]从图像金字塔计算卷积特征，用于分类、定位、检测。在共享的卷积特征映射上自适应大小的pooling（SPP）[^1]能有效用于基于区域的目标检测[^1] [^30]和语义分割[^29]。Fast R-CNN[^2]实现了在共享卷积特征上训练的端到端检测器，显示出令人惊叹的准确率和速度。

## 3.Faster R-CNN

我们的目标检测系统称为Faster R-CNN，由两个模块组成。第一个模块是提出区域提案的深度全卷积网络，第二个模块是使用区域提案的Fast R-CNN检测器[^2]。整个系统是一个统一的目标检测网络（图2）。使用最近流行的神经网络术语“注意力”[^31]机制，RPN模块告诉Fast R-CNN模块要看哪里。在3.1节中，我们介绍了区域提案网络的设计和属性。在3.2节中，我们介绍用于训练具有共享特征的两个模块的算法。

![Figure 2](/assets/2017-10-12-faster-r-cnn/figure2.png)

图2. Faster R-CNN是用于目标检测的单个统一网络。 RPN模块作为统一网络的“注意力”。

### 3.1区域提案网络

区域提案网络（RPN）将一个图像（任意大小）作为输入，输出矩形目标提案框的集合，每个框有一个objectness得分（“区域”是一个通用术语，在本文中，我们只考虑矩形区域，这与许多方法是一致的（例如[^27] [^4] [^6]）。 “objectness”衡量一组目标类与背景的成员关系。）。我们用全卷积网络[^7]对这个过程构建模型，本章会详细描述。因为我们的最终目标是和Fast R-CNN目标检测网络[^2]共享计算，所以假设**这两个网络共享一系列卷积层**。在实验中，我们详细研究Zeiler和Fergus的模型[^32]（ZF），它有5个可共享的卷积层，以及Simonyan和Zisserman的模型[^3]（VGG），它有13个可共享的卷积层。

为了生成区域提案框，我们在最后一个共享的卷积层输出的卷积特征映射上滑动小网络，这个网络连接到输入卷积特征映射的$n \times n$的空间窗口上。**每个滑动窗口映射到一个低维向量上**（对于ZF是256-d，对于VGG是512-d，后面接一个ReLU[^33]）。这个向量输出给两个同级的全连接的层：检测框回归层（reg）和检测框分类层（cls）。本文中$n=3$，注意图像的有效感受野很大（ZF是171像素，VGG是228像素）。图3（左）以这个小网络在某个位置的情况举了个例子。注意，由于小网络是滑动窗口的形式，所以全连接层（$n \times n$的）被所有空间位置共享（指所有位置用来计算内积的$n \times n$的层参数相同）。这种结构实现为$n \times n$的卷积层，后接两个同级的$1 \times 1$的卷积层（分别对应reg和cls），ReLU[^33]应用于$n \times n$卷积层的输出。

![Figure 3](/assets/2017-10-12-faster-r-cnn/figure3.png)

图3：左：区域提案网络（RPN）。右：用RPN提案框在PASCAL VOC 2007测试集上的检测实例。我们的方法可以在很大范围的尺度和长宽比中检测目标。

#### 3.1.1锚点(Anchor)

在每一个滑动窗口的位置，我们同时预测k个区域提案，所以reg层有$4k$个输出，即$k$个box的坐标编码。cls层输出$2k$个得分，即对每个提案框是目标/非目标的估计概率（为简单起见，是用二分类的Softmax层实现的cls层，也可以用Logistic回归来生成$k$个得分）。$k$个提案框被相应的$k$个称为anchor的box参数化。每个anchor以当前滑动窗口中心为中心，并对应一种尺度和长宽比（图3，左），默认情况下，我们使用3种尺度和3种长宽比，这样在每一个滑动位置就有$k=9$个anchor。对于大小为$W \times H$（典型值约2,400）的卷积特征映射，总共有$WHk$个anchor。

**平移不变锚点**

我们的方法有一个重要特性，就是平移不变性，对anchor和对计算anchor相应的提案框的函数而言都是这样。如果平移了图像中的目标，提案框也应该平移，也应该能用同样的函数预测提案框。我们的方法确保了这种平移不变的属性（如FCN[^7]的情况，在网络的总体步幅以内，我们的网络是平移不变的。）。作为比较，MultiBox方法[27]用k-means生成800个anchor，但不具有平移不变性。因此，MultiBox不具有平移不变性。

平移不变性也减少了模型大小。 MultiBox有$(4 + 1) \times 800$维全连接输出层，而在k = 9个锚点的情况下，我们的方法有$(4 + 2) \times 9$维的卷积输出层。因此，我们的输出层具有$2.8 \times 10^4$个参数（VGG-16为$512 \times (4 + 2) \times 9$），比具有$6.1 \times 10^6$个参数的MultiBox输出层（MultiBox[^27]使用的GoogleNet[^34]为$1536 \times (4 + 1) \times 800$）少两个数量级。如果考虑特征提取层，我们的提案层的参数比MultiBox （考虑到特征提取层，我们的提案层的参数计数为$3 \times 3 \times 512 \times 512 + 512 \times 6 \times 9 = 2.4 \times 10^6$，MultiBox的提案图层参数计数为$7 \times 7 \times (64 + 96 + 64 + 64) \times 1536 + 1536 \times 5 \times 800 = 27 \times 10^6$。）的参数还要小一个数量级。这样在PASCAL VOC这种小数据集上出现过拟合的风险较小。

**多尺度锚点作为回归参考**

我们的锚定设计提出了一种解决多尺度（和高宽比）的新方案。如图1所示，已经有两种流行的多尺度预测方式。第一种方法是基于图像/特征金字塔，例如在DPM[^8]和基于CNN的方法[^9] [^1] [^2]中。图像以多尺度调整大小，并且为每个尺度计算特征图（HOG[^8]或深度卷积特征[^9] [^1] [^2]）（图1（a））。这种方式通常是有效的，但是耗时。第二种方法是在特征图上使用多个尺度（和/或纵横比）的滑动窗口。例如，在DPM[^8]中，使用不同的卷积核尺寸（如$5 \times 7$和$7 \times 5$）分别对不同宽高比的模型进行了训练。如果用这种方式来处理多个尺度，就可以将其视为“卷积核金字塔”（图1（b））。第二种方式通常与第一种方式一起使用[^8]。

作为比较，我们基于锚点的方法建立在一个锚点金字塔上，这更具成本效益。我们的方法参照多个尺度和纵横比的锚点框分类和回归边界框。它仅依赖于单个尺度的图像和特征图，并使用单个尺寸的卷积（特征图上的滑动窗口）。我们通过实验展示了该方案对于多种尺度和尺寸的影响（表8）。

由于这种基于锚点的多尺度设计，我们可以简单地使用单尺度图像上的卷积特征，这也是Fast R-CNN检测器[^2]所完成的。多尺度锚点的设计是共享特征的关键组件，无需额外的成本来缩放尺寸。

#### 3.1.2损失函数

为了训练RPN，我们给每个anchor分配一个二值的标签（是不是目标）。我们分配**正标签**给两类anchor：（i）与检测框真值IoU最高的anchor（ii）与任意检测框真值有大于0.7的IoU交叠的anchor。注意到一个检测框真值可能分配正标签给多个anchor。通常第二个条件足以确定正样本。但是我们仍然采取第一个条件，因为在极少数情况下，第二个条件可能没有发现正样本。我们分配**负标签**给与所有检测框真值的IoU比率都低于0.3的anchor。非正非负的anchor对训练目标没有任何作用。

有了这些定义，我们遵循Fast R-CNN[^5]中的多任务损失，最小化目标函数。我们对一个图像的损失函数定义为 
$$
L(\lbrace p_i \rbrace, \lbrace t_i \rbrace) = \frac{1}{N_{cls}} \sum_i L_{cls}(p_i, p_i^{\ast}) + \lambda \frac{1}{N_{reg}} \sum_i  p_i^{\ast} L_{reg}(t_i, t_i^{\ast}) \tag{1}
$$

这里，$i$是一个mini-batch中anchor的索引，$_pi$是anchor $i$是目标的预测概率。如果anchor为正，检测框真值标签$p_i^{\ast}$就是1，如果anchor为负，$p_i^{\ast}$就是0。$t_i$是一个向量，表示预测的检测框的4个参数化坐标，$t_i^{\ast}$是与正anchor对应的检测框真值的坐标向量。分类损失$L_{cls}$是两个类别（目标vs.非目标）的对数损失。对于回归损失，我们用$L_{reg}(t_i, t_i^{\ast}) = R(t_i - t_i^{\ast})$来计算，其中$R$是[^2]中定义的鲁棒的损失函数（smooth $L_1$）。$ p_i^{\ast} L_{reg}$这一项意味着只有正anchor($P_i^{\ast} = 1$)才有回归损失，其他情况就没有($P_i^{\ast} = 0$)。cls层和reg层的输出分别由$\lbrace p_i \rbrace$和$\lbrace t_i \rbrace$组成。

这两项分别由$N_{cls}$和$N_{reg}$以及一个平衡权重λ归一化。目前的实现中（参见公开的代码），公式$(1)$中的cls项的归一化值为mini-batch的大小（即$N_{cls} = 256$），reg项的归一化值为anchor位置的数量（即$N_{reg}$约为2,400），默认情况下，$\lambda = 10$，这样cls和reg项差不多是等权重的。我们通过实验显示，结果对$\lambda$在很大范围内的值不敏感（表9）。我们还注意到，不需要上述的归一化，可以简化。

对于回归，我们依照[^5]采用4个坐标： 
$$
\begin{matrix}
 t_x = (x - x_a) / w_a , & t_y = (y - y_a) / h_a , \\ 
 t_w = \log(w / w_a), & t_h = \log(h / h_a), \\ 
 t_x^{\ast} = (x^{\ast} - x_a) / w_a , & t_y^{\ast} = (y^{\ast} - y_a) / h_a , \\ 
 t_w^{\ast} = \log(w^{\ast} / w_a), & t_h^{\ast} = \log(h^{\ast} / h_a), 
\end{matrix} \tag{2}
$$

$x$，$y$，$w$，$h$指的是包围盒中心的坐标、宽、高。变量$x$，$x_a$，$x^{\ast}$分别指预测的检测框、anchor box、检测框真值（就像$y$，$w$，$h$一样）。可以理解为从anchor box到附近的检测框真值的检测框回归。 

无论如何，我们用了一种与之前的基于RoI的方法[^1] [^2]不同的方法实现了检测框回归算法。在[^1] [^2]中，检测框回归是通过从任意大小的区域中池化特征实现的，回归权重是所有不同大小的区域共享的。在我们的方法中，用于回归的特征在特征映射中具有相同的空间大小($3 \times 3$)。考虑到各种不同的大小，需要学习一系列$k$个检测框回归量。每一个回归量对应于一个尺度和长宽比，$k$个回归量之间不共享权重。因此，即使特征具有固定的尺寸/尺度，预测各种尺寸的检测框仍然是可能的，这要归功于anchor的设计。

#### 3.1.3训练RPN

RPN可以通过反向传播和随机梯度下降(SGD)[^35]端到端训练。我们遵循[2]中的“image-centric”采样策略训练这个网络。每个mini-batch由包含了许多正负anchor样本的单个图像组成。我们可以优化所有anchor的损失函数，但是这会偏向于负样本，因为它们是主要的。因此，我们随机地在一个图像中采样256个anchor，计算mini-batch的损失函数，其中采样的正负anchor的比例最多是1:1。如果一个图像中的正样本数小于128，我们就用负样本填补这个mini-batch。

我们通过从零均值标准差为0.01的高斯分布中获取的权重来随机初始化所有新层（最后一个卷积层其后的层），所有其他层（即共享的卷积层）是通过对ImageNet分类[^36]预训练的模型来初始化的，这也是标准惯例[^5]。我们调整ZF网络的所有层，VGG网络的conv3\_1以上的层，以节约内存[2]。我们在PASCAL数据集上对于60k个mini-batch用的学习率为0.001，对于下一20k个mini-batch用的学习率是0.0001。动量是0.9，权重衰减为0.0005[^37]。我们的实现使用了Caffe[^38]。

### 3.2共享RPN与Fast R-CNN的特征

迄今为止，我们已经描述了如何为生成区域提案训练网络，而没有考虑基于区域的目标检测CNN如何利用这些提案框。对于检测网络，我们采用Fast R-CNN[^2]，现在描述一种算法，学习由RPN和Fast R-CNN之间共享的卷积层（图2）。 

RPN和Fast R-CNN都是独立训练的，要用不同方式修改它们的卷积层。因此我们需要开发一种允许两个网络间共享卷积层的技术，而不是分别学习两个网络。我们讨论三种训练具有共享特征的网络的解决方案：

1. 交替训练。在这个解决方案中，我们首先训练RPN，并使用提案训练Fast R-CNN。然后，使用Fast R-CNN微调过后的网络初始化RPN，并重复此过程。这是本文所有实验中使用的解决方案。
2. 近似联合训练。在这个解决方案中，RPN和Fast R-CNN网络在训练期间被合并到一个网络中，如图2所示。在每个SGD迭代中，前向传递产生区域提案，在训练时被视为固定的，训练Fast R-CNN检测器前预先计算提案。反向传播像往常一样发生，其中对于共享层，反向传播信号为来自RPN的损失和Fast R-CNN的损失的组合。这个解决方案很容易实现。但是这个解决方案忽略了衍生的w.r.t.提案框的坐标也是网络响应，所以是近似。在我们的实验中，我们发现这个解决方案产生了相近的结果，与交替训练相比，训练时间减少了约25-50％。该解决方案包含在我们发布的Python代码中。
3. 非近似联合训练。如上所述，由RPN预测的检测框也是输入的函数。 Fast R-CNN中的RoI池化层[^2]接受卷积特征以及预测的检测框作为输入，因此理论上有效的反向传播求解器也应该包含梯度w.r.t.框坐标。这些梯度在上述近似联合训练中被忽略。在非近似联合培训解决方案中，我们需要一个可区分w.r.t.框坐标的RoI池化层。这是一个非常重要的问题，解决方案可以由[^15]中开发的“RoI缩放”层给出，这超出了本文的范围。

**四步交替训练。**我们开发了一种实用的4步训练算法，通过交替优化来学习共享的特征。 第一步，我们依上述训练RPN，如3.1.3节所述。该网络用ImageNet预训练的模型初始化，并端到端微调用于区域提案任务。第二步，我们利用第一步的RPN生成的提案框，由Fast R-CNN训练一个单独的检测网络，这个检测网络同样是由ImageNet预训练的模型初始化的，这时候两个网络还没有共享卷积层。第三步，我们用检测网络初始化RPN训练，但我们固定共享的卷积层，并且只微调RPN独有的层，现在两个网络共享卷积层了。第四步，保持共享的卷积层固定，微调Fast R-CNN的fc层。这样，两个网络共享相同的卷积层，构成一个统一的网络。类似的交替训练可以运行更多的迭代，但是我们已经观察到的改进已经微乎其微了。

### 3.3实现细节

我们训练、测试区域提案和目标检测网络都是在单一尺度的图像上[^1] [^2]。我们缩放图像，让它们的短边$s = 600$像素[^2]。多尺度特征提取可能提高准确率但是不利于速度与准确率之间的权衡[^2]。我们也注意到ZF和VGG网络，对缩放后的图像在最后一个卷积层的总步长为16像素，这样相当于一个典型的PASCAL图像（约$500 \times 375$）上大约10个像素（$600/16=375/10$）。即使是这样大的步长也取得了好结果，尽管若步长小点准确率可能得到进一步提高。 

对于anchor，我们用3个简单的尺度，包围盒面积为$128^2$，$256^2，$512^2，和3个简单的长宽比，1:1，1:2，2:1。这些超参数不是特定数据集的选择，我们在下一节提供其影响的消融实验。讨论过，我们的解决方案不需要图像金字塔或卷积金字塔来预测多个尺度的区域，从而节省相当长的运行时间。图3（右）显示了我们的算法处理多种尺度和长宽比的能力。表1显示了用ZF网络对每个anchor学到的平均提案框大小。我们注意到，我们的算法允许预测框大于感受野。这样的预测并不是不可能的 - 如果只有目标的中间是可见的，那么仍然可以粗略地推断目标的范围。

![Table 1](/assets/2017-10-12-faster-r-cnn/table1.png)

表1. 使用ZF网络对每个anchor学到的平均提案框大小（$s = 600$）。

跨越图像边界的anchor包围盒要小心处理。在训练中，我们忽略所有跨越图像边界的anchor，这样它们不会对损失有影响。对于一个典型的$1000 \times 600$的图像，差不多总共有20k（约$60 \times 40 \times 9）anchor。忽略了跨越边界的anchor以后，每个图像只剩下6k个anchor需要训练了。如果跨越边界的异常值在训练时不忽略，就会带来又大又困难的修正误差项，训练也不会收敛。在测试时，我们还是应用全卷积的RPN到整个图像中，这可能生成跨越边界的提案框，我们将其裁剪到图像边缘位置。 

有些RPN提案框和其他提案框大量重叠，为了减少冗余，我们基于提案区域的cls得分，对其采用非极大值抑制（non-maximum suppression, NMS）。我们固定对NMS的IoU阈值为0.7，这样每个图像只剩2k个提案区域。正如下面展示的，NMS不会影响最终的检测准确率，但是大幅地减少了提案框的数量。NMS之后，我们用提案区域中的top-N个来检测。在下文中，我们用2k个RPN提案框训练Fast R-CNN，但是在测试时会对不同数量的提案框进行评价。

## 4.实验

### 4.1在PASCAL VOC上的实验

我们在PASCAL VOC2007检测基准[^11]上综合评价我们的方法。此数据集包括20个目标类别，大约5k个trainval图像和5k个test图像。我们还对少数模型提供PASCAL VOC2012基准上的结果。对于ImageNet预训练网络，我们用“fast”版本的ZF网络[^32]，有5个卷积层和3个 fc层，公开的VGG-16模型([www.robots.ox.ac.uk/\~vgg/research/very deep/](www.robots.ox.ac.uk/\~vgg/research/very deep/))[^3]，有13 个卷积层和3 个fc层。我们主要评估检测的平均精度（mean Average Precision, mAP），因为这是对目标检测的实际度量标准（而不是侧重于目标提案框的代理度量）。 

表2（上）显示了使用各种区域提案的方法训练和测试时Fast R-CNN的结果。这些结果使用的是ZF网络。对于选择性搜索（SS）[^4]，我们用“fast”模式生成了2k个左右的SS提案框。对于EdgeBoxes（EB）[^6]，我们把默认的EB设置调整为0.7IoU生成提案框。SS的mAP 为58.7％，EB的mAP 为58.6％。RPN与Fast R-CNN实现了有竞争力的结果，当使用300个提案框时的mAP就有59.9％（对于RPN，提案框数量，如300，是一个图像产生提案框的最大数量。RPN可能产生更少的提案框，这样提案框的平均数量也更少了）。使用RPN实现了一个比用SS或EB更快的检测系统，因为有共享的卷积计算；提案框较少，也减少了区域方面的fc消耗（表5）。

![Table 2](/assets/2017-10-12-faster-r-cnn/table2.png)

表2. PASCAL VOC2007年测试集的检测结果（在VOC2007 trainval训练）。该检测器是Fast R-CNN与ZF，但使用各种提案框方法进行训练和测试。

**RPN的消融试验。**为了研究RPN作为提案框方法的表现，我们进行了多次消融研究。首先，我们展示了RPN和Fast R-CNN检测网络之间共享卷积层的影响。要做到这一点，我们在4步训练过程中的第二步后停下来。使用分离的网络时的结果稍微降低为58.7％（RPN+ ZF，非共享，表2）。我们观察到，这是因为在第三步中，当调整过的检测器特征用于微调RPN时，提案框质量得到提高。 
接下来，我们理清了RPN在训练Fast R-CNN检测网络上的影响。为此，我们用2k个SS提案框和ZF网络训练了一个Fast R-CNN模型。我们固定这个检测器，通过改变测试时使用的提案区域，评估检测的mAP。在这些消融实验中，RPN不与检测器共享特征。 
在测试时用300个RPN提案框替换SS，mAP为56.8％。mAP的损失是训练/测试提案框之间的不一致所致。该结果作为以下比较的基准。 
有些奇怪的是，在测试时使用排名最高的100个提案框时，RPN仍然会取得有竞争力的结果（55.1％），表明这种高低排名的RPN提案框是准确的。另一种极端情况，使用排名最高的6k个RPN提案框（没有NMS）取得具有可比性的mAP（55.2％），这表明NMS不会降低检测mAP，反而可以减少误报。 
接下来，我们通过在测试时分别移除RPN的cls和reg中的一个，研究它们输出的作用。当在测试时（因此没有用NMS/排名）移除cls层，我们从没有计算得分的区域随机抽取N个提案框。N =1k 时mAP几乎没有变化（55.8％），但当N=100则大大降低为44.6％。这表明，**cls得分是排名最高的提案框准确的原因**。 
另一方面，当在测试时移除reg层（这样的提案框就直接是anchor框了），mAP下降到52.1％。这表明，高品质的提案框主要归功于回归后的位置。单是anchor框不足以精确检测。 
我们还评估更强大的网络对RPN的提案框质量的作用。我们使用VGG-16训练RPN，并仍然使用上述SS+ZF检测器。mAP从56.8％（使用RPN+ZF）提高到59.2％（使用RPN+VGG）。这是一个满意的结果，因为它表明，RPN+VGG的提案框质量比RPN+ZF的更好。由于RPN+ZF的提案框是可与SS竞争的（训练和测试一致使用时都是58.7％），我们可以预期RPN+VGG比SS好。下面的实验证明这一假说。 
**VGG-16的性能。**表3展示了VGG-16对提案框和检测的结果。使用RPN+VGG，Fast R-CNN对不共享特征的结果是68.5％，比SS基准略高。如上所示，这是因为由RPN+VGG产生的提案框比SS更准确。不像预先定义的SS，RPN是实时训练的，能从更好的网络获益。对特征共享的变型，结果是69.9％——比强大的SS基准更好，提案框几乎无损耗。在PASCAL VOC2007 trainval和2012 trainval的并集上进一步训练RPN，mAP是73.2％。图5显示了PASCAL VOC 2007测试集的一些结果。跟[5]一样在VOC 2007 trainval+test和VOC2012 trainval的并集上训练时，我们的方法在PASCAL VOC 2012测试集上（表4）有70.4％的mAP。表6和表7显示了详细数字。

表5中我们总结整个目标检测系统的运行时间。SS需要1~2秒，取决于图像内容（平均1.51s），采用VGG-16的Fast R-CNN在2k个SS提案框上需要320ms（若是用了SVD在fc层的话只用223ms[^2]）。我们采用VGG-16的系统生成提案框和检测一共只需要198ms。卷积层共享时，RPN只用10ms来计算附加的几层。由于提案框较少（300），我们的区域计算花费也很低。我们的系统采用ZF网络时的帧率为17fps。

![Table 3](/assets/2017-10-12-faster-r-cnn/table3.png)

表3. 在PASCAL VOC 2007测试集上的检测结果，检测器是Fast R-CNN和VGG16。训练数据：“07”：VOC2007 trainval，“07+12”：VOC 2007 trainval和VOC 2012 trainval的并集。对RPN，用于Fast R-CNN训练时的提案框是2k。这在[^2]中有报告；利用本文所提供的仓库（repository），这个数字更高（68.1）。

![Table 4](/assets/2017-10-12-faster-r-cnn/table4.png)

表4. PASCAL VOC 2012测试集检测结果。检测器是Fast R-CNN和VGG16。训练数据：“07”：VOC 2007 trainval，“07++12”： VOC 2007 trainval+test和VOC 2012 trainval的并集。对RPN，用于Fast R-CNN训练时的提案框是2k。

![Table 5](/assets/2017-10-12-faster-r-cnn/table5.png)

表5.  K40 GPU上的用时（ms），除了SS提案框是在CPU中进行评价的。“区域方面”包括NMS，pooling，fc和softmax。请参阅我们发布的代码运行时间的分析。

![Table 6](/assets/2017-10-12-faster-r-cnn/table6.png)

表6. PASCAL VOC 2012测试集检测结果。检测器是Fast R-CNN和VGG16。对RPN，用于Fast R-CNN训练时的提案框是2k。$RPN^{\ast}$表示非共享特征版本。

![Table 7](/assets/2017-10-12-faster-r-cnn/table7.png)

表7. PASCAL VOC 2012测试集检测结果。检测器是Fast R-CNN和VGG16。对RPN，用于Fast R-CNN训练时的提案框是2k。

![Figure 5](/assets/2017-10-12-faster-r-cnn/figure5.png)

图5. 使用Faster R-CNN系统的PASCAL VOC 2007测试集上的目标检测结果的选定示例。模型为VGG-16，训练数据为07 + 12 trainval（2007的测试集mAP为73.2％）。我们的方法可以检测各种尺度和宽高比的物体。每个输出框与类别标签和$\lbrack 0, 1 \rbrack$的Softmax分数相关联。分数阈值为0.6。获取这些结果的运行时间为每个图像198ms，包括所有步骤。

**超参数敏感度。**在表8中，我们调查了锚点的设置。默认情况下，我们使用3个尺度和3个纵横比（表8中为69.9％mAP）。如果在每个位置只使用一个锚点，mAP将明显下降3-4％。如果使用3个尺度（1个纵横比）或3个纵横比（1个刻度），则mAP更高，表明使用多个尺寸的锚点作为回归参考是一个有效的解决方案。仅使用3个具有1个纵横比（69.8％）的尺度与在该数据集上使用3个长宽比的3个尺度一样好。但是我们仍然在设计中采用这两个维度来保持系统的灵活性。

在表9中，我们比较了公式$(1)$中的$\lambda$的不同值。默认情况下，我们使用$\lambda = 10$，这使得公式$(1)$中的两个项在归一化后大致相等地加权。表9显示，当$\lambda$在约两个数量级（1到100）的范围内时，我们的结果仅略微受到影响（约1％）。这表明结果在很大范围内对$\lambda$不敏感。

![Table 8](/assets/2017-10-12-faster-r-cnn/table8.png)

表8：使用不同anchor设置的PASCAL VOC 2007测试集中Faster R-CNN的检测结果。网络是VGG-16。训练数据是VOC 2007 train。使用3个尺度和3个宽高比（69.9％）的默认设置与表3中相同。

![Table 9](/assets/2017-10-12-faster-r-cnn/table9.png)

表9：使用公式$(1)$中不同$\lambda$值的PASCAL VOC 2007测试集中Faster R-CNN的检测结果。网络是VGG-16。培训数据是VOC 2007 train。使用$\lambda = 10$（69.9％）的默认设置与表3中相同。

**IoU召回率分析。**接下来，我们计算提案框与检测框真值在不同的IoU比例时的召回率。值得注意的是，该IoU召回率度量标准与最终的检测准确率只是松散[^19] [^20] [^21]相关的。更适合用这个度量标准来诊断提案框方法，而不是对其进行评估。 
在图4中，我们展示使用300，1k，和2k个提案框的结果。我们将SS和EB作比较，并且这N个提案框是基于用这些方法生成的按置信度排名的前N个。该图显示，当提案框数量由2k下降到300时，RPN方法的表现很好。这就解释了使用少到300个提案框时，为什么RPN有良好的最终检测mAP。正如我们前面分析的，这个属性主要是归因于RPN的cls项。当提案框变少时，SS和EB的召回率下降的速度快于RPN。

![Figure 4](/assets/2017-10-12-faster-r-cnn/figure4.png)

图4：PASCAL VOC 2007测试集上的召回率 vs. IoU重叠率

**单级的检测vs. 两级的提案框+检测。**OverFeat论文[^9]提出在卷积特征映射的滑动窗口上使用回归和分类的检测方法。OverFeat是一个单级的，类特定的检测流程，我们的是一个两级的，由类无关的提案框方法和类特定的检测组成的级联方法。在OverFeat中，区域方面的特征来自一个滑动窗口，对应一个尺度金字塔的一个长宽比。这些特征被用于同时确定物体的位置和类别。在RPN中，特征都来自相对于anchor的方形（$3 \times 3$）滑动窗口和预测提案框，是不同的尺度和长宽比。虽然这两种方法都使用滑动窗口，区域提案任务只是RPN + Fast R-CNN的第一级——检测器致力于改进提案框。在我们级联方法的第二级，区域一级的特征自适应地从提案框进行pooling[^1] [^5]，更如实地覆盖区域的特征。我们相信这些特征带来更准确的检测。 
为了比较单级和两级系统，我们通过单级的Fast R-CNN模拟OverFeat系统（因而也规避实现细节的其他差异）。在这个系统中，“提案框”是稠密滑动的，有3个尺度（128，256，512）和3个长宽比（1:1，1:2，2:1）。Fast R-CNN被训练来从这些滑动窗口预测特定类的得分和回归盒的位置。由于OverFeat系统采用多尺度的特征，我们也用由5个尺度中提取的卷积特征来评价。我们使用[^1] [^2]中一样的5个尺度。 
表10比较了两级系统和两个单级系统的变体。使用ZF模型，单级系统具有53.9％的mAP。这比两级系统（58.7％）低4.8％。这个实验证明级联区域提案方法和目标检测的有效性。类似的观察报告在[^2] [^39]中，在两篇论文中用滑动窗口取代SS区域提案都导致了约6％的下降。我们还注意到，单级系统比较慢，因为它有相当多的提案框要处理。

![Table 10](/assets/2017-10-12-faster-r-cnn/table10.png)

表10：单级检测vs.两级提案+检测。检测结果都是在PASCAL VOC2007测试集使用ZF模型和Fast R-CNN。RPN使用非共享的特征。

### 4.2在MS COCO上的实验

我们在Microsoft COCO目标检测数据集上提供更多的结果[^12]。此数据集涉及80个目标类别。我们对训练集上的80k图像，验证集上的40k图像以及test-dev上的20k图像进行了实验。我们评估了$IoU \in \lbrack 0.5:0.05:0.95 \rbrack$的mAP均值（COCO的度量标准，简单地表示为mAP @\[.5，.95\]）和mAP@0.5（PASCAL VOC度量标准）。

我们的系统进行了一些细微的修改来适配对这个数据集。我们在8 GPU实现上训练我们的模型，RPN（每个GPU 1个）有效的小批量大小为8，Fast R-CNN（每个GPU 2个）小批量大小16。 RPN步骤和Fast R-CNN步骤都训练了240k次迭代，学习率为0.003，然后用0.0003进行80k次迭代。我们修改学习率（从0.003开始，而不是0.001），因为小批量大小改变了。对于锚点，我们使用3个纵横比和4个尺度（增加$64^2$），主要为了处理该数据集上的小对象。此外，在我们的Fast R-CNN步骤中，负样本被定义为与检测框真值的最大IoU在$\lbrack 0, 0.5)$之间，而不是[^1] [^2]中使用的$\lbrack 0.1, 0.5)$。我们注意到，在SPPnet系统[^1]中，在$\lbrack 0.1, 0.5)$之间的负样本用于网络微调，但在$\lbrack 0, 0.5)$之间的负样本仍然在具有难负样本重训练的SVM步骤中被访问。但是Fast R-CNN系统[^2]放弃了SVM步骤，所以在$\lbrack 0, 0.1)$之间的负样本从未被访问过。包含这些$\lbrack 0, 0.1)$之间的样本在COCO数据集上提高了Fast R-CNN和Faster R-CNN系统的mAP@0.5（但PASCAL VOC的影响可以忽略不计）。

其余的实施细节与PASCAL VOC相同。特别是，我们继续使用300个提案和单尺度（$s = 600$）测试。 COCO数据集上的每个图像的测试时间仍然是大约200ms。

![Table 11](/assets/2017-10-12-faster-r-cnn/table11.png)

表11：MS COCO数据集上的目标检测结果（％）。模型为VGG-16。

在表11中，我们首先使用[^2]的实现评估了Fast R-CNN系统的结果。我们的Fast R-CNN基线在test-dev上mAP@0.5为39.3％，高于[2]中的结果。我们推测，这个差距的原因主要是由于负样本的定义以及小批量大小的变化。我们还注意到，mAP@\[.5，.95\]差不多。

接下来我们评估我们的Faster R-CNN系统。使用COCO训练集进行训练，Faster R-CNN在COCO test-dev上mAP@0.5为42.1％，mAP@\[.5，.95\]为21.5％的。与相同配置下的Fast R-CNN比较（表11），mAP@0.5提高2.8％，mAP@\[.5，.95\]提高2.2％。这表明RPN在较高的IoU阈值下表现出优异的定位精度。使用COCO train训练，Faster R-CNN在COCO test-dev上具mAP@0.5为42.7％，mAP@\[.5，.95\]为21.9％。图6显示了MS COCO test-dev的一些结果。

![Figure 6](/assets/2017-10-12-faster-r-cnn/figure6.png)

图6. 使用Faster R-CNN系统的MS COCO test-dev上的目标检测结果的选定示例。模型为VGG-16，训练数据为COCOtrainval（在test-dev上mAP@0.5为42.7％）。每个输出框与类别标签和$\lbrack 0, 1 \rbrack$的Softmax分数相关联。分数阈值为0.6。对于每个图像，一种颜色表示在该图像中的一种目标类别。

**ILSVRC和COCO 2015比赛中的Faster R-CNN：**我们已经证明，Faster R-CNN从更好的特征中获益更多，这得益于RPN完全通过神经网络学习区域提案的事实。即使将深度大大增加到100多层，这一结论仍然有效[^18]。只通过用101层的残差网络（ResNet-101）代替VGG-16 [^18]，Faster R-CNN系统将在COCO val上mAP从41.5％/ 21.2％（VGG-16）提高到了48.4％/ 27.2％（ResNet -101）。再加上其它对Faster R- CNN的改进，He等人[^18]获得了55.7％/ 34.9％的单模型结果，COCO test-dev的综合结果为59.0％/ 37.4％，在COCO 2015目标检测竞赛中排名第一。同样的系统[^18]也在ILSVRC 2015目标检测比赛中荣获第一名，超过第二名8.5％。 RPN也是ILSVRC 2015定位和COCO 2015分割比赛中第一名获奖作品的组成部分，细节分别在[^18]和[^15]中提供。

### 4.3从MS COCO到PASCAL VOC

大规模数据对于改进深层神经网络至关重要。接下来，我们调查MS COCO数据集如何帮助PASCAL VOC改善检测性能。

![Table 12](/assets/2017-10-12-faster-r-cnn/table12.png)

表12：PASCAL VOC 2007测试集和2012测试集中使用不同训练数据的Faster R-CNN检测mAP（％）。模型为VGG-16。 “COCO”表示COCO训练集用于训练。另见表6和表7。

作为一个简单的基线，我们直接评估了PASCAL VOC数据集中的COCO检测模型，而不对任何PASCAL VOC数据进行微调。这种评估是可能的，因为COCO的类别是PASCAL VOC类别的超集。在COCO中独有的类别在本实验中被忽略，Softmax层仅在20个类别和背景上执行。在PASCAL VOC 2007测试集上，此设置下的mAP为76.1％（表12）。尽管PASCAL VOC数据没有得到利用，但这一结果比VOC07 + 12（73.2％）得到了很好的提升。

然后我们对VOC数据集上的COCO检测模型进行微调。在该实验中，COCO模型代替ImageNet预先训练的模型（用于初始化网络权重），并且Faster R-CNN系统按3.2节所述进行微调。结果是PASCAL VOC 2007测试集上的mAP为78.8％。 COCO集合的额外数据将mAP增加5.6％。表6显示，在PASCAL VOC 2007上，针对COCO + VOC训练的模型对于每个类别都具有最佳的AP。PASCAL VOC 2012测试集（表12和表7）也有类似的改进。我们注意到，获得这些强大结果的测试速度仍然是每个图像约200ms。

## 5.总结

我们对高效和准确的区域提案的生成提出了区域提案提案网络（RPN）。通过与其后的检测网络共享卷积特征，区域提案的步骤几乎是无损耗的。我们的方法使一个一致的，基于深度学习的目标检测系统以近乎实时的帧率运行。学到的RPN也改善了区域提案的质量，进而改善整个目标检测的准确性。

表6：Fast R-CNN检测器和VGG16在PASCAL VOC 2007测试集的结果。对于RPN，Fast R-CNN训练时的提案框是2k个。RPN表示非共享特征的版本。

表7：Fast R-CNN检测器和VGG16在PASCAL VOC 2012测试集的结果。对于RPN，Fast R-CNN训练时的提案框是2k个。

图3：对最终的检测结果使用具有共享特征的RPN + FastR-CNN在PASCAL VOC 2007测试集上的例子。模型是VGG16，训练数据是07 + 12trainval。我们的方法检测的目标具有范围广泛的尺度和长宽比。每个输出框与一个类别标签和一个范围在[0,1]的softmax得分相关联。显示这些图像的得分阈值是0.6。取得这些结果的运行时间是每幅图像198ms，包括所有步骤。

**参考文献：**

[^1]: K. He, X. Zhang, S. Ren, and J. Sun, “Spatial pyramid pooling in deep convolutional networks for visual recognition,” in European Conference on Computer Vision (ECCV), 2014.
[^2]: R. Girshick, “Fast R-CNN,” in IEEE International Conference on Computer Vision (ICCV), 2015.
[^3]: K. Simonyan and A. Zisserman, “Very deep convolutional networks for large-scale image recognition,” in International Conference on Learning Representations (ICLR), 2015.
[^4]: J. R. Uijlings, K. E. van de Sande, T. Gevers, and A. W. Smeulders, “Selective search for object recognition,” International Journal of Computer Vision (IJCV), 2013.
[^5]: R. Girshick, J. Donahue, T. Darrell, and J. Malik, “Rich feature hierarchies for accurate object detection and semantic segmentation,” in IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2014.
[^6]: C. L. Zitnick and P. Dollár, “Edge boxes: Locating object proposals from edges,” in European Conference on Computer Vision (ECCV), 2014.
[^7]: J. Long, E. Shelhamer, and T. Darrell, “Fully convolutional networks for semantic segmentation,” in IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2015.
[^8]: P. F. Felzenszwalb, R. B. Girshick, D. McAllester, and D. Ramanan, “Object detection with discriminatively trained part-based models,” IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI), 2010.
[^9]: P. Sermanet, D. Eigen, X. Zhang, M. Mathieu, R. Fergus, and Y. LeCun, “Overfeat: Integrated recognition, localization and detection using convolutional networks,” in International Conference on Learning Representations (ICLR), 2014.
[^10]: S. Ren, K. He, R. Girshick, and J. Sun, “Faster R-CNN: Towards real-time object detection with region proposal networks,” in Neural Information Processing Systems (NIPS), 2015.
[^11]: M. Everingham, L. Van Gool, C. K. I. Williams, J. Winn, and A. Zisserman, “The PASCAL Visual Object Classes Challenge 2007 (VOC2007) Results,” 2007.
[^12]: T.-Y. Lin, M. Maire, S. Belongie, J. Hays, P. Perona, D. Ramanan, P. Dollár, and C. L. Zitnick, “Microsoft COCO: Common Objects in Context,” in European Conference on Computer Vision (ECCV), 2014.
[^13]: S. Song and J. Xiao, “Deep sliding shapes for amodal 3d object detection in rgb-d images,” arXiv:1511.02300, 2015.
[^14]: J. Zhu, X. Chen, and A. L. Yuille, “DeePM: A deep part-based model for object detection and semantic part localization,” arXiv:1511.07131, 2015.
[^15]: J. Dai, K. He, and J. Sun, “Instance-aware semantic segmentation via multi-task network cascades,” arXiv:1512.04412, 2015.
[^16]: J. Johnson, A. Karpathy, and L. Fei-Fei, “Densecap: Fully convolutional localization networks for dense captioning,” arXiv:1511.07571, 2015.
[^17]: D. Kislyuk, Y. Liu, D. Liu, E. Tzeng, and Y. Jing, “Human curation and convnets: Powering item-to-item recommendations on pinterest,” arXiv:1511.04003, 2015.
[^18]: K. He, X. Zhang, S. Ren, and J. Sun, “Deep residual learning for image recognition,” arXiv:1512.03385, 2015.
[^19]: J. Hosang, R. Benenson, and B. Schiele, “How good are detection proposals, really?” in British Machine Vision Conference (BMVC), 2014.
[^20]: J. Hosang, R. Benenson, P. Dollár, and B. Schiele, “What makes for effective detection proposals?” IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI), 2015.
[^21]: N. Chavali, H. Agrawal, A. Mahendru, and D. Batra, “Object-Proposal Evaluation Protocol is ’Gameable’,” arXiv: 1505.05836, 2015.
[^22]: J. Carreira and C. Sminchisescu, “CPMC: Automatic object segmentation using constrained parametric min-cuts,” IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI), 2012.
[^23]: P. Arbeláez, J. Pont-Tuset, J. T. Barron, F. Marques, and J. Malik, “Multiscale combinatorial grouping,” in IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2014.
[^24]: B. Alexe, T. Deselaers, and V. Ferrari, “Measuring the objectness of image windows,” IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI), 2012.
[^25]: C. Szegedy, A. Toshev, and D. Erhan, “Deep neural networks for object detection,” in Neural Information Processing Systems (NIPS), 2013.
[^26]: D. Erhan, C. Szegedy, A. Toshev, and D. Anguelov, “Scalable object detection using deep neural networks,” in IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2014.
[^27]: C. Szegedy, S. Reed, D. Erhan, and D. Anguelov, “Scalable, high-quality object detection,” arXiv:1412.1441 (v1), 2015.
[^28]: P. O. Pinheiro, R. Collobert, and P. Dollar, “Learning to segment object candidates,” in Neural Information Processing Systems (NIPS), 2015.
[^29]: J. Dai, K. He, and J. Sun, “Convolutional feature masking for joint object and stuff segmentation,” in IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2015.
[^30]: S. Ren, K. He, R. Girshick, X. Zhang, and J. Sun, “Object detection networks on convolutional feature maps,” arXiv:1504.06066, 2015.
[^31]: J. K. Chorowski, D. Bahdanau, D. Serdyuk, K. Cho, and Y. Bengio, “Attention-based models for speech recognition,” in Neural Information Processing Systems (NIPS), 2015.
[^32]: M. D. Zeiler and R. Fergus, “Visualizing and understanding convolutional neural networks,” in European Conference on Computer Vision (ECCV), 2014.
[^33]: V. Nair and G. E. Hinton, “Rectified linear units improve restricted boltzmann machines,” in International Conference on Machine Learning (ICML), 2010.
[^34]: C. Szegedy, W. Liu, Y. Jia, P. Sermanet, S. Reed, D. Anguelov, D. Erhan, and A. Rabinovich, “Going deeper with convolutions,” in IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2015.
[^35]: Y. LeCun, B. Boser, J. S. Denker, D. Henderson, R. E. Howard, W. Hubbard, and L. D. Jackel, “Backpropagation applied to handwritten zip code recognition,” Neural computation, 1989.
[^36]: O. Russakovsky, J. Deng, H. Su, J. Krause, S. Satheesh, S. Ma, Z. Huang, A. Karpathy, A. Khosla, M. Bernstein, A. C. Berg, and L. Fei-Fei, “ImageNet Large Scale Visual Recognition Challenge,” in International Journal of Computer Vision (IJCV), 2015.
[^37]: A. Krizhevsky, I. Sutskever, and G. Hinton, “Imagenet classification with deep convolutional neural networks,” in Neural Information Processing Systems (NIPS), 2012.
[^38]: Y. Jia, E. Shelhamer, J. Donahue, S. Karayev, J. Long, R. Girshick, S. Guadarrama, and T. Darrell, “Caffe: Convolutional architecture for fast feature embedding,” arXiv:1408.5093, 2014.
[^39]: K. Lenc and A. Vedaldi, “R-CNN minus R,” in British Machine Vision Conference (BMVC), 2015.
