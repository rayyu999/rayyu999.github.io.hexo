---
title: >-
  论文笔记 ABY - A Framework for Efficient Mixed-Protocol Secure Two-Party
  Computation
date: 2020-06-11 22:09:08
categories: Papers
tags: [MPC, 密码学, PPML, Secret Sharing]
---

*Daniel Demmler, Thomas Schneider, Michael Zohner*

[NDSS 2015](https://dblp.uni-trier.de/db/conf/ndss/ndss2015.html#Demmler0Z15)

https://www.ndss-symposium.org/ndss2015/ndss-2015-programme/aby-framework-efficient-mixed-protocol-secure-two-party-computation/

<!--more-->

# 摘要

安全计算使相互不信任的各方能够在其私有输入上联合计算函数，而不会泄露函数的输出以外的任何信息。半诚实模型中的通用安全计算协议已经被广泛研究，并形成了几个最佳实践。在这项工作中，我们设计并实现了一个混合协议框架，称为Aby，它有效地结合了基于算术共享、布尔共享和姚的乱码电路的安全计算方案，并为安全两方计算提供了最佳实践方案。我们的框架允许预计算几乎所有的密码运算，并基于预计算的不经意传输扩展在安全计算方案之间提供新颖、高效的转换。ABY支持多种标准操作，我们在本地网络和公共洲际云上执行基准测试。从我们的基准测试中，我们对安全计算协议的高效设计有了新的见解，最突出的是，基于不经意传输的乘法比基于同态加密的乘法要高效得多。我们使用ABY为三个示例应用(私有集合交集、生物特征匹配和模幂运算)构建混合协议，并表明它们比使用单一协议更有效。

**关键词**：安全的两方计算；混合协议；高效的协议设计





# 共享类型

在本节中，详细介绍作者的Aby框架使用的共享类型：算术共享(§III-A)、布尔共享(§III-B)和姚共享(§III-C)。对于每种共享类型，本文将在各自的小节中描述共享的语义、标准操作和最新技术。

## 算术共享

对于算术共享，在环 $\mathbb{Z}_{2^l}$(整数模 $2^l$)中相加地共享 $l$ 位值 $x$ 作为两个值的和。以下描述的协议基于[2]、[44]、[67]。首先，作者定义了共享语义(§III-A1)和操作(§III-A2)，并概述了基于算术共享的安全计算的相关工作(§III-A3)。然后，作者详细介绍了如何使用同态加密(§III-A4)或OT(§III-A5)生成算术乘法三元组；作者在§V-C后面的部分对这两种方法的性能进行了实验比较。在下面，作者假设要在环Z2‘中执行的所有算术运算，即所有运算都是($\bmod 2^l$)。



### 共享语义

算术共享基于在各方之间附加共享私有值，如下所示：

* **被共享的值。**对于一个 $l$ 位的 $x$ 的算术共享 $\langle x \rangle ^ A$，有 $\langle x \rangle ^A_0 + \langle x \rangle ^A_1 \equiv x (\bmod 2^l)$，其中 $\langle x \rangle ^ A_0, \langle x \rangle ^A_1 \in \mathbb{Z}_{2^l}$。
* **共享。**$\mathsf{Shr}^A_i(x)$：$P_i$ 选择 $r \in_R \mathbb{Z}_{2^l}$，设 $\langle x \rangle^A_i = x - r$，然后发送 $r$ 给 $P_{1-i}$，后者设 $\langle x \rangle^A_{1-i} = r$。
* **重构。**$\mathsf{Rec}^A_i(x)$：$P_{1-i}$ 发送它的份额 $\langle x \rangle^A_{1-i}$ 给 $P_i$，后者计算 $x = \langle x \rangle ^A_0 + \langle x \rangle ^A_1$。



### 操作

每个算术电路都是一系列加法和乘法门，评估如下：

* **加法。**$\langle z \rangle ^ A = \langle x \rangle ^ A + \langle y \rangle ^ A$：$P_i$ 在本地计算 $\langle z \rangle ^ A_i = \langle x \rangle ^ A_i + \langle y \rangle ^ A_i$。
* **乘法。**$\langle z \rangle ^ A = \langle x \rangle ^ A \cdot \langle y \rangle ^ A$：乘法使用形式为 $\langle c \rangle ^ A = \langle a \rangle ^ A \cdot \langle b \rangle ^ A$ 的预计算算术乘法三元组[4]：$P_i$ 设 $\langle e \rangle ^A_i = \langle x \rangle ^A_i - \langle a \rangle ^A_i$ 和 $\langle f \rangle ^A_i = \langle y \rangle ^A_i - \langle b \rangle ^A_i$，双方执行 $\mathsf{Rec}^A(e)$ 和 $$\mathsf{Rec}^A(f)$$，然后 $P_i$ 设 $\langle z \rangle ^ A_i = i \cdot e \cdot f + f \cdot \langle a \rangle ^ A_i + e \cdot \langle b \rangle ^ A_i + \langle c \rangle ^ A_i$。本文给出了预计算算术乘法的协议。



### 利用加性同态加密生成算术乘法三元组

通常，$\langle a \rangle ^ A \cdot \langle b \rangle ^ A = \langle c \rangle ^ A$ 形式的算术乘法三元组在设置阶段使用如协议1所示的加法同态加密方案来生成。该用于生成乘法三元组的协议在[2，附录A]中被提到为“众所周知的民间传说”。对于同态加密，我们使用Paillier[25]、[26]、[62]的密码系统，或者使用Damgard-Geisler-Krøigaard(DGK)[22]、[23]的密码系统，并使用如[10]、[32]、[52]中描述的Pohlig-Hellman算法[65]进行完全解密。在Paillier加密中，明文空间为 $Z_N$，使用参数为 $r$ 的统计盲化；在DGK加密中，将明文空间设置为 $\mathbb{Z}_{2^{2l+1}}$，使用参数为 $r$ 的完全盲化。关于安全性和正确性的证明，请参阅[67]和[66]。

**复杂度。**为了生成 $l$ 位乘法三元组，$P_0$ 和 $P_1$ 交换 $3$ 个密文，对于Paillier每个密文长度为 $2\varphi$ 比特(而DGK为 $\varphi$ 比特)，导致总共6个 $\varphi$ 比特的通信(对应的3个 $\varphi$ 比特)。对于Paillier加密，我们还使用了[67]中描述的打包优化，该优化将从 $P_1$ 到 $P_0$ 的多个消息打包成单个密文，从而减少解密次数，并将每个乘法三倍的通信量减少到 $4 \varphi + 2 \varphi / \lfloor \varphi / (2l+1+ \sigma) \rfloor$ 比特。

![](http://images.yingwai.top/picgo/aby2p1.png)



### 通过不经意传输生成算术乘法三元组

可以基于OT扩展生成算术乘法三元组，而不是使用同态加密。该议定书是在[33，Sect. 4.1]中提出的，并在[15]中使用。它允许使用OT有效地计算两个秘密共享值的乘积。在下面，本文将描述使用更有效的相关OT扩展的协议的一个微小变体。总体而言，可以在 $l$ 位串上使用 $2l$ 个相关的OT，即 C-OT$^{2l}_l$ (或者甚至在更短的串上，如下所述)来生成 $l$ 位乘法三元组。



## 布尔共享

布尔共享使用基于异或的秘密共享方案。为了简化表示，假定使用单个比特值；对于 $l$ 比特的值，每个操作都并行执行多次。



### 共享语义

* **被共享的值。**如果有 $\langle x \rangle ^B_0 \oplus \langle x \rangle ^B_1 = x$，其中 $\langle x \rangle ^B_0, \langle x \rangle ^B_1 \in \mathbb{Z}_2$，那么就说比特 $x$ 的布尔共享 $\langle x \rangle ^B$ 是在两方之间共享的。
* **共享。**$\mathsf{Shr}_i^B(x)$：$P_i$ 选择 $r \in_R \{0,1\}$，计算 $\langle x \rangle ^B_i = x \oplus r$，然后发送 $r$ 给 $P_{1-i}$，后者设 $\langle x \rangle ^B_{1-i} = r$。
* **重构。**$\mathsf{Rec}_i^B(x)$：$P_{1-i}$ 发送它的份额 $\langle x \rangle ^B_{1-i}$ 给 $P_i$，后者计算 $x = \langle x \rangle ^B_0 \oplus \langle x \rangle ^B_1$。



### 操作

每个有效计算的函数都可以表示为一个由异或门和与门组成的布尔电路，本文将在下面详细说明对它的求值。

* **异或。**$\langle z \rangle ^B = \langle x \rangle ^B \oplus \langle y \rangle ^B$：$P_i$ 本地计算 $\langle z \rangle ^B_i = \langle x \rangle ^B_i \oplus \langle y \rangle ^B_i$。
* **与。**$\langle z \rangle ^B = \langle x \rangle ^B \wedge \langle y \rangle ^B$：与使用预计算布尔乘法三元组 $\langle c \rangle ^B = \langle a \rangle ^B \wedge \langle b \rangle ^B$ 进行评估，如下所示：$P_i$ 计算 $\langle e \rangle ^B_i = \langle a \rangle ^B_i \oplus \langle x \rangle ^B_i$ 和 $\langle f \rangle ^B_i = \langle b \rangle ^B_i \oplus \langle y \rangle ^B_i$，双方执行 $\mathsf{Rec}^B(e)$ 和 $\mathsf{Rec}^B(f)$，然后 $P_i$ 设 $\langle z \rangle ^B_i = i \cdot e \cdot f \oplus f \cdot \langle a \rangle ^B_i \oplus e \cdot \langle b \rangle ^B_i \oplus \langle c \rangle ^B_i$。如文献[1]所述，使用 R-OT$^2_1$ 可以有效地预计算布尔乘法三元组。
* **多路复用。**对于多路复用器操作，本文使用[54]中提出的协议，该协议只需要 R-OT$^2_l$，而评估具有 $l$ 个与门的多路复用电路需要 R-OT$^{2l}_1$(参见[64]中的向量乘法三元组)。
* **其它。**对于标准功能，本文使用[69]中总结的深度优化电路结构。



## 姚氏共享

在Yao用于安全两方计算的乱码电路协议[74]中，一方(称为Garbler)将布尔函数加密为乱码电路，由另一方(称为赋值器)进行评估。更详细地，加布勒将要计算的函数表示为布尔电路，并将满足 $k_0^w, k_1^w \in \{0,1\}^{\kappa}$ 的两个线密钥 $(k_0^w, k_1^w)$ 分配给每条线 $w$。然后，Garbler使用加密函数 $\mathsf{Gb}$ 对两个输入线密钥的所有可能组合上的每个门的输出线密钥进行加密(详情见§III-C2中的共享)。然后，他将损坏的电路(由所有损坏的门组成)连同电路的相应输入键一起发送给评估器(参见§III-C1的共享)。评估者使用门的输入线路密钥迭代地解密每个乱码的门，以获得输出线路密钥(参见§III-C2中的与)，并最终重构电路的明文输出(参见§III-C1中的重构)。

在下面，我们假设 $P_0$ 充当Garbler，$P_1$ 充当评估者，并详细说明Yao共享，假设使用free-XOR[47]和点置换[53]优化的乱码方案。使用这些技术，Garbler随机选择 $R[0]=1$ 的全局 $\kappa$ 位串 $R$。对于每根导线 $w$，线密钥分别为 $k^w_0 \in_R \{0,1\}^{\kappa}$ 和 $k^w_1 = k^w_0 \oplus R$。最低有效位 $k^w_0[0]$、$k^w_1[0]=1 − k^w_0[0]$ 称为置换位。作者指出，YAO共享也可以与其他改进方案一起实例化。



### 共享语义

直观地，对于每根导线 $w$，$P_0$ 持有两个键 $k_0^w$ 和 $k_1^w$，以及 $P_1$ 持有这些键中的一个，而不知道它对应于两个明文值中的哪一个。为了简化表示，我们假定使用单个比特值；对于 $l$ 比特的值，每个操作都并行执行多次。

* **被共享的值。**值 $x$ 的乱码电路共享 $\langle x \rangle ^Y$ 被共享为 $\langle x \rangle ^Y_0 = k_0$ 和 $\langle x \rangle ^Y_1 = k_x = k_0 \oplus xR$。
* **共享。**$\mathsf{Shr}^Y_0(x)$：$P_0$ 取随机值 $\langle x \rangle ^Y_0 = k_0 \in_R \{0,1\}^{\kappa}$ 然后发送 $k_x = k_0 \oplus xR$ 给 $P_1$。
* **重构。**$\mathsf{Rec}^Y_i (x)$：$P_{1-i}$ 发送它的置换位 $\pi = \langle x \rangle ^Y_{1-i}[0]$ 给 $P_i$，后者计算 $x = \pi \oplus \langle x \rangle ^Y_i[0]$。



### 操作

使用姚氏共享，由XOR和AND门组成的布尔电路评估如下：

* **异或。**$\langle z \rangle ^Y = \langle x \rangle ^Y \oplus \langle y \rangle ^Y$ 使用 free-XOR 技术[47]进行评估：$P_i$ 本地计算 $\langle z \rangle ^Y_i = \langle x \rangle ^Y_i \oplus \langle y \rangle ^Y_i$。
* **与。**$\langle z \rangle ^Y = \langle x \rangle ^Y \wedge \langle y \rangle ^Y$ 评估如下：$P_0$ 使用 $\mathsf{Gb}_{\langle z \rangle ^Y_0}(\langle x \rangle ^Y_0, \langle y \rangle ^Y_0)$ 生成乱码表，其中 $\mathsf{Gb}$ 是[7]中定义的乱码函数。$P_0$ 发送该表给 $P_1$，后者使用它的密钥 $\langle x \rangle ^Y_1$ 和 $\langle y \rangle ^Y_1$ 进行解密。
* **其它。**对于标准功能，本文使用[45]中总结的尺寸优化的电路结构。



# 共享转换

在本节中，将详细介绍在不同共享之间进行转换的方法。首先解释已经存在的或直接的转换：Y2B(§IV-A)、B2Y(§IV-B)、A2Y(§IV-C)和A2B(§IV-D)。然后，我们详细说明了B2A(§IV-E)和Y 2A(§IV-F)的改进结构。作者将共享、重构和转换操作的复杂性总结在表1。

![](http://images.yingwai.top/picgo/aby2t1.png)

<center>
    <i>表1 用于l比特的值的共享、重构和转换操作的在线阶段的总计算(对称密码操作数)、通信和消息数。κ是对称安全参数。</i>
</center>




## 姚氏到布尔共享(Y2B)

将姚共享 $\langle x \rangle ^Y$ 转换为布尔共享 $\langle x \rangle ^B$ 是最简单的转换，基本上是免费的。关键的发现是 $\langle x \rangle ^Y_0$ 和 $\langle x \rangle ^Y_1$ 的排列位已经形成了 $x$ 的有效布尔共享。因此，$P_i$ 在本地设置 $\langle x \rangle ^B_i = Y2B(\langle x \rangle ^Y_i) = \langle x \rangle ^Y_i[0]$。



## 布尔到姚氏共享(B2Y)

将布尔共享 $\langle x \rangle ^B$ 转换为姚共享 $\langle x \rangle ^Y$ 非常类似于 $\mathsf{Shr}^Y_1$ 操作(参见§III-C1)：在下文中，我们假设 $x$ 是1比特；对于 $l$ 比特的值，每个运算都并行完成 $l$ 次。设 $x_0 = \langle x \rangle ^B_0$ 和 $x_1 = \langle x \rangle ^B_1$。$P_0$ 选取 $\langle x \rangle ^Y_0 = k_0 \in_R \{0,1\}^{\kappa}$。双方执行 OT$^1_{\kappa}$，其中 $P_0$ 作为具有输入的发送方 $(k_0 \oplus x_0 \cdot R;k_0 \oplus (1−x_0) \cdot R)$，而 $P_1$ 作为具有选择位 $x_1$ 的接收方，并且不经意地获得 $\langle x \rangle ^Y_1 = k_0 \oplus (x_0 \oplus x_1) \cdot R = k_x$。



## 算术到姚氏共享(A2Y)

算术共享 $\langle x \rangle ^A$ 到Yao共享 $\langle x \rangle ^Y$ 的转换在[35]、[44]、[46]中概述，并且可以通过安全地评估加法电路来完成。更准确地说，各方秘密地将他们的算术份额 $x_0 = \langle x \rangle ^A_0$ 和 $x_1 = \langle x \rangle ^A_1$ 共享为 $\langle x_0 \rangle ^Y = \mathsf{Shr}^Y_0(x_0)$ 和 $\langle x_1 \rangle ^Y = \mathsf{Shr}^Y_1(x_1)$，并计算 $\langle x \rangle ^Y = \langle x_0 \rangle ^Y + \langle x_1 \rangle ^Y$。



## 算术到布尔共享(A2B)

可以使用布尔加法电路(类似于§IV-C中描述的A2Y转换)或通过使用算术位提取电路[17]、[18]、[21]、[70]来完成将算术共享 $\langle x \rangle ^A$ 转换为布尔共享 $\langle x \rangle ^B$。如在[69]中总结的，布尔加法电路可以被实例化为大小优化的随 $O(l)$ 大小和深度变化的变量，或者实例化为随 $O(l\log_2l)$ 大小和 $O(\log_2l)$ 深度变化的深度优化的变量。在本文的框架中，$Y2B$ 转换是免费的，我们简单地计算 $\langle x \rangle ^B = A2B(\langle x \rangle ^A) = Y2B(A2Y(\langle x \rangle ^A))$，因为我们在§V-D中的评估表明，Yao共享中的加法比布尔共享中的加法更有效。



## 布尔到算术共享(B2A)

将 $l$ 比特布尔共享 $\langle x \rangle ^B$ 转换为算术共享 $\langle x \rangle ^A$ 的简单解决方案是评估布尔减法电路，其中 $P_0$ 输入 $\langle x \rangle ^B_0$ 和随机数 $r \in_R \{0,1\}^l$，并且设置 $\langle x \rangle ^A_0 = r$，然后 $P_1$ 输入 $\langle x \rangle ^B_1$ 并获得 $\langle x \rangle ^A_1 = x - r$。然而，评估这样的布尔减法电路将具有 $O(l)$ 大小和深度或者 $O(l \log_2 l)$ 大小和 $O(\log_2 l)$ 深度[69]。

为了提高转换的性能，可以使用与§III-A5中描述的算术乘法三次生成类似的技术。一般的想法是对每个比特执行OT，其中我们不经意地转移了两个值，这两个值被2的幂相加相关。接收方可以获得这些值中的一个，并且通过将它们相加，各方获得有效的算术份额。

更详细地说，在OT协议中，$P_0$ 充当发送者，$P_1$ 充当接收者。在第 $i$ 个OT中，$P_0$ 随机选择 $r_i \in_R \{0, 1\}^l$ 以及输入 $(s_{i,0}, s_{i_1})$，其中 $s_{i,0} = (1 - \langle x \rangle^B_0[i]) \cdot 2^i - r_i$ 以及 $s_{i,1} = \langle x \rangle^B_0[i] \cdot 2^i - r_i$，而 $P_1$ 输入 $\langle x \rangle^B_1[i]$ 作为选择位，收到输出 $s_{\langle x \rangle ^B_1[i]} = (\langle x \rangle^B_0[i] \oplus \langle x \rangle^B_1[i]) \cdot 2^i - r_i$。最后，$P_0$ 计算 $\langle x \rangle^A_0 = \sum^l_{i=1}r_i$，$P_1$ 计算 $\langle x \rangle^A_1$ $=\sum^l_{i=1}s_{\langle x \rangle^B_1[i]}$ $=\sum^l_{i=1}(\langle x \rangle^B_0[i] \oplus \langle x \rangle^B_1[i]) \cdot$ $2^i-\sum^l_{i=1}r_i$ $=\sum^l_{i=1}x[i] \cdot 2^i -$ $\sum^l_{i=1}r_i$ $=x - \langle x \rangle^A_0$。安全性和正确性类似于§III-A5的协议。

**复杂度。**观察到，由于我们传输一个随机元素和另一个作为相关性，并且只需要第 $i$ 个OT中的 $l-i$ 个最低有效位，所以我们可以使用C-OT和§III-A5中概述的相同技巧，导致(平均)C-OT$^l_{(l+1)/2}$ 和恒定轮数。相比之下，当使用布尔共享评估减法电路时，对于深度为 $O(\log_2l)$ 的电路各方将需要评估 $O(l \log_2 l)$次R-OT或对于深度为 $l$ 的电路评估 $2l$ 次ROT。本文的转换方法也比转换成姚共享(这已经需要 $2l$ OT)并在乱码电路中进行减法运算。



## 姚氏到算术共享(Y2A)

在[35]，[44]，[46]中描述了从姚共享 $\langle x \rangle ^Y$ 到算术共享 $\langle x \rangle ^A$ 的转换：$P_0$ 随机选择 $r \in_R \mathbb{Z}_{2^l}$，执行 $\mathsf{Shr}^Y_0$，然后双方评估布尔减法电路 $\langle d \rangle ^Y = \langle x \rangle ^Y - \langle r \rangle ^Y$，以获得它们的算术份额为 $\langle x \rangle ^A_0 = r$ 和 $\langle x \rangle ^A_1 = \mathsf{Rec}^Y_1(\langle d \rangle ^Y)$。

然而，由于我们免费执行 $Y2B$，而 $B2A$ 在计算和通信方面更便宜，我们建议计算 $\langle x \rangle ^A = Y2A(\langle x \rangle ^Y) = B2A(Y2B(\langle x \rangle ^Y))$。