---
title: '论文笔记 Improved Garbled Circuit: Free XOR Gates and Applications'
date: 2020-06-16 11:07:59
categories: Papers
tags: [MPC, 密码学, Garbled Circuit]
---

*Vladimir Kolesnikov, Thomas Schneider*

 [ICALP (2) 2008](https://dblp.uni-trier.de/db/conf/icalp/icalp2008-2.html#KolesnikovS08)

https://dl.acm.org/doi/10.1007/978-3-540-70583-3_40

<!--more-->

# 摘要

本文提出了一种新的用于两方安全函数评估(SFE)的乱码电路结构。在本文的单轮协议中，XOR门的评估是“免费的”，这导致了对最佳乱码电路实现的相应改进(例如Fairplay)。

本文几乎完全使用XOR门来构建置换网络和通用电路(UC)；这使得它们的SFE(在计算和通信方面)提高了多达4倍。本文还改进了整数加法和等价性测试，最高可达2倍。

本文依赖随机预言(RO)假设。本文的构造在半诚实模型中被证明是安全的。



## 本文的贡献

* 在半诚实模型下，本文提出了一种新的用于两方安全函数评估(SFE)的乱码电路结构。
* 在本文的一轮协议中，XOR门的计算是“免费的”(也就是说，不使用相关的乱码表和相应的散列或对称密钥操作)。在处理其他门时，本文的构造与最好的乱码电路实现(例如Fairplay)一样有效。





# Setting and Preliminaries

两方通用安全函数评估(SFE)允许双方评估其各自输入 $x$ 和 $y$ 上的任何函数，同时保持 $x$ 和 $y$ 的隐私。

本文考虑具有 $k$ 个门和任意扇出的非循环布尔电路。也就是说，每个门的(单个)输出可以用作任意数量的门的输入。本文假设电路的门 $G_1,...,G_k$ 按拓扑排序。该顺序(不一定是唯一的)确保第 $i$ 个门 $G_i$ 不具有作为连续门 $G_j,\ j>i$ 的输出的输入。总是可以通过 $O(k)$ 计算在非循环电路上获得拓扑顺序。

本文专注于半诚实的模式，在这种模式下，参与者遵循协议，但试图从执行记录中学习信息。

本文使用以下标准符号：$\in_R$ 表示均匀随机采样，$\|$ 表示位串的连接。$\langle a, b \rangle$ 是一个包括了 $a,b$ 两个元素的向量，且其比特串表示为 $a \| b$。$W_c = g(W_a, W_b)$ 表示2输入门 $G$，其用输入线 $W_a,W_b$ 和输出线 $W_c$ 计算函数 $g: \{0,1\}^2 \rightarrow \{0,1 \}$。

设 $N$ 为安全参数。设 $S$ 是无限集，设 $X=\{X_s\}_{s \in S}$ 和 $Y=\{Y_s\}_{s \in S}$ 为分布全体。我们说 $X$ 和 $Y$ 在计算上是不可区分的，记为 $X \overset{c}{\equiv} Y$，如果对每个非均匀多项式时间微分器 $D$ 和所有足够大的 $s \in S, \ |Pr[D(X_s)=1]−Pr[D(Y_s)=1]|<1/p(|s|) $ 对每个多项式 $p$。

**随机预言。**RO模型是一个有用的抽象。RO只是一个随机选择的函数 $\{0,1\}^* \mapsto \{0,1\}^N$ -一个不能被多项式参与者完全存储或遍历的大对象。RO模型为所有参与者提供了Oracle访问此类功能的权限。实际上，RO由散列函数(如SHA)建模。

**不经意传输(OT)。**2选1OT是两方协议。*发送方* $P_1$ 具有两个秘密 $m_0,m_1$，*接收方* $P_2$ 具有选择位 $i \in \{0,1\}$。在协议的最后，$P_2$ 得到 $m_i$，但没有获得关于 $m_{i-1}$ 的任何信息，$P_1$ 没有获得关于 $i$ 的任何信息。

**姚氏乱码电路(GC)。**参与者 $P_1$ 首先扰乱电路 $C$：对于每条线 $W_i$ 的值 $j$，他随机选择两个秘密 $w^0_i$ 和 $w^1_i$，其中 $w^j_i$ 是一个乱值，或者说是乱码。(注：$w^j_i$ 不透露 $j$。)。此外对于每个门 $G_i$，$P_1$ 创建具有以下属性的乱码表 $T_i$ 并将其发送到 $P_2$：给定一组 $G_i$ 的输入乱码，$T_i$ 允许恢复相应 $G_i$ 的输出的乱码，而不透露其他信息。然后参与方输入的乱码被(不经意地)传输到 $P_2$。现在 $P_2$ 可以使用表格 $T_i$ 通过逐个门地评估乱码电路来简单地获得乱码输出。如果在给定输入上对 $C$ 求值时，$W_i$ 取值 $j$，则称 $W_i$ 的乱码 $w^j_i$ *有效*。观察到，对于每根导线，$P_2$ 只能获得其有效乱码。电路的输出线没有乱码(或其乱码被公布)，因此 $P_2$ (仅)获知电路的输出，而不学习内部线路值。$P_1$ 从(半诚实的) $P_2$ 学习输出。(这一步在半诚实模型中是微不足道的，通常不会在分析中考虑。)。GC的正确性源于表 $T_i$ 的构建方法。任何一方都不会从协议执行中获知任何附加信息。



# 本文的协议

首先，本文展示了XOR门 $G$ 的SFE实现，它源自[14]中的一个。设 $G$ 有两根输入线 $W_a, W_b$，输出线 $W_c$。按如下方式对导线值进行乱码处理：随机选择 $w^0_a, w^0_b, R \in_R \{0,1\}^N$。设 $w^0_c = w^0_a \oplus w^0_b$ 和 $\forall i \in \{a,b,c\}: w^1_i = w^0_i \oplus R$。很容易看出，乱码门输出只需对乱码门输入进行异或运算即可获得：
$$
\begin{align}
w^0_c &= w^0_a \oplus w^0_b = (w^0_a \oplus R) \oplus (w^0_b \oplus R) = w^1_a \oplus w^1_b \\
w^1_c &= w^0_c \oplus R = w^0_a \oplus (w^0_b \oplus R) = w^0_a \oplus w^1_b = (w^0_a \oplus R) \oplus w^0_b = w^1_a \oplus w^0_b
\end{align}
$$
且乱码 $w^j_i$ 不会透露它们对应的线路值。

上述异或结构对乱码值施加的限制：电路中每根导线的两个值的乱码必须相差相同的值，即对于某个全局 $R$，$\forall i: w^1_i = w^0_i \oplus R$。相比之下，在以前的GC结构中，所有乱码 $w^j_i$ 是独立随机选择的，并且安全性证明依赖于该性质。



## 本文的乱码电路构造

设 $C$ 是一个电路。首先注意到，NOT门可以通过简单地消除它们并颠倒导线的值和乱码的对应关系来“免费”实现。

在下面的Alg.1中，每个乱码 $w= \langle k, p \rangle$ 由密钥 $k \in \{0,1\}^N$ 和排列位 $p \in \{0,1\}$ 组成。密钥用于表条目的解密，$p$ 用于选择要解密的条目。每条导线的两个乱码 $w^0_i, w^1_i$ 按照异或构造的要求相关：对于选定的 $R\in_R \{0,1\}^N$，$\forall i: w^1_i = \langle k^1_i, p^1_i \rangle = \langle k^0_i \oplus R, p^0_i \oplus 1 \rangle$，其中 $w^0_i = \langle k^0_i, p^0_i \rangle$。$H: \{0,1\}^* \mapsto \{0,1\}^{N+1}$ 是一个RO。

现在我们将上述想法形式化，并给出GC结构(Alg.1)和评估(Alg.2)。在SFE中，Alg.1由 $P_1$ 执行，Alg.2由 $P_2$ 执行。

![](http://images.yingwai.top/picgo/freeXORa1.png)

注意，本文的表项加密(步骤3(C)iii)类似于Fairplay。Fairplay使用 $e_{v_a,v_b} = H(k^{v_a}_a \| i \| p^{v_a}_a \| p^{v_b}_b) \oplus$ $H(k^{v_b}_b \| i \| p^{v_a}_a \| p^{v_b}_b) \oplus$ $w^{g_i(v_a, v_b)}_c$。这是一个不重要的区别；我们可以使用Fairplay的加密。

**安全方面的直觉。**Alg.1使用RO $H$ 的输出作为一次性垫来加密乱码表中的乱码输出值(步骤3(C)III)和乱码输出表(步骤4a)。注意，在本文的整个构造过程中，$H$ 的输入(密钥和门索引)的任何特定组合都用于加密至多一个表项。(本文假设 $H$ 中的串联和字符串表示是“正确的”。)。此外，由于被篡改电路的评估者每条线路只知道一个被篡改的值，所以他可以精确地解密 $G_i$ 的乱码表中的一个条目。所有其他条目都使用多次赋值器无法猜到的至少一个密钥进行加密。因此，在他看来，每根电线的两个乱值中的一个看起来是随机的。

现在给出相应的GC评估算法，由 $P_2$ 运行。回想一下，$P_2$ 会从 $P_1$ 获取所有乱码表格和 $P_1$ 输入值的乱码。由 $P_2$ 保存的输入值的乱码通过OT发送。

![](http://images.yingwai.top/picgo/freeXORa2.png)

GC构造和评估算法可以直接用于以标准方式获得基于GC的SFE协议。为完整起见，本文包含了对此协议的描述。

----

**Protocol 1.** *(Two-party SFE protocol):*

- **Inputs:**  $P_1$ has private input $x = \langle x_1,..,x_{u_1} \rangle \in \{0,1\}^{u_1}$and $P_2$ has private input $y = \langle y_1,..,y_{u_2} \rangle \in \{0,1\}^{u_2}$.

- **Auxiliary input:**  A boolean acyclic circuit $C$ such that $\forall x \in \{0,1\}^{u_1}, y \in \{0,1\}^{u_2}$, it holds that $C(x,y) = f(x,y)$, where $f: \{0,1\}^{u_1} \times \{0,1\}^{u_2} \rightarrow \{0,1\}^v$. We require that $C$ is such that if a circuit-output wire leaves some gate $G$, then gate $G$ has no other wires leading from it into other gates (i.e., no circuit-output wire is also a gate-input wire). Likewise, a circuit-input wire that is also a circuit-output wire enters no gates. We also require that $C$ is modified to contain no NOT-gates and all n-input XOR-gates with $n > 2$ replaced by 2-input XOR-gates as described in Section 3.1.

- **The protocol:**

  1. $P_1$ constructs the garbled circuit using Algorithm 1 and sends it (i.e. the garbled tables) to $P_2$.

  2. Let $W_1,.., W_{u_1}$ be the circuit input wires corresponding to $x$, and let $W_{u_1+1},..,W_{u_1+u_2}$ be the circuit input wires corresponding to $y$. Then, 

     ​	(a) $P_1$ sends $P_2$ the garbled values $w^{x_1}_1,..,w^{x_{u_1}}_{u_1}$.

     ​	(b) For every $i \in \{1, .., u_2\}$, $P_1$ and $P_2$ execute a 1-out-of-2 oblivious transfer protocol, where $P_1$’s input is $(k^0_{u_1+i}, k^1_{u_1+i})$, and $P_2$’s input is $y_i$. All $u_2$ OT instances can be run in parallel.

  3. $P_2$ now has the garbled tables and the garblings of circuit’s input wires. $P_2$ evaluates the garbled circuit, as described in Alg. 2, and outputs $f(x, y)$.

----

