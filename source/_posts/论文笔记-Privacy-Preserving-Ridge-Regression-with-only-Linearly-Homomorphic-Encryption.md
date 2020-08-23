---
title: >-
  论文笔记 Privacy-Preserving Ridge Regression with only Linearly-Homomorphic
  Encryption
date: 2020-08-10 19:49:57
categories: Papers
tags: [PPML, Linear Regression, HE]
---

*Irene Giacomelli, Somesh Jha, Marc Joye, C. David Page, and Kyonghwan Yoon*

[ACNS 2018](https://dblp.uni-trier.de/db/conf/acns/acns2018.html#GiacomelliJJPY18)

https://link.springer.com/chapter/10.1007/978-3-319-93387-0_13

<!--more-->

# 摘要

具有L2范数正则化的线性回归(即岭回归)是一种重要的统计技术，它使用线性函数来模拟某些解释值和结果值之间的关系。在许多应用中(例如，个性化医疗中的预测建模)，这些值表示由不愿意共享它们的几个不同方拥有的敏感数据。在这种情况下，训练线性回归模型变得具有挑战性，并且需要特定的密码解决方案。尼古拉恩科等人优雅地解决了这个问题。在标准普尔(奥克兰)2013年。他们提出了一个双服务器系统，使用线性同态加密(LHE)和姚的两方协议(乱码电路)。在这项工作中，我们提出了一种新的系统，它可以只使用LHE(即不使用姚的协议)来训练脊线性回归模型。这极大地提高了整体性能(在计算和通信方面)，因为姚的协议是以前解决方案中的主要瓶颈。在人工生成数据集和真实数据集上验证了该系统的有效性。



## 本文的贡献

本文提出了一种新的两服务台模型中的岭回归系统。

* 对于第一阶段，作者扩展了Nikolaenko等人使用的方法，涉及使用标记同态加密技术加密任意划分的数据集，以支持通过LHE方案加密的密文对之间的乘法。通过这种方式，作者展示了仅基于LHE的解决方案可以处理比水平分区情况更复杂的场景。

* 对于第二阶段，作者通过设计一个只利用底层加密方案的线性同态性质求解 $A \boldsymbol{w}=\boldsymbol{b}$ 的加性同态两方协议来避免姚的协议。这可以提高整体性能，特别是显著降低通信开销。作为一个亮点，如果我们水平划分(分成10个大小相等的部分)1000万个实例和20个特征的数据集，我们的隐私保护回归方法运行在2分钟以内，产生1.3MB的通信开销。在[23]中提出的系统需要超过50min和270MB的交换数据来执行类似的计算。

* 最后，作者注意到基于梯度下降的解决方案使用迭代算法，并且存在估计迭代次数 $t$ 的问题。或者 $t$ 被固定为确保找到模型的良好近似的高值，这为协议带来了更高的复杂度；或者 $t$ 是基于数据集自适应地选择的，这在隐私保护设置中可能是不可行的；或者 $t$ 是基于数据集自适应地选择的，这在隐私保护设置中可能是不可行的。本文求解 $A \boldsymbol{w}=\boldsymbol{b}$ 的解决方案不存在这个问题。



# Background

## 线性回归

见[LoPED](https://yuyingwai.cn/2020/08/09/论文笔记-Linear-Regression-on-Packed-Encrypted-Data-in-the-Two-Server-Model/)中的2.1小节。



## 密码工具

一般的加法同态算法只能计算一个密文和一个明文的乘积：
$$
\mathsf{cMult}(a,c) = c \odot c \cdots \odot c \quad (a \ \mathrm{times})
$$
而不能计算两个密文的乘积 $c \times c'$。因此作者在这里使用了一种称为 *labeled-homomorphic encryption*（labHE）的同态加密技术：

令 $(\mathsf{Gen, Enc, Dec})$ 为一个具有安全参数 $\kappa$ 和明文空间 $\mathcal{M}$ 的LHE方案。假设乘法运算在 $\mathcal{M}$ 中给出；例如 $(\mathcal{M}, +, \cdot)$ 是一个有限环。令 $F: \{0,1 \}^s \times \mathcal{L} \rightarrow \mathcal{M}$ 是一个有种子空间为 $\{0,1\}^s \ (s = \mathrm{poly}(\kappa))$ 以及标记空间 $\mathcal{L}$ 的伪随机函数。定义：

* $\mathsf{labGen}(\kappa)$：对输入 $\kappa$，它运行 $\mathsf{Gen}(\kappa)$ 然后输出 $(sk, pk)$。

* $\mathsf{localGen}(pk)$：对于每个用户 $i$ 以及公钥作为输入，它在 $\{0,1\}^s$ 中选取随机种子 $\sigma_i$ 然后计算 $pk_i = \mathsf{Enc}_{pk}(\underline{\sigma_i})$，其中 $\underline{\sigma_i}$ 是作为 $\mathcal{M}$ 中元素的 $\sigma_i$ 的编码。它输出 $(\sigma_i, pk_i)$。

* $\mathsf{labEnc}_{pk}(\sigma_i, m, \tau)$：对来自用户 $i$ 的一个带有标签 $\tau \in \mathcal{L}$ 的消息 $m \in \mathcal{M}$，它计算 $b = F(\sigma_i, \tau)$ 然后输出标记的密文 $\boldsymbol{c} = (a, c) \in \mathcal{M \times C}$，其中 $a = m - b$ 在 $\mathcal{M}$ 中而 $c = \mathsf{Enc}_{pk}(b)$。

* $\mathsf{labMult}(\boldsymbol{c, c'})$：输入两个标记的密文 $\boldsymbol{c} = (a, c)$ 和 $\boldsymbol{c'} = (a', c')$，它计算一个密文下的“乘积” $d = \mathsf{labMult}(\boldsymbol{c, c'})$ 为 $d = \mathsf{Enc}_{pk}(a \cdot a') \odot \mathsf{cMult}(a, c') \odot \mathsf{cMult}(a', c)$。容易验证 $\mathsf{Dec}_{sk}(d) = m \cdot m' - b \cdot b'$：
  $$
  \begin{align}
  m \cdot m' &= (a+b)(a'+b')\\
  &= a \cdot a' + a \cdot c' + a' \cdot c + b \cdot b'
  \end{align}
  $$

* $\mathsf{labDec}_{sk}(pk_i, pk_j, \tilde{c})$：输入 $\tilde{c}$，它首先从 $\mathsf{Dec}_{sk}(pk_i)$ 恢复 $\sigma_i$ 和 $\sigma_j$。然后，它对所有 $t = 1,...,n$ 计算 $b_t = F(\sigma_i, \tau_t)$ 和 $b'_t = F(\sigma_j, \tau_t')$。最后，它计算 $\tilde{b} = \sum^n_{t=1} b_t \cdot b_t'$ 和 $\tilde{m} = \mathsf{Dec}_{sk}(\tilde{c}) + \tilde{b}$。容易验证 $\tilde{m} = \sum^n_{t=1} m_t \cdot m_t'$。



# 系统概览

[LoPED](https://yuyingwai.cn/2020/08/09/论文笔记-Linear-Regression-on-Packed-Encrypted-Data-in-the-Two-Server-Model/)是基于本篇文章的，因此设定差不多。本文考虑这样的设置，其中训练数据集对于想要训练岭回归模型的实体不是明文可用的。相反，后者可以访问数据的加密副本，因此需要处理密钥的一方的帮助才能了解所需的模型。更准确地说，本文中的协议是为以下各方设计的：

* *数据拥有者*：有 $m$ 个数据拥有者 $DO_1,...,DO_m$；每个 $DO_i$ 都有一个私有的数据集 $\mathcal{D}_i$ ，并且愿意共享其加密后的版本。
* *机器学习引擎*（MLE）：这是希望对通过合并本地数据集 $\mathcal{D}_1,...,\mathcal{D}_m$ 而获得的数据集 $\mathcal{D}$ 运行线性回归算法，但只能访问它们的加密副本的一方。因此，MLE需要加密服务提供商的帮助。
* *加密服务提供商*（CSP）负责初始化系统中使用的加密方案，并与MLE交互以帮助其完成其任务(计算线性回归模型)。CSP管理加密密钥，并且是唯一能够解密的实体。

本文假设MLE和CSP没有串通，而且所有涉及的各方都是诚实但好奇的。此外，本文假设对于协议中涉及的每一对参与方，都存在一个私有的、经过身份验证的对等信道。特别是，任何两个参与者之间的通信都不能被窃听。

本文的目标是确保MLE获得模型，而MLE和CSP都不会学习到模型本身所揭示的关于私有数据集 $\mathcal{D}_i$ 的任何其他信息。即使在两个服务器(MLE或CSP)中的一个与一些数据所有者勾结的情况下，它们也不应该了解关于诚实的数据所有者持有的数据的额外信息。为了实现这一目标，本文设计了一个系统，它可以被视为由前面提到的 $m+2$ 方运行的多方协议，并由一系列步骤指定。该系统具有以下两阶段架构：

**阶段1（聚合本地数据集）：**CSP生成密钥对 $(sk,pk)$，保存 $sk$ 以及公开 $pk$；每个 $DO_i$ 用 $pk$ 加密自己的数据集 $\mathcal{D}_i$ 并发送给MLE。MLE使用接收到的密文和同态的特性来获得 $A$ 和 $\boldsymbol{b}$ 的密文。

**阶段2（计算模型）：**MLE盲化 $A$ 和 $\boldsymbol{b}$ 并发送给CSP；后者解密对盲化后的数据解密并运行给定的算法。该算法的输出（“盲化模型”）是一个向量 $\tilde{\boldsymbol{w}}$，CSP把它发送给MLE，后者去盲化后获得 $\boldsymbol{w}^*$。

* MLE选取一个随机可逆矩阵 $R \in \mathrm{GL}(d, \mathcal{M})$ 和随机向量 $\boldsymbol{r} \in \mathcal{M}$，然后使用线性同态加密的性质计算 $C' = \mathsf{Enc}_{pk}(AR)$ 和 $\boldsymbol{d'} = \mathsf{Enc}_{pk}(\boldsymbol{b} + A \boldsymbol{r})$。值 $C = AR$ 和 $\boldsymbol{d} = \boldsymbol{b} + A \boldsymbol{r}$ 就是“盲化数据”。
* CSP解密 $C'$ 和 $\boldsymbol{d'}$，然后计算 $\tilde{\boldsymbol{w}} = C^{-1} \boldsymbol{d}$。向量 $\tilde{\boldsymbol{w}}$ 为“盲化模型”，发送回给MLE。
* MLE计算 $\boldsymbol{w}^* = R \tilde{\boldsymbol{w}} - \boldsymbol{r}$，容易验证 $R \tilde{\boldsymbol{w}} - \boldsymbol{r} = R(AR)^{-1}(\boldsymbol{b} + A \boldsymbol{r}) - \boldsymbol{r} = A^{-1} \boldsymbol{b}$。



# 协议描述

## 阶段1：聚合数据集

两种情况，第一种情况为 $X$ 和 $\boldsymbol{y}$ 的数据集被水平划分为 $m$ 个数据集，即每个 $DO$ 拥有：
$$
\mathcal{D}_k = \{(\boldsymbol{x}_{n_{k-1}+1}, y_{n_{k-1}+1}),...,(\boldsymbol{x}_{n_k}, y_{n_k}) \} \tag{2}
$$
其中 $k = 1,...,m \ (0 = n_0 < n_1<\cdots<n_m=m)$。这种情况下就是每个 $DO_i$ 对其数据集 $A$ 和 $\boldsymbol{b}$ 的所有项加密发送给MLE，后者进行聚合：

![](http://images.yingwai.top/picgo/PPRRwoLHEp1.png)

第二种情况为每个 $DO$ 拥有 $X$ 和 $\boldsymbol{y}$ 中的某些元素：
$$
\mathcal{D}_{k}=\left\{X[i, j]=\boldsymbol{x}_{i}[j] \mid(i, j) \in D_{k}\right\} \cup\left\{\boldsymbol{y}[i]=y_{i} \mid(i, 0) \in D_{k}\right\} \tag{3}
$$
其中 $D_{k} \subseteq\{1, \ldots, n\} \times\{0,1, \ldots, d\}$。这种情况下，要计算矩阵 $A$ 和向量 $\boldsymbol{b}$ 就要把来自两个用户的数据乘在一起（例如一方拥有 $X$ 而另一方拥有 $\boldsymbol{y}$），此时可以用到前面的 $\mathsf{labMult}$：

![](http://images.yingwai.top/picgo/PPRRwoLHEp2.png)



## 阶段2：计算模型

在阶段1的最后，MLE知道了 $A$ 和 $\boldsymbol{b}$ 的密文。第3节中已经描述过大致的过程，即MLE先发送盲化后的数据给CSP，后者求解得到盲化后的模型后发送回给MLE，最后MLE去盲化得到模型 $\boldsymbol{w}^*$：

![](http://images.yingwai.top/picgo/PPRRwoLHEp3.png)



## 参数的选择

在 $\Pi_2$ 的最后一步使用了有理数重构从 $\mathbb{Z}_N$ 中计算的 $A \boldsymbol{w} = \boldsymbol{b}$ 的解中恢复 $\boldsymbol{w}^* \in \mathbb{Q}^d$。如果有理数 $t=r/s$ 且 $−R\leq r\leq R, 0<s \leq S$ 以及 $\gcd(s,N)=1$ 在 $\mathbb{Z}_N$ 中表示为 $t' = rs^{-1} \bmod N$，则拉格朗日-高斯算法在 $2RS <N$ 的情况下唯一地恢复 $r$ 和 $s$。

由于 $\boldsymbol{w}^* = A^{-1}\boldsymbol{b} = \frac{1}{\mathrm{det}(A)} \mathrm{adj}(A)\boldsymbol{b} \in \mathbb{Q}^d$，为了使得选择的 $N$ 满足前面提到的条件，需要限制 $\mathrm{det}(A)$ 和向量 $\mathrm{adj}(A)\boldsymbol{b}$ 的条目。令 $\alpha = \max \{\|A \|_{\infty},\| \boldsymbol{b} \|_{\infty} \}$，利用Hadamard不等式，有 $0 < \mathrm{det}(A) \leq \alpha^d$（$A$ 是正定矩阵）和 $\| \mathrm{adj}(A)\boldsymbol{b} \|_{\infty} \leq d(d-1)^{\frac{d-1}{2}} \alpha^d$。

对 $X$ 和 $\boldsymbol{y}$ 使用与第二节相同的假设（即 $X$ 和 $\boldsymbol{y}$ 的条目都是 $[-\delta, \delta]$ 中的实数，有最多 $\ell$ 位小数），有 $\alpha \leq 10^{2\ell}(n \delta^2 + \lambda)$。由此得出条件 $2RS < N$ 在以下情况时满足：
$$
2d(d-1)^{\frac{d-1}{2}} 10^{4\ell d}(n\delta^2 + \lambda)^{2d} < N \tag{4}
$$


## 复杂度

[SecureML](https://yuyingwai.cn/2020/06/17/论文笔记-SecureML-A-System-for-Scalable-Privacy-Preserving-Machine-Learning/)仅使用相加的秘密共享和海狸三元组来设计假设数据集的任意分区的系统。就通信复杂度而言，SecureML在任意划分的情况下比本文的解决方案执行得更好。然而，如果训练数据集是水平划分的，并且 $n \gg d$（例如 $n=Θ(d^{2.5})$）。例如，如果 $d=100, n=105$时，SecureML中的系统只有预处理阶段的200MB，而 $Π_{1,\mathrm{hor}}+Π_2$ 的总成本小于120MB。

