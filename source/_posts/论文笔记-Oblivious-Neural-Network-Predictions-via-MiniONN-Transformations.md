---
title: 论文笔记 Oblivious Neural Network Predictions via MiniONN Transformations
date: 2020-05-26 16:09:23
categories: Papers
tags: [密码学, PPML, MPC, Neural Network]
---

*Jian Liu, Mika Juuti, Yao Lu, N. Asokan*

CCS 2017

https://dl.acm.org/doi/10.1145/3133956.3134056

<!--more-->



# 摘要

托管在云服务中的机器学习模型越来越受欢迎，但存在隐私风险：向该服务发送预测请求的客户端需要披露潜在的敏感信息。在本文中，我们探讨了隐私保护预测问题：在每次预测之后，服务器对客户端的输入一无所知，客户端对模型一无所知。

我们提出了MiniONN，这是第一种将现有的神经网络转换为不经意的神经网络的方法，该网络以合理的效率支持隐私保护预测。与以前的工作不同，MiniONN不需要改变模型的训练方式。为此，我们为神经网络预测模型中常用的操作设计了健忘协议。我们证明了MiniONN在响应延迟和消息大小方面优于现有的工作。通过对标准数据集训练的几种典型神经网络模型的变换，证明了MiniONN的广泛适用性。

**关键词**：隐私；机器学习；神经网络预测；安全两方计算



## 贡献

* 提出了MiniONN，这是第一个**可以将任何普通神经网络模型转换为不经意神经网络**的技术，而不需要对训练阶段进行任何修改。
* **为神经网络预测中的常见操作设计了不经意的协议**。特别地，作者**使非线性函数(例如，Sigmoid和tanh)服从于作者的ONN变换**，而精确度损失可以忽略不计。
* 构建了**MiniONN的完整实现**，并通过使用它来**转换从几个标准数据集训练**的神经网络模型来展示其广泛的适用性。特别是，对于从MNIST数据集[38]训练的相同模型，MiniONN的性能**明显**好于以前的工作[28, 44]。
* 分析了**模型复杂性对转换后的ONN的预测精度和计算/通信开销的影响**。讨论了神经网络设计者如何在预测精度和开销之间选择合适的折衷方案。



# BACKGROUND AND PRELIMINARIES

![](http://images.yingwai.top/picgo/minionnt1.png)

<center>
    <i>表1 符号表示</i>
</center>



# MiniONN概述

在本节中，通过转换以下形式的神经网络来解释MiniONN的基本思想：
$$
\mathbf{z} := \mathbf{W'} \cdot f(\mathbf{W} \cdot \mathbf{x} + \mathbf{b}) + \mathbf{b'} \tag{4}
$$
其中$\mathbf{x} = \left[ \begin{matrix} x_1\\ x_2 \end{matrix} \right]$，$\mathbf{W} = \left[ \begin{matrix} w_{1,1} & w_{1,2}\\ w_{2,1} & w_{2,2} \end{matrix} \right]$，$\mathbf{b} = \left[ \begin{matrix} b_1\\ b_2 \end{matrix} \right]$，$\mathbf{W'} = \left[ \begin{matrix} w'_{1,1} & w'_{1,2}\\ w'_{2,1} & w'_{2,2} \end{matrix} \right]$ 以及 $\mathbf{b'} = \left[ \begin{matrix} b'_1\\ b'_2 \end{matrix} \right]$。

MiniONN的核心思想是让 $\mathcal{S}$ 和 $\mathcal{C}$ 加法共享神经网络每一层的输入和输出值。也就是说，在每一层的开始，$\mathcal{S}$ 和 $\mathcal{C}$ 将各自持有一份“份额”，使得份额的模加等于该神经网络的非不经意版本中对该层的输入。输出值将用作下一层的输入。

为此，让 $\mathcal{S}$ 和 $\mathcal{C}$ 首先进入预计算阶段(该阶段独立于 $\mathcal{C}$ 的输入$\mathbf{x}$)，在该阶段中，它们为权重矩阵的每一行(在本例中为 $\mathbf{W}$ 和 $\mathbf{W'}$)联合生成一组点积三元组 $\left \langle u,v,\mathbf{w} \cdot \mathbf{r} \right \rangle$。具体地说，对于 $\mathbf{w}$的每一行，$\mathcal{S}$ 和 $\mathcal{C}$，运行一个协议，该协议安全地实现理想功能 $\mathcal{F}$ 三元组(在图1中)，以生成点积三元组，从而：
$$
\begin{align}
u_1 + v_1 (\bmod N) &= w_{1,1} r_1 + w_{1,2} r_2,\\
u_2 + v_2 (\bmod N) &= w_{2,1} r_1 + w_{2,2} r_2,\\
u'_1 + v'_1 (\bmod N) &= w'_{1,1} r'_1 + w'_{1,2} r'_2,\\
u'_2 + v'_2 (\bmod N) &= w'_{2,1} r'_1 + w'_{2,2} r'_2.
\end{align}
$$
![](http://images.yingwai.top/picgo/minionnf1.png)

<center>
    <i>图1 理想的生成点积三元组的功能</i>
</center>

当 $\mathcal{C}$ 想要请求 $\mathcal{S}$ 计算向量 $\mathbf{x}=[x_1,x_2]$ 的预测时，对于每个 $x_i$，$\mathcal{C}$ 选择在预计算阶段中生成的三元组，并使用它的 $r_i$ 来盲化 $x_i$。
$$
\begin{align}
x_1^{\mathcal{C}} &:= r_1, x_1^{\mathcal{S}} := x_1 - r_1 (\bmod N),\\
x_2^{\mathcal{C}} &:= r_2, x_2^{\mathcal{S}} := x_2 - r_2 (\bmod N).
\end{align}
$$
然后 $\mathcal{C}$ 然后发送 $\mathbf{x}^{\mathcal{S}}$ 给 $\mathcal{S}$，$\mathcal{S}$ 计算
$$
\begin{align}
y_1^{\mathcal{S}} &:= w_{1,1} x_1^{\mathcal{S}} + w_{1,2} x_2^{\mathcal{S}} + b_1 + u_1 (\bmod N),\\
y_2^{\mathcal{S}} &:= w_{2,1} x_1^{\mathcal{S}} + w_{2,2} x_2^{\mathcal{S}} + b_2 + u_2 (\bmod N).
\end{align}
$$
同时，$\mathcal{C}$ 设：
$$
\begin{align}
y_1^{\mathcal{C}} &:= v_1 (\bmod N),\\
y_2^{\mathcal{C}} &:= v_2 (\bmod N).
\end{align}
$$
显然
$$
\begin{align}
y_1^{\mathcal{C}} + y_1^{\mathcal{S}} &= w_{1,1} x_1 + w_{1,2} x_2 + b_1 (\bmod N),\\
y_2^{\mathcal{C}} + y_2^{\mathcal{S}} &= w_{2,1} x_1 + w_{2,2} x_2 + b_2 (\bmod N).
\end{align}
$$
因此，在此交互结束时，$\mathcal{S}$ 和 $\mathcal{C}$ 相加地共享由层1中的线性变换产生的输出值 $\mathbf{y}$，而不需要 $\mathcal{S}$ 学习输入 $\mathbf{x}$，任何一方都不学习 $\mathbf{y}$。

对于激活/池化操作 $f()$，$\mathcal{S}$ 和 $\mathcal{C}$ 运行安全地实现图2中的理想功能的协议，该协议隐式地重构每个 $y_i := y^{\mathcal{C}}_i + y^{\mathcal{S}}_i (\bmod N)$ 并返回 $x^{\mathcal{S}}_i := f(y_i) - x^{\mathcal{C}}_i$ 给 $\mathcal{S}$，其中 $x^{\mathcal{C}}_i$ 是来自预计算阶段的先前共享的三元组的 $\mathcal{C}$ 分量，即 $x_1^{\mathcal{C}} := r'_1$ 和 $x_2^{\mathcal{C}} := r'_2$。

![](http://images.yingwai.top/picgo/minionnf2.png)

<center>
    <i>图2 理想的不经意激活/池化f()的功能</i>
</center>

最后一层的变换与第一层相同。也就是说，$\mathcal{S}$ 计算：
$$
\begin{align}
y_1^{\mathcal{S}} &:= w'_{1,1} x_1^{\mathcal{S}} + w'_{1,2} x_2^{\mathcal{S}} + b'_1 + u'_1 (\bmod N),\\
y_2^{\mathcal{S}} &:= w'_{2,1} x_1^{\mathcal{S}} + w'_{2,2} x_2^{\mathcal{S}} + b'_2 + u'_2 (\bmod N);
\end{align}
$$
$\mathcal{C}$ 设：
$$
\begin{align}
y_1^{\mathcal{C}} &:= v'_1 (\bmod N),\\
y_2^{\mathcal{C}} &:= v'_2 (\bmod N),
\end{align}
$$
最后，$\mathcal{S}$ 将 $[y_1^{\mathcal{S}}, y_2^{\mathcal{S}}]$ 返回给 $\mathcal{C}$，$\mathcal{C}$ 输出最终预测：
$$
\begin{align}
z_1 &:= y_1^{\mathcal{C}} + y_1^{\mathcal{S}},\\
z_2 &:= y_2^{\mathcal{C}} + y_2^{\mathcal{S}}.
\end{align}
$$
注意到MiniONN在 $\mathbb{Z}_N$ 中工作，而神经网络需要浮点计算。一种简单地解决办法是把神经网络中的值与一个固定的常数相乘，将小数部分放大到整数。一种类似的技术被用来减少神经网络预测中的存储器需求，而精确度损失可以忽略不计[42]。必须确保任何(中间)结果的绝对值不会超过 $\lfloor N/2 \rfloor$。