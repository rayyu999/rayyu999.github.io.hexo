---
title: >-
  论文笔记 High-Throughput Semi-Honest Secure Three-Party Computation with an Honest
  Majority
date: 2020-06-10 20:07:19
categories: Papers
tags: [MPC, 密码学, Secret Sharing]
---

*Toshinori Araki, Jun Furukawa, Yehuda Lindell, Ariel Nof, Kazuma Ohara*

CCS 2016

https://dl.acm.org/doi/10.1145/2976749.2978331

<!--more-->

# 摘要

在这篇文章中，作者描述了一个新的信息论协议(和一个计算安全的变体)，用于诚实多数的安全三方计算。该协议的计算量和通信量非常小；对于布尔电路，每一方只为每个与门发送一位(对于异或门则不发送任何内容)。本文的协议在半诚实的攻击者面前是(基于模拟的)安全的，在恶意攻击者面前实现了客户机/服务器模型下的隐私。

在具有10Gbps连接的三个20核服务器的集群上，本文的协议的实现每秒执行超过130万次AES计算，涉及每秒处理超过70亿个门。此外，作者还开发了一个Kerberos扩展，它使用服务器之间共享的密钥/密码，用本文的协议取代了MIT-Kerberos中密钥分发中心(KDC)上的票证授予-票证加密。这样可以在保护密码的同时使用Kerberos。本文的实现能够支持每秒超过35,000个登录的登录风暴，即使对于非常大的组织也足够了。本文的工作证明了在标准硬件上实现高通量安全计算是可能的。



# THE NEW PROTOCOL

## Securely Computing Boolean Circuits

为了简化说明，首先描述具有与和异或门的布尔电路的特殊情况的协议。假设各方 $P_1,P_2,P_3$ 能够获得随机的 $x_1, x_2, x_3 \in \{0,1 \}$，使得 $x_1 \oplus x_2 \oplus x_3 = 0$。

**秘密共享。**作者定义了一个3取2的秘密共享方案，记为$\left( \begin{array}{c} 3\\2 \end{array} \right)$-共享，如下所示。为了共享比特 $v$，分发者在 $x_1 \oplus x_2 \oplus x_3 = 0$ 的约束下选择三个随机比特 $x_1, x_2, x_3 \in \{0,1 \}$。然后：

* $P_1$ 的份额为 $(x_1, a_1)$，其中 $a_1 = x_3 \oplus v$；
* $P_2$ 的份额为 $(x_2, a_2)$，其中 $a_2 = x_1 \oplus v$；
* $P_3$ 的份额为 $(x_3, a_3)$，其中 $a_3 = x_2 \oplus v$。

很明显，任何一方的份额都不能揭示关于v的任何信息。此外，任何两个份额都足以获得 $v$；例如，给定 $x_1,x_2,a_1,a_2$，我们可以计算 $v = a_2 \oplus x_1$。

**异或(加法)门。**设 $(x_1,a_1),(x_2,a_2),(x_3,a_3)$ 是 $v_1$ 的秘密共享，$(y_1,b_1),(y_2,b_2),(y_3,b_3)$ 是 $v_2$ 的秘密共享。然后，为了计算 $v_1$ 和 $v_2$ 的秘密共享，每个 $P_i$ 在本地计算 $z_i = x_i \oplus y_i$ 和 $c_i = a_i \oplus b_i$ 得到 $(z_i, c_i)$ (不需要通信)。为了查看结果是否构成 $v_1 \oplus v_2$ 的有效$\left( \begin{array}{c} 3\\2 \end{array} \right)$-共享，首先注意到 $z_1 \oplus z_2 \oplus z_3 = 0$ (因为 $x_1 \oplus x_2 \oplus x_3 = 0$ 和 $y_1 \oplus y_2 \oplus y_3 = 0$)。接下来，观察到对于每个 $i \in \{1,2,3\}$，它持有 $c_i = z_{i−1} \oplus (v_1 \oplus v_2)$，其中 $i=1$ 时 $i−1=3$；例如，我们有 $c_1=a_1 \oplus b_1$ $=x_3 \oplus v_1 \oplus y_3 \oplus v_2$ $=(x_3 \oplus y_3)\oplus (v_1 \oplus v_2)$ $=z_3\oplus (v_1 \oplus v_2)$。因此，这构成具有随机性 $z_1,z_2,z_3$ 的 $v_1 \oplus v_2$ 的共享。

**与(乘法)门。**现在展示各方如何计算与(乘法)门；此子协议要求每一方只发送单个比特。该协议分两个阶段工作：第一阶段双方计算输入位的与的简单$\left( \begin{array}{c} 3\\3 \end{array} \right)$异或-共享，第二阶段将$\left( \begin{array}{c} 3\\3 \end{array} \right)$-共享转换为上述定义的$\left( \begin{array}{c} 3\\2 \end{array} \right)$-共享。

设 $(x_1,a_1),(x_2,a_2),(x_3,a_3)$ 是 $v_1$ 的秘密共享，$(y_1,b_1),(y_2,b_2),(y_3,b_3)$ 是 $v_2$ 的秘密共享。假设各方 $P_1,P_2,P_3$ 分别具有相关随机性 $\alpha, \beta, \gamma$，其中 $\alpha \oplus \beta \oplus \gamma = 0$。双方计算 $v_1 \cdot v_2 = v_1 \wedge v_2$ 的$\left( \begin{array}{c} 3\\2 \end{array} \right)$-份额如下(从这里开始，将简单地将 $a$ 和 $b$ 的积表示为 $ab$)：

1. 第一步——计算$\left( \begin{array}{c} 3\\3 \end{array} \right)$-共享：

   * $P_1$ 计算 $r_1 = x_1 y_1 \oplus a_1 b_1 \oplus \alpha$，然后发送 $r_1$ 给 $P_2$；
   * $P_2$ 计算 $r_2 = x_2 y_2 \oplus a_2 b_2 \oplus \beta$，然后发送 $r_2$ 给 $P_3$；
   * $P_3$ 计算 $r_3 = x_3 y_3 \oplus a_3 b_3 \oplus \gamma$，然后发送 $r_3$ 给 $P_1$。

   这些消息是并行地计算和发送的。

2. 第二步——计算$\left( \begin{array}{c} 3\\2 \end{array} \right)$-共享：在此步骤中，各方根据其给定的$\left( \begin{array}{c} 3\\3 \end{array} \right)$-共享和上一步中发送的消息构建$\left( \begin{array}{c} 3\\2 \end{array} \right)$-共享，只需要本地计算。

   *  $P_1$ 存储 $(z_1, c_1)$，其中 $z_1 = r_1 \oplus r_3$，以及 $c_1 = r_1$；
   *  $P_2$ 存储 $(z_2, c_2)$，其中 $z_2 = r_2 \oplus r_1$，以及 $c_2 = r_2$；
   *  $P_3$ 存储 $(z_3, c_3)$，其中 $z_3 = r_3 \oplus r_2$，以及 $c_3 = r_3$；

**对第一步的解释：**证明第一步中，$r_1 \oplus r_2 \oplus r_3 = v_1 \wedge v_2$。首先注意到：
$$
a_1 b_1 = (x_3 \oplus v_1) (y_3 \oplus v_2) = x_3 y_3 \oplus x_3 v_2 \oplus y_3 v_1 \oplus v_1 v_2 \tag{1}
$$
类似地有 $a_2 b_2 = x_1 y_1 \oplus x_1 v_2 \oplus y_1 v_1 \oplus v_1 v_2$ 和 $a_3 b_3 = x_2 y_2 \oplus x_2 v_2 \oplus y_2 v_1 \oplus v_1 v_2$。因此，
$$
\begin{align}
r_1 &\oplus r_2 \oplus r_3 \\
 &= (x_1 y_1 \oplus a_1 b_1 \oplus \alpha) \oplus (x_2 y_2 \oplus a_2 b_2 \oplus \beta) \oplus (x_3 y_3 \oplus a_3 b_3 \oplus \gamma) \\
 &= x_1 y_1 \oplus x_2 y_2 \oplus x_3 y_3 \oplus b_1 a_1 \oplus a_2 b_2 \oplus a_3 b_3 \\
 &= x_1 y_1 \oplus x_2 y_2 \oplus x_3  y_3 \oplus (x_3 y_3 \oplus x_3 v_2 \oplus y_3 v_1 \oplus v_1 v_2) \\
 & \qquad \qquad \qquad \qquad \quad  \oplus (x_1 y_1 \oplus x_1 v_2 \oplus y_1 v_1 \oplus v_1 v_2) \\
 & \qquad \qquad \qquad \qquad \quad  \oplus (x_2 y_2 \oplus x_2 v_2 \oplus y_2 v_1 \oplus v_1 v_2) \\
 &= (x_1 \oplus x_2 \oplus x_3) v_2 \oplus (y_1 \oplus y_2 \oplus y_3) v_1 \oplus v_1 v_2 = v_1 v_2
\end{align}
$$
第二个等号成立是因为 $\alpha \oplus \beta \oplus \gamma = 0$，第三个等号成立是因为公式 $(1)$，第四个等号成立就是简单的删除重复元素，最后一个等号成立是因为 $x_1 \oplus x_2 \oplus x_3 = y_1 \oplus y_2 \oplus y_3 = 0$。

**对第二步的解释：**根据定义，为了证明结果是有效的 $v_1v_2$ 的$\left( \begin{array}{c} 3\\2 \end{array} \right)$-共享，我们需要证明 $z_1,z_2,z_3$ 使得 $z_1 \oplus z_2 \oplus z_3 = 0$，并且 $c_1,c_2,c_3$ 是所定义的形式。

首先，$z_1 \oplus z_2 \oplus z_3 = (r_1 \oplus r_3) \oplus (r_2 \oplus r_1) \oplus (r_3 \oplus r_2) = 0$。其次，注意到$c_1 \oplus c_2 \oplus c_3 = r_1 \oplus r_2 \oplus r_3 = v_1 v_2$，移项得 $c_1 = r_1 = v_1 v_2 \oplus r_2 \oplus r_3$。又因为 $r_2 \oplus r_3 = z_3$，于是 $c_1 = v_1 v_2 \oplus z_3$，得证。对于 $c_2, c_3$ 也可以同样证明。

**协议。**完整的三方协议以自然的方式工作。各方首先使用秘密共享方法共享其输入。然后，它们根据电路的预定拓扑顺序计算电路中的每个XOR与AND门。最后，各方在输出线上重建其输出。(在客户端/服务器模型中，外部客户端将其输入的共享发送给三方，然后三方根据收到的共享以相同的方式计算电路。)

请注意，每一方只与另一方进行通信。这一性质也适用于Sharemind的协议。然而，本文的秘密共享方案和乘法协议是完全不同的。



## Generating Correlated Randomness

这里将说明如何高效地为每个与门生成随机比特 $\alpha, \beta, \gamma \in \{0, 1\}$ ，满足 $\alpha \oplus \beta \oplus \gamma = 0$。

**信息论相关随机性。**通过使每一方 $P_i$ 简单地选择随机 $\rho_i \in \{0,1\}$ 并将其发送到 $P_{i+1}$ (其中 $P_3$ 发送到 $P_1$)，可以安全地生成具有完美安全性的相关随机性。然后，每一方将其随机比特作为其选择的比特与其接收的比特的异或：$P_1$ 计算 $\alpha = \rho_3 \oplus \rho_1$，$P_2$ 计算 $\beta = \rho_1 \oplus \rho_2$ 以及 $P_3$ 计算 $\gamma = \rho_2 \oplus \rho_3$。观察到 $\alpha + \beta + \gamma = 0$ 满足要求。此外，如果 $P_1$ 损坏，则它除了知道 $\beta \oplus \gamma = \alpha$ 之外，对 $\beta$ 和 $\gamma$ 一无所知。这是因为 $\beta$ 和 $\gamma$ 在他们的计算中都包含了 $\rho_2$，而这对于 $P_1$ 是未知的。类似的论点也适用于损坏的 $P_2$ 或 $P_3$。尽管此解决方案既优雅又简单，但作者使用了不同的方法。这是因为这将使每个与门的通信增加一倍；确实，这仍然是非常少的通信。但是，考虑到通信是瓶颈，这将使吞吐量减半。

**计算相关随机性。**接下来作者展示了如何通过计算安全地计算相关随机性，而不需要除短初始设置之外的任何交互。这使我们能够维持各方只需要在每个与门传输单个比特的当前情况。设 $\kappa$ 为安全参数，$F: \{0,1\}^{\kappa} \times \{0,1\}^{\kappa} \rightarrow \{0,1\}$ 为输出单比特的伪随机函数。

1. **Init：**

   * 每个 $P_i$ 选择一个随机的 $k_i \in \{0,1\}^{\kappa}$；
   * $P_1$ 发送 $k_1$ 给 $P_3$，$P_2$ 发送 $k_2$ 给 $P_1$，$P_3$ 发送 $k_3$ 给 $P_2$。

   $P_1$ 持有 $k_1, k_2$，$P_2$ 持有 $k_2, k_3$，$P_3$ 持有 $k_3, k_1$。

2. **GetNextBit：**给定唯一标识符 $id \in \{0,1\}^{\kappa}$，

   * $P_1$ 计算 $\alpha = F_{k_1}(id) \oplus F_{k_2}(id)$；
   * $P_2$ 计算 $\beta = F_{k_2}(id) \oplus F_{k_3}(id)$；
   * $P_3$ 计算 $\gamma = F_{k_3}(id) \oplus F_{k_1}(id)$。

注意到 $\alpha \oplus \beta \oplus \gamma = 0$。此外，$P_1$ 不知道用于生成 $\beta$ 和 $\gamma$ 的 $k_3$。因此，在 $\beta$ 和 $\gamma$ 的约束下，$\beta \oplus \gamma = \alpha$ 对 $P_1$ 是伪随机的。实际上，$id$ 可以是所有各方在每次调用 **GetNextBit** 时本地递增的计数器。



## The Ring Modulo $2^n$ and Fields

上一节讲到的是布尔电路下的协议，接下来将其推广到模 $2^n$ 的环和大于 $2$ 的任意域的一般情况。当取 $n=1$ 时，加法(和减法)与异或相同，乘法与与相同。在这种情况下，这里的协议与第2.1节中描述的协议完全相同。

$\left( \begin{array}{c} 3\\2 \end{array} \right)$**-共享。**为了共享一个元素 $v \bmod 2^n$，分发者选择三个满足约束 $x_1 + x_2 + x_3 = 0$ 的随机元素 $x_1, x_2, x_3 \in \mathbb{Z}_{2^n}$。然后 $P_1$ 的份额为 $(x_1, a_1)$，其中 $a_1 = x_3 - v$；$P_2$ 的份额为 $(x_2, a_2)$，其中 $a_2 = x_1 - v$；$P_3$ 的份额为 $(x_3, a_3)$，其中 $a_3 = x_2 - v$。跟布尔电路的情况一样，每一方的份额都没有暴露关于 $v$ 的信息，而且任意两方可以重构 $v$。

**加法门。**与布尔电路的情况一样，各方在本地将对应的份额进行相加模 $2^n$ 即可。

**乘法门。**设 $(x_1,a_1),(x_2,a_2),(x_3,a_3)$ 是 $v_1$ 的秘密共享，$(y_1,b_1),(y_2,b_2),(y_3,b_3)$ 是 $v_2$ 的秘密共享。假设各方 $P_1,P_2,P_3$ 分别有 $\alpha, \beta, \gamma \in \mathbb{Z}_{2^n}$，其中 $\alpha + \beta + \gamma = 0$。为了计算两个值的积的$\left( \begin{array}{c} 3\\2 \end{array} \right)$-份额，各方计算：

1. $P_1$ 计算 $r_1 = \frac{a_1 b_1 - x_1 y_1 + \alpha}{3}$，然后发送 $r_1$ 给 $P_2$；
2. $P_2$ 计算 $r_2 = \frac{a_2 b_2 - x_2 y_2 + \beta}{3}$，然后发送 $r_2$ 给 $P_3$；
3. $P_3$ 计算 $r_3 = \frac{a_3 b_3 - x_3 y_3 + \gamma}{3}$，然后发送 $r_3$ 给 $P_1$；
4. $P_1$ 将它的份额定义为 $z_1 = r_3 - r_1$ 和 $c_1 = -2r_3 - r_1$；
5. $P_2$ 将它的份额定义为 $z_2 = r_1 - r_2$ 和 $c_2 = -2r_1 - r_2$；
6. $P_3$ 将它的份额定义为 $z_3 = r_2 - r_3$ 和 $c_3 = -2r_2 - r_3$。

上面的计算是合法的，因为 $3$ 与 $2^n$ 是互质的；因此，$3$ 是可逆的。此外，上述结果在3个以上元素的有限域上都成立。

为了验证 $r_1 + r_2 + r_3 = v_1 v_2$，首先观察到
$$
a_1 b_1  = (x_3 - v_1)(y_3 - v_2) = x_3 y_3 - x_3 v_2 - y_3 v_1 + v_1 v_2 \tag{2}
$$
类似地，$a_2 b_2 = x_1 y_1 - x_1 v_2 - y_1 v_1 + v_1 v_2$，$a_3 b_3 = x_2 y_2 - x_2 v_2 - y_2 v_1 + v_1 v_2$。然后
$$
\begin{align}
3(&r_1 + r_2 + r_3) \\
&= a_1 b_1 - x_1 y_1 + \alpha + a_2 b_2 - x_2 y_2 + \beta + a_3 b_3 - x_3 y_3 + \gamma \\
&= a_1 b_1 + a_2 b_2 + a_3 b_3 - x_1 y_1 - x_2 y_2 - x_3 y_3 \\
&= 3v_1 v_2 - v_1(y_1 + y_2 + y_3) - v_2(x_1 + x_2 + x_3) = 3v_1 v_2
\end{align}
$$
接下来证明各方的$\left( \begin{array}{c} 3\\2 \end{array} \right)$-份额是有效的：根据定义，各方的份额为 $(z_1, z_3 - v_1 v_2), (z_2, z_1 - v_1 v_2)$ 和 $(z_3, z_2 - v_1 v_2)$，其中 $z_1 + z_2 + z_3 = 0 \bmod 2^n$。首先后者很容易看出来是成立的，根据 $z_1, z_2, z_3$ 的定义。其次，$P_1$ 持有 $c_1$ $= -2 r_3 - r_1$ $= -r_3 - r_3 - r_1 - r_2 + r_2$ $=(r_2 - r_3)-(r_1 + r_2 + r_3)$ $=z_3 - v_1 v_2$，满足要求。对于 $P_2$ 和 $P_3$ 的情况同样成立。

**生成相关随机性。**各方使用与第2.2节所述相同的(计算)方法，但有以下不同之处。首先，我们假设 $F_k$ 是将字符串映射到 $\mathbb{Z}_{2^n}$ (或等价于 $\{0,1\}^n$)的伪随机函数。其次，$P_1$计算 $\alpha = F_{k_1}(id) - F_{k_2}(id)$，$P_2$计算 $\beta = F_{k_2}(id) - F_{k_3}(id)$，$P_3$计算 $\gamma = F_{k_3}(id) - F_{k_1}(id)$。