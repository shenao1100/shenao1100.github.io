---
title: 机器学习专有名词解释
description: 以Image Super-Resolution Using Deep Convolutional Networks为例尝试解释专有名词
author: ShenNya
date: 2024-11-09 17:32:00 +0800
categories: [Machine Learing]
tags: [Machine Learing, 神经网络]
#pin: true
math: true
mermaid: true
---

## 写在前面

> 此笔记尚未完成
{: .prompt-info }

本人纯小白，仅仅是对ML感兴趣，甚至因病没有高数基础，本文的内容既不严谨也不能保证完全准确，内容请自行评判

文章地址(arXiv)

[Image Super-Resolution Using Deep Convolutional Networks](https://arxiv.org/abs/1501.00092)

[PDF链接](https://arxiv.org/pdf/1501.00092)


# 网络退化

文中提到了名为`网络退化`的问题，传统的神经网络理论认为增加层数可以提升神经网络的表达能力，更深的网络可以逼近更复杂的函数，然而实际训练时，超过了一定层数模型的表达效果反而更差

## 恒等映射

恒等映射是一个数学概念: 一个从某个集合到自身的映射，它将集合中的每一个元素映射到它自己

符号表达法如下：
```
I:X→X其中I(x)=x,∀x∈X
```
- I: 通常指代映射函数
- I(x)=x: 表示对于集合X中任意元素x，经过映射函数I后输出的元素是x本身

在机器学习或其他数学应用中，恒等映射常用于表示一种没有变化的变换，在文中表示`跳跃连接（skip connections）`和`残差连接（residual connections）`

# 残差学习

为了解决这个问题，文中作者认为在增加层数后至少让增加的层数能够学习[恒等映射](#恒等映射)，也就是即使带不来好处至少不要劣化模型

于是提出了残差学习的概念

残差学习的核心思想是，直接让网络学习`残差（Residual）`而不是`期望映射 (Expected Mapping)`
## 梯度

指一个函数在某一点的变化率或方向，通常用于优化算法中，帮助我们找到一个函数最小值或最大值的方向

通过计算梯度，我们可以确定如何调整参数，以减少误差，从而让模型的预测更加准确

在训练神经网络时，`反向传播算法（Backpropagation）`通过计算梯度来调整每一层的参数，使得预测误差逐步减小。梯度越大，参数更新的步伐就越大；梯度越小，参数更新就越小

## 缓解梯度: 梯度消失

在深度神经网络的反向传播过程中，梯度在逐层传播时会逐渐减小，导致靠近输入层的参数更新非常缓慢，甚至几乎不更新。这会使得模型难以学习到有效的特征

## 缓解梯度: 梯度爆炸

与梯度消失相反，在某些情况下，梯度值会随着传播逐渐增大，导致靠近输入层的参数更新剧烈，模型变得不稳定，训练难以收敛
