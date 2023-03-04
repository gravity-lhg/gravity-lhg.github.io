---
layout: post
title: 'Awesome Backnone Plan'
data: 2023-03-04
author: 刘浩光
cover: 'assets/images/AwesomeBackbone.png'
tags: 深度学习
---

# Awesome Backbone Plan

> 针对现有 Backbone 的学习笔记

##### 针对网络结构细微调整和训练策略的研究  [Bag of Tricks for Image Classification with Convolutional Neural Networks](https://arxiv.org/pdf/1812.01187.pdf)

***Efficient Training***

- *large-batch training*    对于凸优化问题，随着 batch 的增加其收敛速率会降低，神经网络也会存在这样的问题。
  - *linear scaling learning rate*    增加 batch 时不会改变期望，但方差会减小。换句话说，大的 batch 会减少梯度中的噪声，所以增加学习率沿着梯度方向相反的方向可以加速网络收敛。假设 ResNet-50 的训练以 0.1 *learning rate*，256 batch size 为基准，当我们改变训练的 batch 大小时（假设改为b），那么应该调整学习率为 *lr=0.1 x b/256* 
  - *learning rate warmup*    初始化时整个网络的参数是随机的，远离最终优化后的结果。太大的学习率导致训练初期不稳定，采用学习率升温的策略，即从一个很小的学习率逐渐上升到初始学习率，可稳定初期的训练，便于得到最优的性能。一般用线性升温的学习率策略，假设使用前 m 次迭代，初始的学习率为 a，则每次迭代的学习率为 *i · a / m*
  - *zero $\gamma$*    对每一个 resnet 块的最后一个 BN 层的 weight 进行初始化为 0 的操作， 这样所有的块都返回他们的输入，模拟层数较少且更容易训练的初始阶段
  - *No bias decay*    **权重衰减** 应用于所有可学习参数，weight 和 bias。其就像在所有参数上应用 L2 正则化。仅对 weight 使用正则化可避免过拟合。==（没太理解）==
- *low-precision training*    将模型精度从 FP32 转为 FP16 可将训练速度提升2～3倍，精度不会大幅度降低且有时会有更高的结果。

***Model Tweaks***

![ResNet architecture and some tweaks](/assets/images/resnet_tweak.png)

- *ResNet-B*    其认为在 block 中第一个 1x1 的卷积使用步长为 2 会丢失觉很大一部分信息，将其移致 3x3 的卷积更为合理。
- *ResNet-C*    因为卷积的计算复成本和其核边长是二次相关的，其将最前面的 7x7 卷积用 3 个 3x3 的卷积替换，以降低其计算成本。具体细节是将通道数变化改为 3 >> 32，32 >> 32，32  >> 64，对应的步长分别是 2，2，1。
- *ResNet-D*    论文中提出的，灵感来源于 *ResNet-B* 结构。其发现下采样路径中使用的卷积大小为 1x1，但其步长为 2，这样的卷积会都是一部分信息。在 1x1 卷积的前面加上一个步长为 2 的全局平均池化（GAP）并将 1x1 卷积步长恢复到 1，这样可以在无参数增加、仅增加小部分计算量的前提下提升网络的精度。 

以上消融实验结果如下所示。（采用 *Efficient Training* 中所有tricks，需要注意的是**三个 tweak 是逐级叠加的优化**）

![resnet_tweak_ablation](/assets/images/resnet_tweak_ablation.png)

***Training Refinements***

