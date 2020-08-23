---
title: 论文笔记 Linear-Regression on Packed Encrypted Data in the Two-Server Model
date: 2020-08-09 17:03:33
categories: Papers
tags: [PPML, Linear Regression, HE]
---

*Adi Akavia, Hayim Shaul, Mor Weiss, Zohar Yakhini*

[WAHC'19: Proceedings of the 7th ACM Workshop on Encrypted Computing & Applied Homomorphic Cryptography](https://dl.acm.org/doi/proceedings/10.1145/3338469)

https://dl.acm.org/doi/abs/10.1145/3338469.3358942

<!--more-->

![](http://images.yingwai.top/picgo/LoPED.png)

# 摘要

从包含大量独立样本的联合训练数据中开发机器学习模型是一项重要的任务，可以显著提高学习模型的潜在适用性和预测能力。由于单个用户(如医院或单个实验室)通常以高置信度收集不支持准确学习的数据集，因此希望在不损害数据隐私的情况下组合来自多个用户的数据。在这篇文章中，我们开发了一种隐私保护解决方案，用于从多方(“数据所有者”)共同贡献的数据中学习线性回归模型。我们的协议是基于Giacomelli等人的协议。(ACNS 2018)，利用两个非合谋服务器和线性同态加密(LHE)学习正则化线性回归模型。我们的方法使用不同的LHE方案，使我们能够显著减少同态操作的数量和运行时间，以及总的运行时间复杂度。我们协议的另一个优点是潜在的LHE方案基于与Giacomelli等人不同的(和后量子安全的)安全假设。我们的方法利用中国剩余定理和单指令多数据表示法来获得改进的性能。对于1000x40线性回归任务，对于同态操作，我们可以在总共3秒内学习到一个模型，而文献中报道的时间超过了100秒。我们的方法还可以扩展到更大的功能空间：我们实现了一个可以处理1000x100线性回归任务的系统，在数据所有者进行更重要的离线预处理之后，投入了数分钟的服务器计算时间。我们打算将我们的协议和实现整合到一个全面的系统中，该系统可以在更大范围内处理安全的联合学习。



## 本文贡献

* 线性-压缩加密数据的回归(LoPED)。提出了一种新的PPRR协议，称为“分组加密数据线性回归协议(LoPED)。
* 同态计算时间加速：分析结果。与[6]相比，LoPED在同态运算的数量上提供了高达 $\Theta(d^2)$ 的加速比，其中 $d$ 是特征的数量。更准确地说，加速比是 $\Theta(\mathsf{sl})$，其中 $\mathsf{sl} \leq d^2$ 是每个SIMD密文中封装的时隙数目。
* 同态计算时间加速：实验结果。我们实现了我们的协议并进行了大量的实验，证明了同态运算(又名LHE-COMP)运行时的大幅加速；参见表1。
* 安全性。LoPED基于后量子安全假设，与[6]中使用的基于Paillier的实现形成对比；这是我们系统的一个额外优势。



# PRELIMINARIES

## 岭回归

线性回归的损失函数：
$$
f(w) = \sum^m_{i = 1} \left(y_i - x^T_i w \right)^2
$$
于是 $\boldsymbol{w}$ 的最优解为
$$
\hat{\boldsymbol{w}}^{*}=\underset{\hat{\boldsymbol{w}}}{\arg \min }(\boldsymbol{y}-\mathbf{X} \hat{\boldsymbol{w}})^{\mathrm{T}}(\boldsymbol{y}-\mathbf{X} \hat{\boldsymbol{w}})
$$
对 $\hat{\boldsymbol{w}}$ 求导并令导数为零，当 $\mathbf{X}^{\mathrm{T}} \mathbf{X}$ 为满秩或正定矩阵时可得
$$
\hat{\boldsymbol{w}}^{*}=\left(\mathbf{X}^{\mathrm{T}} \mathbf{X}\right)^{-1} \mathbf{X}^{\mathrm{T}} \boldsymbol{y}
$$
然而现实中 $\mathbf{X}^{\mathrm{T}} \mathbf{X}$ 往往不是满秩矩阵。例如在许多任务中会遇到大量的变量，其数目甚至超过样例数，导致 $\mathbf{X}$ 的列数多于行数，$\mathbf{X}^{\mathrm{T}} \mathbf{X}$ 显然不满秩。从上式可以看出在计算回归系数的时候，我们需要计算矩阵 $\mathbf{X}^{\mathrm{T}} \mathbf{X}$ 的逆，但是如果该矩阵是个奇异矩阵，则无法对其进行求解。

因此如果出现上面的情况，我们需要对最初的标准线性回归做一定的变化使原先无法求逆的矩阵变得非奇异，使得问题可以稳定求解。我们可以通过缩减的方式来处理这些问题，例如岭回归。

岭回归是在标准线性回归的损失函数基础上添加了一个惩罚项 $\lambda \sum^n_{i=1} w^2_i$，称为L2正则化。

这个时候损失函数的形式为
$$
f(w) = \sum^m_{i = 1} \left(y_i - x^T_i w \right)^2 + \lambda \sum^n_{i=1} w^2_i
$$
将岭回归系数用矩阵的形式表示：
$$
\hat{w}=\left(X^{T} X+\lambda I\right)^{-1} X^{T} y
$$
可以看到，就是通过将 $\mathbf{X}^{\mathrm{T}} \mathbf{X}$ 加上一个单位矩阵似的矩阵变成非奇异矩阵并可以进行求逆运算。



## 线性同态加密(LHE)

LHE是一种公钥加密方案，它允许人们在不知道秘密解密密钥的情况下，在加密的“幕后”执行线性运算。我们假设在密钥生成过程中，可以通过指定 $N \in \mathbb{N}$ 来选择明文空间，从而执行模 $N$ 的同态运算。这是通过将明文模 $N$ 显式地合并到方案的排列中来实现的。

线性同态加密(LHE)方案 $\mathcal{E}=(\mathsf{KG}, \mathsf{Enc},\mathsf{Dec},\mathsf{Eval}) $ 由四个算法组成，其中 $\mathsf{KG}, \mathsf{Enc}$ 和 $\mathsf{Eval}$ 是PPT算法，$\mathsf{Dec}$ 是(确定性)多项式时间。算法具有以下语法：

* $\mathsf{KG}(1^\sigma, N)$ 将安全参数 $σ$ 和 $N \in \mathbb{N}$ 作为输入。它输出一对公开和秘密加密密钥 $(\mathsf{pk}, \mathsf{sk})$。我们假设  $\mathsf{pk}$ 在其描述中包含 $N$，而不失一般性。
* $\mathsf{Enc}(\mathsf{pk}, \mathsf{msg})$ 将公钥 $\mathsf{pk}$ 和消息 $\mathsf{msg} \in \mathbb{Z}_N$ 作为输入，输出密文 $\mathsf{c}$。
* $\mathsf{Dec}(\mathsf{sk},\mathsf{c})$ 将秘密解密密钥 $\mathsf{sk}$ 和密文 $\mathsf{c}$ 作为输入，并输出明文消息 $\mathsf{msg'}$。
* $\mathsf{Eval}(\mathsf{pk}, C, \mathsf{c}_1,..., \mathsf{c}_k)$ 将公钥 $\mathsf{pk}$、电路 $C: \mathbb{Z}^k_N \rightarrow \mathbb{Z}^l_N$，其中 $l, k \in \mathbb{N}$ 和 $k$ 个密文 $\mathsf{c}_1,...,\mathsf{c}_k$ 作为输入，并输出 $l$ 个密文 $(\mathsf{c}'_1,...,\mathsf{c}'_l)$。



# 问题陈述

## 在联合数据上的岭回归

由于每个用户所拥有的数据样本是有限的，往往训练不出来比较好的模型。于是很自然地可以想到与其他用户一起来训练一个模型，每个用户都将自己的数据提供出来，此时样本的数量就足以训练出一个好的模型。这里不同用户的数据样本都是具有同样特征的。

但这样就会有隐私泄露的问题，用户都不想别人知道自己的数据，于是就需要引入隐私保护。这里假设协议的参与者都是城市且好奇的。



## Privacy-Preserving Ridge Regression with only Linearly-Homomorphic Encryption 中的方案

在这篇文章之前最好的双服务器模型PPRR方案是 [*Privacy-Preserving Ridge Regression with only Linearly-Homomorphic Encryption*](https://yuyingwai.cn/2020/08/10/论文笔记-Privacy-Preserving-Ridge-Regression-with-only-Linearly-Homomorphic-Encryption/) 中的方案，如图1所示，

![](http://images.yingwai.top/picgo/LoPEDf1.png)

该协议对参数 $n, d, \ell, \lambda$ 进行操作，并选择明文环 $\mathbb{Z}_N$ 满足：
$$
N > 2d(d - 1)^{\frac{d-1}{2}}10^{4\ell d}(n^2 + \lambda)^{2d} \tag{1}
$$
在初始化阶段，$\mathcal{S}_2$ 生成LHE密钥 $(\mathsf{pk}, \mathsf{sk})$，然后发布公钥 $\mathsf{pk}$。然后该协议对在多个数据所有者之间水平划分的输入 $(X|\vec{y}) \in \mathbb{R}^{n \times (d+1)}$ 训练岭回归模型，如下所示。

1. 首先每个数据拥有者计算其对于 $d \times d$ 矩阵 $A = X^T X + \lambda I$ 的份额，将其缩放为嵌入 $\mathbb{Z}_N$ 的整数值，用LHE方案加密该份额，并将密文发送给 $\mathcal{S}_1$。用类似的方式计算 $\vec{b} = X^T \cdot \vec{y}$。
2. 其次 $\mathcal{S}_1$ 将所有的份额结合起来获得 $A$ 的逐项输入加密 $\mathbf{A}$，假设其在 $\mathbb{Z}^{d \times d}_N$ 中是可逆的。然后 $\mathcal{S}_1$ 使用同态性质对 $A$ 进行盲化，具体为 $\mathcal{S}_1$ 使用LHE计算 $A \cdot R$ 的加密 $\mathbf{C}$，其中 $R \in \mathsf{GL}(d, \mathbb{Z}_N)$ 是一个随机的可逆矩阵。用类似的方法盲化 $\vec{b}$，得到 $\vec{\mathbf{v}}$ 为加密后的 $\vec{v} = \vec{b} + A \cdot \vec{r}$，其中 $r \in \mathbb{Z}^d_N$ 是一致随机的。然后 $\mathcal{S}_1$ 将 $\mathbf{C}$ 和 $\vec{\mathbf{v}}$ 发送给 $\mathcal{S}_2$。
3. $\mathcal{S}_2$ 使用密钥解密，求解线性系统 $C \cdot \vec{w}^* = \vec{v} \bmod N$ 来获得盲化后的模型 $\vec{w}^*$，然后发送给 $\mathcal{S}_1$。
4. 最后 $\mathcal{S}_1$ 去除盲化获得模型 $\vec{w}' = R \cdot \vec{w}^* - \vec{r} \in \mathbb{Z}^d_N$。对 $\vec{w}'$ 使用有理数重构获得输出模型 $\vec{w} \in \mathbb{Q}^d$，然后 $\mathcal{S}_1$ 对各方进行广播。



# 本文的隐私保护岭回归协议

本文的模型与 *Privacy-Preserving Ridge Regression with only Linearly-Homomorphic Encryption* 中的模型一样，主要包括两个主体：数据拥有者 $DO$ 和服务器 $\mathcal{S}$。其中数据拥有者的数量是已知的；服务器有两个，$\mathcal{S}_1$ 负责将 $DO$ 的数据聚合，$\mathcal{S}_2$ 负责更新权重。分为了三个阶段：初始化、上传输入和训练阶段。

![](http://images.yingwai.top/picgo/LoPEDf2.png)



## 初始化

初始化（图2）包括 $\mathcal{S}_2$ 生成LHE方案密钥，和发布公钥。

由于 *Privacy-Preserving Ridge Regression with only Linearly-Homomorphic Encryption* 中的明文模数太大，而HElib和SEAL分别只支持62位和60位的明文模数，因此本文使用了中国剩余定理(CRT)来将明文空间缩小，使用 $\mathbb{Z}_{p_1},...,\mathbb{Z}_{p_t}$ 中的消息列表来表示 $\mathbb{Z}_N$ 中的消息，使得 $N = \prod^t_{i=1} p_i$，每个模 $p_i$ 需要独立的加密密钥也由 $\mathcal{S}_2$ 来生成。（前面提到的另一个方案中的操作是直接在 $\mathbb{Z}_N$ 中进行的，因此 $\mathcal{S}_2$ 仅生成一对密钥 $(\mathsf{pk}, \mathsf{sk})$。）



## 上传输入

在图一中我们假设输入在数据拥有者之间水平分割，类似横向联邦学习（样本的联合）。

令 $X \in \mathbb{R}^{n \times d}, \vec{y} \in \mathbb{R}^{n \times 1}$ 分别表示数据矩阵和响应向量，于是存在 $[n]$ 的分割 $I_1,...,I_m$ 使得 $DO_j$ 拥有 $X^j = X_{I_j} = (X_k : k \in I_j)$（$X_k$ 表示 $X$ 的第 $k$ 行）和 $\vec{y}^j = \vec{y}_{I_j} = (y_k)_{k \in I_j}$。这些输入按比例缩放并嵌入到足够大的 $N$ 的 $\mathbb{Z}_N$ 中（有关 $N$ 的要求请参见公式1）。对每个 $1 \leq j \leq m$，令 $A^j = \sum_{k \in I_j} \left(X^j_k \right)^T \cdot X^j_k \in \mathbb{Z}^{d \times d}_N$，和 $\vec{b}_j = \sum_{k \in I_j} X^j_k \cdot \vec{y}^j_k \in \mathbb{Z}^d_N$。每个 $DO_j$ 在本地从他的输入计算 $A^j, \vec{b}^j$，计算
$$
A^{j,p_i} = \left(A^j \bmod p_i \right) \quad \mathrm{and} \quad \vec{b}^{j, p_i} = \left(\vec{b}^j \bmod p_i \right)
$$
然后发送加密后的 $A^{j, p_i}, \vec{b}^{j, p_i}, i \in [t]$ 给 $\mathcal{S}_1$。



## 训练

![](http://images.yingwai.top/picgo/LoPEDf3.png)

在这个阶段，$\mathcal{S}_1$ 结合以及盲化来自所有数据拥有者的数据，$\mathcal{S}_2$ 对盲化后的数据执行学习。

在使用下面的方式将来自所有数据拥有者的数据组合后可以获得 $A = X^T X + \lambda I$（其中 $\lambda$ 是图1中的正则化参数，$I$ 是单位矩阵）和 $\vec{b} = X^T \vec{y}$：在前一个阶段 $\mathcal{S}_1$ 获得了 $A^1,...,A^m$ 和 $\vec{b}^1,...,\vec{b}^m$，于是有
$$
A = \sum^{m}_{j=1} A_j + \lambda I \quad \mathrm{and} \quad b = \sum^m_{j=1} b_j
$$
因为本文使用CRT表示，需要计算 $A^{p_i} = A \bmod p_i$ 和 $\vec{b}^{p_i} = \vec{b} \bmod p_i$ 对每个 $i \in [t]$，而不是 $A, \vec{b}$。为此，$\mathcal{S}_1$ 使用LHE方案的线性同态从 $A^{j, p_i}, \vec{b}^{j, p_i}$ 计算 $A^{p_i}, \vec{b}^{p_i}$（参见图3中的步骤1）。

本文的盲化方法类似3.2小节中的描述，但需要进行一些修改去兼容CRT表示。具体地说，$\mathcal{S}_1$ 选择一个随机的可逆矩阵 $R \in \mathbb{Z}^{d \times d}_{N}$ 和随机的 $\vec{r} \in \mathbb{Z}^d_N$，然后计算 $C = A \cdot R$ 和 $\vec{v} = \vec{b} + A \cdot \vec{r}$。另外，由于对每个 $i \in [t]$，对 $A^{p_i}, \vec{b}^{p_i}$ 的操作是在 $\mathbb{Z}_{p_i}$ 内的，所以使用 $R^{p_i} = R \bmod p_i$ 和 $\vec{r}^{p_i} = \vec{r} \bmod p_i$ 来执行盲化，于是盲化后的数据矩阵和响应向量分别为 $C^{p_i} = A^{p_i} \cdot R^{p_i} \bmod p_i$ 和 $\vec{v}^{p_i} = \vec{b}^{p_i} + A^{p_i} \cdot \vec{r}^{p_i} \bmod p_i$（见图3中的步骤3a）。$C^{p_i}, \vec{v}^{p_i}$ 是由 $\mathcal{S}_1$ 使用LHE方案的线性同态来计算的，然后发送给 $\mathcal{S}_2$。$\mathcal{S}_2$ 解密这些密文来获取 $C^{p_i}, \vec{v}^{p_i}, i \in [t]$，然后使用CRT重构来恢复 $C, \vec{v}$。

然后 $\mathcal{S}_2$ 计算 $C^{-1} = \mathsf{adj}(C) / \mathsf{det}(C)$（其中 $\mathsf{adj}(C), \mathsf{det}(c)$ 分别表示 $C$ 的伴随矩阵和行列式）来求解线性系统 $C \cdot \vec{w} = \vec{v}$，获得盲化后的模型 $\vec{w}^*$ 并发送给 $\mathcal{S}_1$。于是 $\mathcal{S}_1$ 可以计算 $\vec{w} = R \cdot \vec{w}^* - \vec{r}$ 来去盲化。这给出了 $\mathbb{Z}^d_N$ 中的一个模型，$\mathbb{Q}^d$ 中的相应模型可以使用有理重构从该模型中重构出来(参见第3.2节中的讨论)。



# 将SIMD集成到LoPED

## SIMD背景知识

支持单指令多数据(SIMD)的加密方案和实现包括：Brakerski-Gentry-Vaikuntanathan和Fan-Vercauteren的基于RLWE的方案(通常称为BGV/FV)及其在HElib和SEAL库中的实现。这些方案允许在单个密文中将多个消息“打包”在不同的“槽”中。本文将打包参数，即打包在一个密文中的消息数目记为 $\mathsf{sl}$，并且密文中的不同时隙记为 $\mathbf{c} = (\mathbf{c}(1),...,\mathbf{c}(\mathsf{sl}))$。对打包密文的操作以SIMD（单指令多数据）方式完成。例如，由 $c=a \odot b$ 表示的SIMD乘法定义为 $\mathbf{c}(i) = \mathbf{a}(i) \odot \mathbf{b}(i)$，其中 $i = 1,...,\mathsf{sl}$。

重要的是要注意，在整个计算过程中幼稚地使用SIMD可能不会提高效率，因为涉及不同时隙上的计算的操作(例如，矩阵乘法，见下文)会招致高开销，这实际上可能会损害效率。



## 将SIMD集成到LoPED

首先看看LoPED中同态计算的操作：

1. 向量相加（图3步骤1）
2. 矩阵相加（图3步骤1）
3. 矩阵相乘（图3步骤3a）
4. 矩阵向量乘法（图3步骤3a）

在这些操作中最昂贵的是第三个，因此本文将其与作者引入的矢量的新表示法以及作者为有效执行上述其他操作而设计的新协议相结合。



### 矩阵编码和逐个矩阵乘法

如上所述，本文使用 *Secure Out-sourced Matrix Computation and Application to Neural Networks* 的方案来使用SIMD进行高效的矩阵乘法，因此使用它们的矩阵编码。上面的方案区分从左侧相乘的矩阵和从右侧相乘的矩阵，并对每个矩阵使用不同的编码。设 $L$ 和 $R$ 是两个 $d\times d$ 矩阵，我们希望计算 $A=L \cdot R$，现在描述 $L$ 和 $R$ 的编码。



#### Type-L 和 Type-R 编码以及 Type-N 表示

本文将矩阵 $L$（乘积中的左矩阵）的编码表示为 Type-L 编码，其由 $L$ 的 $d$ 个旋转 $L_1,...,L_d$ 组成，其中 $L_i$ 为一个 $L$ 旋转其行后计算得到的 $d \times d$ 矩阵；类似地，将矩阵 $R$（乘积中的右矩阵）的编码表示为 Type-R 编码，也由 $d$ 个旋转组成，每个 $R_i$ 是 $R$ 旋转其列后得到的 $d \times d$ 矩阵（例子见图4）。然后用 Type-N 表示不作任何处理的原始矩阵的排列。

对于以上两种编码，个人的理解是（拿 $L$ 来说）：

* $L_1$：第 $i$ 行的元素往前移动 $i-1$ 个位置（前 $i-1$ 个移动到后面）；
* $L_2$：第 $i$ 行的元素往前移动 $i$ 个位置；
* 以此类推：$L_j$：第 $i$ 行的元素往前移动 $i+j-2$ 个位置。

$R$ 也是类似的。

作者还说到可以将这 $d$ 个矩阵打包到较少数量的密文中，从而减少开销。具体地说，原来的矩阵表示需要 $d^2$ 密文，而本文的表示需要 $d \cdot \lceil \frac{d^2}{\mathsf{sl}} \rceil$ 密文(当 $\mathsf{sl} > d$ 时，其优于 $d^2$)。



#### 逐个矩阵乘法

![](http://images.yingwai.top/picgo/LoPEDf4.png)

矩阵乘法的例子见上图，计算 $A = L \cdot R$ 可以分解成计算 $A = \sum^d_{i=1} A_i$，其中 $A_i = L_i \odot R_i$ 为两个矩阵的对应元素相乘。

在我们的协议中，我们将加密矩阵与公共矩阵相乘(参见图3中的步骤3a)。这是使用基础LHE方案的 $\mathsf{Eval}$ 算法完成的，用 $\mathsf{Eval}(\mathsf{pk},\mathsf{MatMult}, \mathbf{L}, R)$ 表示在输入电路 $C_{\mathsf{MatMult}, R}$ 和密文 $\mathbf{L}$ 上运行 $\mathsf{Eval}$，其中 $C_{\mathsf{MatMult}, R}$ 输入一个矩阵 $L$ 的Type-L编码，计算输出得到其与一个公共矩阵 $R$ 的乘积 $A = L \cdot R$。



#### 通过序列化提高效率

本文通过将多个矩阵条目打包到单个密文中来提高效率（并克服了保存每个矩阵的多个旋转副本所带来的开销）。这是通过首先将 $d \times d$ 矩阵表示(“序列化”)为长度为 $d^2$ 的向量来实现的。

![](http://images.yingwai.top/picgo/LoPEDf5.png)

就是将矩阵从左到右、从上到下，按列的顺序打包到密文中。也就是说，$A$ 的元素 $A_{c,l}$ 出现在串行化后的 $A$ 的第 $((c−1)d+l)$ 个位置。然后，可以将串行化矩阵（如果需要则用零填充）加密成密文（打包到时隙中）。



### 向量表示

本文描述了两种不同的矢量编码，每种编码都用于优化不同操作的复杂度。第一个编码称之为Type-M，允许高效的矩阵-向量乘法。第二种编码称之为Type-A，允许高效的向量相加。

![](http://images.yingwai.top/picgo/LoPEDf6.png)

在计算矩阵-向量乘法时，首先将向量映射为一个 $d \times d$ 矩阵（Type-M编码：其中第一列为向量，其余列用0填充），然后再将该矩阵编码为Type-R矩阵，进行矩阵乘法。而Type-A编码则是对Type-M矩阵的序列化。

我们注意到，虽然可以对类型M编码执行加法，但它的效率低于将两个类型A编码相加。例如当 $\mathsf{sl}=d$ 时，将两个Type-M向量相加需要 $d$ 次运算，但是添加两个Type-A编码只需要一次操作。

![](http://images.yingwai.top/picgo/LoPEDf7.png)

Type-M编码的向量与Type-L编码的矩阵相乘得到的结果序列化后，直接把多余的0去掉就可以变成与Type-A编码的向量格式一致，从而可以直接把两个向量相加（例子见图7）。

