---
title: >-
  论文笔记 Efficient Multi-Key Homomorphic Encryption with Packed Ciphertexts with
  Application to Oblivious Neural Network Inference
date: 2020-07-09 14:56:51
categories: Papers
tags: [HE, PPML, Neural Network]
---

*Hao Chen, Wei Dai, Miran Kim, Yongsoo Song*

CCS 2019

https://dl.acm.org/doi/10.1145/3319535.3363207

<!--more-->

# 本文的贡献

本文设计了BFV[6，22]和CKKS[16]方案的多密钥变体。本文提出了一种新的生成重线性化密钥的方法，与文献[13]中的以前技术相比，该方法更简单、更快。此外，本文将这些方案[9，12，14]的最新自举算法应用于多密钥场景，以构建具有压缩密文的多密钥完全同态加密。最后，作者使用Microsoft SEAL[47]给出了本文的多密钥方案的概念验证实现，并给出了实验结果。据作者所知，这是支持打包密文的MKHE方案的第一个实际实现。

本文还提出了MKHE的第一个可行的应用，它安全地评估了预先训练的卷积神经网络(CNN)模型。本文构建了一个高效的协议，云服务器使用模型提供者提供的分类器为数据所有者提供在线预测服务，同时使用MKHE保护数据和模型的隐私。如图1所示，本文的方案支持多密钥操作，使得以较低的端到端延迟和接近最佳的数据和模型提供者成本来实现这一点。服务器可以存储以不同密钥加密的大量密文，但是特定任务的计算成本仅取决于与电路相关的各方的数量。作者注意到，本文的解决方案比单密钥HE更有优势，因为ML模型提供者不需要将未加密的模型发送到服务器。

![](http://images.yingwai.top/picgo/emkhef1.png)



# BACKGROUND

## 记号法

除非另有说明，否则所有对数都以 $2$ 为底。我们用粗体表示向量，例如$\mathbf{a}$，用大写粗体表示矩阵，例如 $\mathbf{A}$。本文用 $\langle \mathbf{u}, \mathbf{v} \rangle$ 表示两个向量 $\mathbf{u}, \mathbf{v}$ 的通常点积。对于实数 $r$，$\lfloor r \rceil$ 表示最表示最接近 $r$ 的整数，在平局的情况下向上舍入。本文用 $x \leftarrow D$ 表示按分布 $D$ 抽样的 $x$。对于有限集 $S$，$U(S)$ 表示 $S$ 上的均匀分布。本文用 $\lambda$ 表示安全参数：所有已知的对作用域内的密码方案的有效攻击都应该采用 $\Omega(2^\lambda)$ 位运算。



## 多密钥同态加密

多密钥同态加密是一种密码系统，它允许我们对可能在不同密钥下加密的密文评估算术电路。

设 $\mathcal{M}$ 是具有算术结构的消息空间。MKHE方案MKHE由五个PPT算法($\mathsf{Setup}, \mathsf{KeyGen}, \mathsf{Enc}, \mathsf{Dec}, \mathsf{Eval}$)组成。本文假设每个参与方都有对其公钥和私钥的引用(索引)。多密钥密文隐式包含*有序*集合 $T = \{id_1, ..., id_k\}$ 相关联的引用。例如，新的密文 $\mathsf{ct} \leftarrow \mathsf{MKHE.Enc}(\mu; \mathsf{pk}_{id})$ 对应于单个元素集合 $T = \{id\}$，但是随着来自不同方的密文之间的计算的进行，参考集合的大小变得更大。

* **初始化**：$pp \leftarrow \mathsf{MKHE.Setup}(1^\lambda)$。将安全参数作为输入并返回公共参数化。本文假设所有其他算法都隐含地将 $pp$ 作为输入。

* **密钥生成**：$(\mathsf{sk}, \mathsf{pk}) \leftarrow \mathsf{MKHE.KeyGen}(pp)$。输出一对私钥和公钥。

* **加密**：$\mathsf{ct} \leftarrow \mathsf{MKHE.Enc}(\mu; \mathsf{pk})$。加密明文 $\mu \in \mathcal{M}$ 并输出密文 $\mathsf{ct} \in \{0,1 \}^*$。

* **解密**：$\mu \leftarrow \mathsf{MKHE.Dec}(\overline{\mathsf{ct}}; \{\mathsf{sk}_{id}\}_{id \in T})$。给定具有对应密钥序列的密文 $\overline{\mathsf{ct}}$，输出明文 $\mu$。

* **同态评估**：
  $$
  \overline{\mathsf{ct}} \leftarrow \mathsf{MKHE.Eval}(\mathcal{C}, (\overline{\mathsf{ct}}_1,...,\overline{\mathsf{ct}}_l),\{\mathsf{pk}_{id}\}_{id \in T})
  $$
  给定电路 $\mathcal{C}$，多密钥密文的元组 $(\overline{\mathsf{ct}}_1,...,\overline{\mathsf{ct}}_l)$ 和对应的一组公钥 $\{\mathsf{pk}_{id}\}_{id \in T}$，输出密文 $\overline{\mathsf{ct}}$。它的参考集是输入密文 $\overline{\mathsf{ct}}_j, 1 \leq j \leq l$ 的联合 $T = T_1 \cup \cdots \cup T_l$。

**语义安全。**对于任意两条消息 $\mu_0, \mu_1 \in \mathcal{M}, \ \ i = 0, 1$ 的分布 $\{\mathsf{MKHE.Enc}(\mu_i; \mathsf{pk}) \}$ 在 $pp \leftarrow \mathsf{MKHE.Setup}(1^\lambda)$ 和 $(\mathsf{sk}, \mathsf{pk}) \leftarrow \mathsf{MKHE.KeyGen}(pp)$ 的计算上应该是不可区分的。

**正确性和紧凑性。**如果与 $k$ 方相关的密文的大小受固定多项式 ${\rm poly}(\cdot, \cdot)$ 的 ${\rm poly}(\lambda, k)$ 的限制，则MKHE方案是紧致的。

对于 $1 \leq j \leq l$，设 $\overline{\mathsf{ct}}_j$ 为密文(参考集合 $T_j$)，使得 $\mathsf{MKHE.Dec}(\overline{\mathsf{ct}}_j; \{\mathsf{sk}_{id}\}_{id \in T} = \mu_j$。设 $\mathcal{C} : \mathcal{M}^l \rightarrow \mathcal{M}$ 为电路，$\overline{\mathsf{ct}} \leftarrow \mathsf{MKHE.Eval}(\mathcal{C}, (\overline{\mathsf{ct}}_1,...,\overline{\mathsf{ct}}_l),\{\mathsf{pk}_{id}\}_{id \in T})$，其中 $T = T_1 \cup \cdots \cup T_l$。然后，
$$
\mathsf{MKHE.Dec}(\overline{\mathsf{ct}}; \{\mathsf{sk}_{id}\}_{id \in T} = \mathcal{C}(\mu_1,...,\mu_l) \tag{1}
$$
以压倒性的可能性。对于近似算术[16]，可以用类似于CKKS格式的近似相等来代替(1)的相等。



## 环上容错学习

在整篇文章中，作者假设 $n$ 是2的幂整数，并且 $R=\mathbb{Z}[X]/(X^n+1)$。对于 $R$ 的模为整数 $q$ 的剩余环，本文记作 $R_q=R/(q·R)$。带参数 $(n,q,\chi,\psi)$ 的环学习假设是给定任意多项式数目的形式的样本 $(a_i,b_i=s \cdot a_i+e_i) \in R^2_q$，其中 $a_i$ 在 $R_q$ 中是一致随机的，$s$ 选自 $R_q$ 上的密钥分布 $\chi$，$e_i$ 取自 $R$ 上的误差分布 $\psi$，$b_i$ 是 $R_q$ 中的均匀随机元素、在计算上是不可区分的。



## 小工具分解

设 $\mathbf{g}=(g_i) \in \mathbb{Z}^d$ 为小工具向量，$q$ 为整数。小工具分解，由 $\mathbf{g}^{-1}$ 表示，是从 $R_q$ 到 $R^d$ 的函数，它将元素 $a \in R_q$ 转换成*小*多项式向量 $u=(u_0,...,u_{d−1})\in R^d$ ，使得 $a = \sum^{d-1}_{i=0} g_i \cdot u_i \pmod{q}$。

小工具分解技术被广泛应用于HE方案的构建中。例如，非线性电路的同态评估是基于密钥切换技术的，并且大多数HE方案利用各种小工具分解方法来控制噪声增长。文献中已经提出了各种分解方法，如位分解[6，7]、基分解[17，21]和基于RNS的分解[4，30]。本文的实现利用了RNS友好的分解来提高效率。



# 本文的构造

## 基本方案

* $\mathsf{Setup}(1^\lambda)$：对于给定的安全参数 $\lambda$，设置RLWE维数 $n$、密文模数 $q$、密钥分布 $\chi$ 和R上的误差分布 $\psi$。生成随机向量 $\mathbf{a} \leftarrow U(R^d_q)$。返回公共参数 $pp=(n, q, \chi, \psi, \mathbf{a})$。
* $\mathsf{KeyGen}(pp)$：选取密钥 $s \leftarrow \chi$。选取误差向量 $e \leftarrow \psi^d$，并将公钥在 $R^d_q$ 中设置为 $\mathbf{b} = -s \cdot \mathbf{a} + \mathbf{e} \pmod{q}$。
* $\mathsf{UniEnc}(\mu;s)$：对于输入明文 $\mu \in R$，生成密文 $\mathbf{D} = [\mathbf{d}_0 | \mathbf{d}_1 | \mathbf{d}_2] \in R^{d \times 3}_q$，如下所示：
  1. 选取 $r \leftarrow \chi$；
  2. 选取 $\mathbf{d}_1 \leftarrow U(R^d_q)$ 和 $\mathbf{e}_1 \leftarrow \psi^d$，然后设 $\mathbf{d}_0 = -s \cdot \mathbf{d}_1 + \mathbf{e}_1 + r \cdot \mathbf{g} \pmod{q}$；
  3. 选取 $\mathbf{e}_2 \leftarrow \psi^d$ 然后设 $\mathbf{d}_2 = r \cdot \mathbf{a} + \mathbf{e}_2 + \mu \cdot \mathbf{g} \pmod{q}$。

公共参数 $pp$ 包含一个随机生成的向量 $\mathbf{a} \in R^d_q$，因此本文假设使用公共引用字符串模型。各方应将相同的公共参数作为密钥生成算法的输入，以支持多密钥同态运算。在以前关于MKHE的所有工作中都做出了同样的假设。

单一加密算法是对称加密，它可以加密单个环形元件。一次加密的密文 $\mathbf{D} = [\mathbf{d}_0 | \mathbf{d}_1 | \mathbf{d}_2] \leftarrow \mathsf{UniEnc}(\mu;s)$ 由 $R^d_q$ 中的三个矢量组成，其大小是 $R^{2d \times 2}_q$ 中普通RGSW密文的(3/4)倍。对于一次加密密文 $\mathbf{D}$，前两列 $[\mathbf{d}_0 | \mathbf{d}_1]$ 可以被视为秘密 $s$ 下的 $r$ 的加密，而 $[\mathbf{d}_2 | -\mathbf{a}]$ 形成秘密 $r$ 下的 $\mu$ 的加密。



## 再线性化

在级联秘密 $\overline{\mathsf{sk}} = (1, s_1, ..., s_k)$ 下加密的两个多密钥密文 $\overline{\mathsf{ct}}_i \in R^{k+1}_q$ 的张量积 $\overline{\mathsf{ct}} = \overline{\mathsf{ct}}_1 \otimes \overline{\mathsf{ct}}_2$ 可以看作是对应于张量平方密钥 $\overline{\mathsf{sk}} \otimes \overline{\mathsf{sk}}$ 的密文。$\overline{\mathsf{sk}} \otimes \overline{\mathsf{sk}}$ 包含一些与两个不同方相关的非线性项 $s_i \cdot s_j$。因此，计算服务器应该能够通过非线性项 $s_i \cdot s_j$ 的线性化来将扩展密文 $\overline{\mathsf{ct}} \in R^{(k+1) \times (k+1)}_q$ 转换为规范密文。

本文的重新线性化方法需要由单个各方生成的相同公共材料(评估密钥)，如下所示：

* $\mathsf{EvkGen}(s)$：给定秘密 $s \in R$，返回 $\mathbf{D} \leftarrow \mathsf{UniEnc}(s;s)$。

准确地说，每一方 $i$ 通过运行算法(si，bi)←KeyGen(Pp)和Di←EvkGen(Si)来生成其自己的秘密密钥、公共密钥和评估密钥，然后发布该对(bi，Di)。在本节的其余部分，我们将介绍两种重新线性化算法，并说明它们的优缺点。