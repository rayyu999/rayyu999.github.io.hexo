---
title: >-
  论文笔记 ASTRA: High Throughput 3PC over Rings with Application to Secure
  Prediction
date: 2020-04-20 21:37:26
categories: Papers
tags: [MPC, Neural Network, PPML, 密码学, SVM, Linear Regression, Logistic Regression, Secret Sharing]
---

*Harsh Chaudhari, Ashish Choudhury, Arpita Patra, Ajith Suresh*

ACM CCSW 2019

https://eprint.iacr.org/2019/429

https://dl.acm.org/doi/10.1145/3338466.3358922

<!--more-->



![](http://images.yingwai.top/picgo/ASTRA.jpg)



## 介绍

### 摘要

安全计算的具体效率一直是近年来许多工作关注的焦点。在本论文中，作者提出了一种具体有效的协议，用于模 $2^l$ 整数环上的安全三方计算（3PC），该协议具有半诚实模型和恶意模型上的安全性。由于环上的计算模拟了现实系统体系结构上的计算，所以环上的安全计算近来获得了发展势头。

​		在离线-在线模式中，本文的结构具体地呈现了最有效的在线阶段。在半诚实的设置下，本文的协议在在线阶段每个乘法门需要2个环元素的通信。在恶意设置下，本文的协议在在线阶段每个乘法门需要4个元素的通信，比最先进的协议需要的5个元素少。使用选择性中止和公平这两个安全概念来实现的公平恶意协议，与仅针对输出门的中止安全性的恶意协议相比，涉及的通信稍微多一些。

​		作者将本文的技术从3PC应用到安全的服务器辅助机器学习（ML）推理机制中，用于一系列预测函数——线性回归、线性SVM回归、Logistic回归和线性SVM分类。本文的设置考虑了拥有训练好的模型参数的模型所有者和查询的客户，后者愿意根据前者的模型参数来学习他查询的预测。输入和计算外包给一组三个非合谋的服务器。本文的构造既迎合了半诚实的世界，也迎合了恶意的世界，比现有的构造表现得更好。




### 贡献

* 作者遵循离线-在线模式，提出了在环 $\mathbb{Z}_{2^l}$（包括布尔环 $\mathbb{Z}_{2^1}$）上的3PC构造，具有最有效的在线阶段。

  虽然重点放在在现阶段，但离线阶段的的成本也有注意并保持在可控范围内。

* 作者给出了一系列满足半诚实安全和恶意安全的构造。

  将技术应用于外包环境中的一系列预测函数的安全预测，并构建了一些容忍半诚实和恶意对手的结构。

本文所有的构建都流露出的一个共同特征：在线阶段不到三对参与者之间需要进行功能依赖的通信，从而产生更好的在线性能。



## 设定

本文考虑一组三方 $\mathcal{P}=\left\{P_0,P_1,P_2\right\}$，它们在同步网络中通过成对的私有和可信信道连接。要计算的函数 $f$ 被表示为环 $\mathbb{Z}_{2^l}$ 上的电路 ckt，该环由2输入加法和乘法门组成。假设 ckt 的拓扑是公知的。术语 D​ 表示 ckt 的乘法深度，而 I、O、A、M 分别表示 ckt 中的输入线、输出线、加法门和乘法门的数目。本文使用符号 $w_x$ 来表示导线 $w$，其中值 $x$ 流经它。本文使用 $g=(w_x,w_y,w_z)$ 来表示 ckt 中具有左输入线 $w_x$、右输入线 $w_y$ 和输出线 $w_z$ 的门。在本文的协议中，将 $\mathcal{P}$ 划分为互不相交的集合 $\left\{P_0\right\}$ 和 $\left\{P_1,P_2\right\}$，其中 $P_0$ 在离线阶段充当“分配器”进行“预处理”，在线阶段“评估者” $P_1$、$P_2$ 用它来评估 ckt。本文使用上标“$s$”和“$m$”分别区分半诚实和恶意设置中的协议。布尔环 $\mathbb{Z}_{2^1}$ 上的协议可以通过将算术加法$(+)$和乘法$(\times)$分别替换为异或$(\oplus)$和与$(\cdot)$来获得。



### 共享密钥设置

为了保存双方之间的通信，使用为伪随机函数（PRF）$F$ 建立预共享随机密钥的一次性设置。在3PC设置[2，30，46]中的已知协议中已经使用了类似的设置。这里 $F:{\{0，1\}}^\kappa \times {\{0，1\}}^\kappa \to X$ 是安全的PRF，同域 $X$ 是 $\mathbb{Z}_{2^l}$。这组密钥是：

* 每对参与方之间共享一个密钥— $k_{01}, k_{02}, k_{12}$，分别用于参与方$(P_0,P_1), (P_0,P_2), (P_1,P_2)$。
* 所有各方之间的一个共享密钥— $k_{\mathcal{p}}$。

本文通过可以使用任何标准安全MPC协议实现的功能 $\mathcal{F}_{\rm setup}$ 来建立密钥设置模型。



### 共享语义

在本节中，将解释本工作中使用的秘密共享的两种变体。这两个变体都在算术（$\mathbb{Z}_{2^l}$）和布尔环（$\mathbb{Z}_{2^1}$）上运行。

$[\cdot]$-共享：如果 $P_1$ 和 $P_2$ 分别持有份额 $v_1$ 和 $v_2$，使得 $v=v_1+v_2$，则称值 $v$ 在$P_1,P_2$之间是 $[\cdot]$-共享的。用 $[\cdot]_{P_i}$ 表示 $P_i, i∈\{1,2\}$的 $[\cdot]$-份额。

$[\![\cdot]\!]$-共享：值 $v$ 在 $P_0, P_1, P_2$ 之间是$[\![\cdot]\!]$-共享的，如果

* 存在值 $\lambda_v, m_v$ 使得 $v=m_v - \lambda_v$；
* $P_0$ 持有 $\lambda_{v,1}$ 和 $\lambda_{v,2}$ 使得 $\lambda_{v} = \lambda_{v,1} + \lambda_{v,2}$；
* $P_1$ 和 $P_2$ 分别持有 $(m_v, \lambda_{v,1})$ 和 $(m_v, \lambda_{v,1})$。

本文将各方的$[\![\cdot]\!]$-共享表示为$[\![v]\!]_{P_0} = (\lambda_{v,1}, \lambda_{v,2}), [\![v]\!]_{P_1} = (m_v, \lambda_{v,1}) $和 $[\![v]\!]_{P_2} = (m_v, \lambda_{v,2})$。用$[\![v]\!] = (m_v, [\lambda_{v}])$表示 $v$ 的$[\![\cdot]\!]$-共享份额。



### 秘密共享方案的线性

给定 $x,y \in \mathbb{Z}_{2^l}$ 和公共常数 $c_1, c_2 \in \mathbb{Z}_{2^l}$的$[\cdot]$-共享，各方可以局部计算 $[c_1x+c_2y]$：

$$[c_1x+c_2y]=(c_1x_1+c_2y_1, c_1x_2+c_2y_2)=c_1[x]+c_2[y]$$

很容易看出线性关系也扩展到$[\![\cdot]\!]$-共享。线性属性使各方能够**本地**执行与公共常量的加法和乘法等操作。



## 3PC协议

### 半诚实下的3PC

协议 $\prod ^{\rm s}_{\rm 3pc}$ 由三个步骤组成—输入共享、电路评估以及输出重构。所有阶段（重构输出除外）都分为离线和在线阶段，其中独立于实际输入的步骤可以在脱机阶段执行。



#### 输入共享

在共享输入阶段，每一方都为自己的输入生成一个随机的$[\![\cdot]\!]$-共享。以下是初始共享协议 $\prod^{\rm s}_{\rm Sh}$：

-----

**Offline:**

- If $P_i = P_0$, parties $P_0, P_j$ for $j \in \{1, 2\}$ locally sample a random $\lambda_{x,j} \in \mathbb{Z}_{2^l}$. Moreover, $P_i$ sets $[\![x]\!]_{P_i} = (\lambda_{x,1}, \lambda_{x,2})$.
- If $P_i = P_1$, parties $P_0, P_1$ sample a random $\lambda_{x,1} \in \mathbb{Z}_{2^l}$ while all the parties in $\mathcal{P}$ sample a random $\lambda_{x,2} \in \mathbb{Z}_{2^l}$.
- If $P_i = P_2$, parties $P_0, P_2$ sample a random $\lambda_{x,2} \in \mathbb{Z}_{2^l}$ while all the parties in $\mathcal{P}$ sample a random $\lambda_{x,1} \in \mathbb{Z}_{2^l}$.

**Online:**

​		$P_i$ computes $\lambda_x = \lambda_{x,1} + \lambda_{x,2}$ and sends $m_x = x + \lambda_x$ to every $P_j$ for $j \in \{1,2\}$ who then sets $[\![x]\!]_{P_j} = (m_x, \lambda_{x,j})$.

-----

在离线阶段根据输入方的序号，使用不同的初始化手段，生成了 $P_0$ 的份额 $[\![x]\!]_{P_0}=(\lambda_{x,1}, \lambda_{x,2})$，而不用任何交互（因为随机数都是使用服务器两两之间的公共随机源生成的，可以理解为编程语言中把同样的随机种子输入到random函数——只要随机种子一样生成的随机数序列就一样）。



#### 电路评估

在电路评估阶段，各方以$[\![\cdot]\!]$-共享的方式评估 ckt。以拓扑顺序评估 ckt 中的每个门 $g$：给定 $g$ 的输入，各方为 $g$ 的输出生成$[\![\cdot]\!]$-共享。

##### 加法

如果 $g$ 是加法门 $(w_x,w_y,w_z)$，可以利用$[\![\cdot]\!]$-共享的线性在本地完成。以下是加法协议 $\prod_{\rm Add} (w_x, w_y, w_z)$：

-----

**Offline:**

​		$P_0, P_1$ set $\lambda_{z,1} = \lambda_{x,1} + \lambda_{y,1}$, while $P_0, P_2$ set $\lambda_{z,2} = \lambda_{x,2} + \lambda_{y,2}$.

**Online:**

​		$P_1$ and $P_2$ set $m_z = m_x + m_y$.

-----

这里的加法协议感觉在线阶段完全可以在本地完成，可以不需要在线阶段。

##### 乘法

如果 $g=(w_x,w_y,w_z)$ 是乘法门，则运行以下乘法协议 $\prod ^{\rm s}_{\rm Mul}$：

-----

**Offline:**

* $P_0$ and $P_1$ locally sample random $\lambda_{z,1}, \gamma_{xy, 1} \in \mathbb{Z}_{2^l}$, while $P_0$ and $P_2$ locally sample random $\lambda_{z,2} \in \mathbb{Z}_{2^l}$.
* $P_0$ computes $\gamma_{xy} = \lambda_x \lambda_y$ and sends $\gamma_{xy,2} = \gamma_{xy} - \gamma_{xy,1}$ to $P_2$.

**Online:**

* $P_i$ for $i \in \{1,2\}$ locally computes $[m_z]_{P_i} = (i-1)m_xm_y - m_x[\lambda_y]_{P_i} - m_y[\lambda_x]_{P_i} + [\lambda_z]_{P_i} + [\gamma_{xy}]_{P_i}$.
* $P_1, P_2$ mutually exchange their shares and reconstruct $m_z$.

-----

将上面的公式相加一下即可验证其正确性。



#### 输出重构

在输出重构阶段，各方重构$[\![\cdot]\!]$-共享电路输出。为了重构 $[\![y]\!]$，可以观察到每个 $P_i$ 缺失的份额都在另外两方手里。因此另外两方的其中一方将缺失的份额发送给 $P_i$ 后，通过计算 $y = m_y - \lambda_{y,1} - \lambda_{y,2}$ 即可重构输出 $y$，将其称为重构协议 $\prod ^{\rm s}_{\rm Rec}([\![y]\!], \mathcal{P})$。

把上面三个阶段总结起来，有以下的协议 $\prod ^{\rm s}_{\rm 3pc}$：

-----

**Pre-processing (Offline Phase):**

* *Input wires:* For $j = 1, ...,l$, corresponding to the circuit-input $x_j$, parties execute the offline steps of the instance $\prod ^{\rm s}_{\rm Sh}(P_i, x_j)$.
* For each gate $g$ in the topological order, execute offline steps of the instance $\prod ^{\rm s}_{\rm Mul}(w_{xj}, w_{yj}, w_{zj})$ if $g$ is the $j$th multiplication gate where $j \in \{1,...,\rm M\}$ or respectively offline steps of the instance $\prod _{\rm Add}(w_{xj}, w_{yj}, w_{zj})$ if $g$ is the $j$th addition gate where $j \in \{1,...,\rm A\}$.

**Circuit Evaluation (Online Phase):**

* *Sharing Circuit-input Values:* For $j = 1, ...,l$, corresponding to the circuit-input $x_j$, parties execute the online steps of the instance $\prod ^{\rm s}_{\rm Sh}(P_i, x_j)$, where $P_i$ is the party designated to provide $x_j$.
* *Gate Evaluation:* For each gate in $g$ in ckt in the topological order, $P_1, P_2$ execute the online steps of the instance $\prod ^{\rm s}_{\rm Mul}(w_{xj},w_{yj},w_{zj})$ if $g$ is the $j$th multiplication gate where $j \in \{1,...,\rm M\}$ or respectively offline steps of the instance $\prod _{\rm Add}(w_{xj}, w_{yj}, w_{zj})$ if $g$ is the $j$th addition gate where $j \in \{1,...,\rm A\}$.
* *Output Reconstruction:* Let $[\![y_1]\!],...,[\![y_\rm O]\!]$ be the shared function outputs. The parties in $\mathcal{P}$ reconstruct $y_j$ for $j = 1,...,\rm O$ by executing $\prod ^{\rm s}_{\rm Rec}([\![y_j]\!], \mathcal{P})$.

-----



### 恶意下的3PC

跟半诚实设置下的一样，恶意设置下的3PC协议 $\prod ^{\rm m}_{\rm 3pc}$ 也由输入共享、电路评估和输出重构三个部分组成。



#### 输入共享和输出重构

在恶意的设置下，要保证服务器之间的共享份额是一致的。在 $\prod ^{\rm s}_{\rm Sh}$ 中，$\lambda$ 的共享是一致的，因为它不需要交互就可以生成。但如果一个腐败的 $P_0$ 拥有 $x$ 并且想制造一个不一致的 $[\![x]\!]$-共享，它可以分别发送两个不一样的 $m_x$ 给 $P_1$ 和 $P_2$。为了检查这种情况的发生，$P_1$ 和 $P_2$ 交换 $H(m_x)$ 并在不一致的时候中止。

令 $[\![y]\!]$ 为一个待重构的一致的共享，$[\![y]\!]_{P_0} = (\lambda_{y,1}, \lambda_{y,2})$、$[\![y]\!]_{P_1} = (m'_{y}, \lambda_{y,1})$ 和 $[\![y]\!]_{P_2} = (m''_{y}, \lambda_{y,2})$ 分别为三个服务器的份额。协议 $\prod ^{\rm m}_{\rm Rec}([\![y]\!], \mathcal{P})$ 允许每一个诚实方输出 $y$ 或 $\perp$：

-----

**Online:**

* $P_0$ and $P_2$ send $\lambda_{y,2}$ and $H(\lambda'_{y,2})$ respectively to $P_1$.
* $P_0$ and $P_1$ send $\lambda_{y,1}$ and $H(\lambda'_{y,1})$ respectively to $P_2$.
* $P_1$ and $P_1$ send $m'_{y}$ and $H(m'_{y})$ respectively to $P_0$.

$P_i$ for $i \in \{0,1,2\}$ abort if the received values mismatch. Else $P_i$ sets $y = m_y - \lambda_{y,1} - \lambda_{y,2}$.

-----

检查的过程中，其中一方发送哈希值可以提高效率。



#### 电路评估

在恶意设置下加法协议 $\prod _{\rm Add}$ 同样是安全的，因为它只涉及本地操作。挑战在于构造乘法协议 $\prod ^{\rm m}_{\rm Mul}$，令其可以容忍其中一方腐败。可以观察到有两种情况：

* $P_0$ 是腐败的：在离线阶段会令 $\gamma_{xy} \neq \lambda_x \lambda_y$
* $P_1$ 或 $P_2$ 是腐败的：会在在线阶段扰乱，使诚实的另一方重构出一个错误的 $m_z$

先看下面的情况，假如 $P_1$ 现在需要验证它重构的 $m_z$ 是否是正确的，可以向 $P_0$ 求助：$P_1$ 可以发送 $m_x, m_y$ 给 $P_0$，因为 $P_0$ 在离线阶段就已经知道 $\lambda_x, \lambda_y$ 和 $\lambda_z$，因此它可以计算出 $m_z$ 并发送给 $P_1$，以此得到验证。但发送 $m_x, m_y$ 给 $P_0$ 会导致打破了原有共享的私密性，所以 $P_1$ 可以对应的值盲化后发送给 $P_0$：$m^{\star}_x = m_x + \delta_x$ 和 $m^{\star}_y = m_y + \delta_y$，然后 $P_0$ 计算 $m^{\star}_z = -m^{\star}_x \lambda_y - m^{\star}_y \lambda_x +\lambda_z + 2\gamma_{xy}$。注意到：
$$
\begin{align}
m^{\star}_z &= -m^{\star}_x \lambda_y - m^{\star}_y \lambda_x +\lambda_z + 2\gamma_{xy}\\
 &= -(m_x + \delta_x) \lambda_y - (m_y + \delta_y) \lambda_x +\lambda_z + 2\gamma_{xy}\\
 &= (m_z - m_x m_y) - \chi
\end{align}
$$

假设 $P_0$ 知道 $\chi = \delta_x \lambda_y + \delta_y \lambda_x - \gamma_{xy}$，它就可以计算出 $m^{\star}_z + \chi$ 然后发送给 $P_1$。因为 $P_1$ 知道 $m_x, m_y$ 的值，因此它可以验证它重构的 $m_z$ 的正确性，对于 $P_2$ 来说也可以这样验证。

现在来描述如何使 $P_0$ 获得 $\chi$：首先直接让 $P_0$ 获得 $\chi$ 会导致共享的私密性被破坏，因为 $P_0$ 知道 $\lambda_x, \lambda_y$ 和 $\gamma_{xy}$ 的值，同时在在线阶段又得到 $m_x + \delta_x$ 和 $m_y + \delta_y$，导致 $P_0$ 可以推导出 $m_x, m_y$ 之间的关系。所以还要在 $\chi$ 中加入一个随机值 $\delta_z$ 进行盲化：$\delta_x \lambda_y + \delta_y \lambda_x + \delta_z - \gamma_{xy}$。

生成 $\chi$ 的过程：$P_1, P_2$ 在本地生成随机数 $\delta_x, \delta_y, \delta_z \in \mathbb{Z}_{2^l}$，计算各自的 $\chi$ 的 $[\cdot]$-共享并发送给 $P_0$。对于 $i \in \{1,2 \}$，令 $[\chi]_{P_i} = \chi_i$。$P_0$ 在本地将共享的份额相加得到 $\chi$。在以上步骤中，腐败的一方可能会在执行过程中引入错误，使得 $P_0$ 获得的 $\chi$ 是错误的。

总而言之在离线阶段有两个问题需要解决：

* 腐败的 $P_0$ 可以不正确地共享 $\gamma_{xy}$
* 腐败的 $P_1$ 或 $P_2$ 可以发送错误的 $\chi$ 的 $[\cdot]$-共享给 $P_0$

为了解决这些问题，$P_0$ 一旦获得 $\chi$，就用下面的方式计算 $a = \delta_x - \lambda_x$、$b = \delta_y - \lambda_y$ 和 $c = (\delta_z + \delta_x \delta_y) - \chi$ 的 $[\![\cdot]\!]$-共享：
$$
\begin{align*}
[\![a]\!]_{P_0} &= (\lambda_{x,1}, \lambda_{x,2}), & [\![b]\!]_{P_0} &= (\lambda_{y,1}, \lambda_{y,2}), & [\![c]\!]_{P_0} &= (\chi_{1}, \chi_{2})\\
 [\![a]\!]_{P_1} &= (\delta_x, \lambda_{x,1}), & [\![b]\!]_{P_1} &= (\delta_y, \lambda_{y,1}), & [\![c]\!]_{P_1} &= (\delta_z + \delta_x \delta_y, \chi_{1})\\
 [\![a]\!]_{P_2} &= (\delta_x, \lambda_{x,2}), & [\![b]\!]_{P_2} &= (\delta_y, \lambda_{y,2}), & [\![c]\!]_{P_2} &= (\delta_z + \delta_x \delta_y, \chi_{2})
\end{align*}
$$
现在 $([\![a]\!], [\![b]\!], [\![c]\!])$ 是一个乘法三元组 $(c=ab)$，当且仅当 $P_0$ 正确分享了 $\gamma_{xy}$（当它腐败时）以及重构的 $\chi$ 是正确的（当 $P_1,P_2$ 其中之一腐败时），因为：
$$
\begin{align}
ab &= (\delta_x - \lambda_x)(\delta_y - \lambda_y) = \delta_x \delta_y + \lambda_x \lambda_y - \delta_x \lambda_y - \delta_y \lambda_x \\
 &= (\delta_x \delta_y + \delta_z) - (\delta_x \lambda_y + \delta_y \lambda_x + \delta_z - \gamma_{xy}) \\
  &= (\delta_x \delta_y + \delta_z) - \chi = c
\end{align}
$$
然后作者给出了一个检查乘法三元组是否正确的协议，这里需要用到另一个有效的乘法三元组 $([\![d]\!], [\![e]\!], [\![f]\!])$，它们满足以下条件：

* $d, e, f$ 都是随机且私密的
* $f = d e$

这里假设这个三元组是通过 $\mathcal{F}_{\rm trip}$ 生成的，在[2, 30]中有具体构造，下面是[30]中的构造：

![](http://images.yingwai.top/picgo/ASTRAf1.png)

用  $\prod _{\rm trip}$ 表示该功能的实例化，下面是作者给出检查有效性的协议 $\prod _{\rm prc}$：

-----

* Parties locally compute $[\![\rho]\!] = [\![a]\!] - [\![d]\!]$ and $[\![\sigma]\!] = [\![b]\!] - [\![e]\!]$.
* Parties reconstruct $\rho$ and $\sigma$ by executing $\prod ^{\rm m}_{\rm Rec}([\![\rho]\!], \mathcal{P})$ and $\prod ^{\rm m}_{\rm Rec}([\![\sigma]\!], \mathcal{P})$ respectively.
* Parties locally compute $[\![\tau]\!] = [\![c]\!] - [\![f]\!] - \sigma [\![d]\!] - \rho [\![e]\!] - \sigma \rho$.
* Parties reconstruct $\tau$ by executing $\prod ^{\rm m}_{\rm Rec}([\![\tau]\!], \mathcal{P})$ and output $\perp$, if $\tau \neq 0$.

-----

协议 $\prod _{\rm prc}$ 需要两对秘密共享三元组 $(a, b, c)$ 和 $(d, e, f)$，验证前一个三元组是否满足 $c = a b$：
$$
\begin{align}
\tau &= c - f - \sigma d - \rho e - \sigma \rho \\
 &= c - de - (b - e)d - (a - d)e - (b - e)(a - d) \\
 &= c - ab = \Delta
\end{align}
$$
所以 $\tau = 0$ 时 $(a,b,c)$ 有效，反之无效。而检验 $\tau$ 是否为0只需要一方跟另外两方各通信一次即可：(1) $P_0$ 跟 $P_1$ 检查 $m_\tau - \lambda_{\tau,1}$ 跟 $\lambda_{\tau, 2}$ 是否相等；(2) $P_1$ 跟 $P_2$ 检查 $m_\tau - \lambda_{\tau,2}$ 跟 $\lambda_{\tau, 1}$ 是否相等；(3) $P_0$ 跟 $P_1$ 检查 $m_\tau - \lambda_{\tau,2}$ 跟 $\lambda_{\tau, 1}$ 是否相等，而不是 $\prod ^{\rm m}_{\rm Rec}$ 的通信三次（两次发送一次接收）。下面给出恶意设置下的乘法协议 $\prod ^{\rm m}_{\rm Mul}(w_x, w_y, w_z)$：

-----

**Offline:**

* Parties $P_0, P_1$ locally sample random $\lambda_{z,1}, \gamma_{xy,1} \in \mathbb{Z}_{2^l}$, while $P_0, P_2$ 
  locally sample a random $\lambda_{z,2}$. $P_0$ locally computes $\gamma_{xy}= \lambda_x \lambda_y$ and
  sends $\gamma_{xy,2} = \gamma_{xy} - \gamma_{xy,1}$ to $P_2$.
* Parties execute $\prod _{\rm trip}$ to generate a triple $([\![d]\!], [\![e]\!], [\![f]\!])$.
* Parties $P_1, P_2$ locally sample random $\delta_x, \delta_y, \delta_z \in \mathbb{Z}_{2^l}$ and compute
  $[\delta_z]$ non-interactively.
* $P_i$ for $i \in \{1, 2\}$ computes $[\chi]_{P_i} = \delta_x[\lambda_y]_{P_i} + \delta_y[\lambda_x]_{P_i} + [\delta_z]_{P_i} − [\gamma_{xy}]_{P_i}$ and sends $[\chi]_{P_i}$ to $P_0$, who computes $\chi$.
* Parties locally compute the $[\![\cdot]\!]$-shares of the values $a = \delta_x - \lambda_x$, $b = \delta_y - \lambda_y$ and $b = (\delta_z + \delta_x \delta_y) - \chi$, as described in the text.
* Parties execute $\prod _{\rm prc}$ on $([\![a]\!], [\![b]\!], [\![c]\!])$ and $([\![d]\!], [\![e]\!], [\![f]\!])$.

**Online:**

* $P_i$ for $i \in \{1, 2\}$ locally computes $[m_z]_{P_i} = (i-1)m_xm_y - m_x[\lambda_y]_{P_i} - m_y[\lambda_x]_{P_i} + [\lambda_z]_{P_i} + [\gamma_{xy}]_{P_i}$. $P_1, P_2$ mutually exchange their shares and reconstruct $m_z$.
* $P_1$ sends $m^{\star}_x = m_x + \delta_x$, $m^{\star}_y = m_y + \delta_y$ to $P_0$, while $P_2$ sends $H(m^{\star}_x || m^{\star}_{y})$ to $P_0$.$P_0$ outputs $\perp$, if the received values are inconsistent.
* $P_0$ computes $m^{\star}_z = -m^{\star}_x \lambda_y - m^{\star}_y \lambda_x +\lambda_z + 2\gamma_{xy} + \chi$ and sends $H(m^\star_z)$ to both $P_1$ and $P_2$.
* $P_i$ for $i \in \{1, 2\}$ abort if $H(m^\star_z) \neq H(m_z - m_x m_y + \delta_z)$.

-----

散列值的使用提高了效率，减少了通信的开销。

对于正确性，首先考虑腐败的 $P_0$ 不正确地共享使得 $\gamma_{xy} = \lambda_x \lambda_y + \Delta$ 的情况，其中 $\Delta$ 是 $P_0$ 引入的不为零的干扰。这种情况在执行 $\prod _{\rm prc}$ 就会被检测到，因为最终计算出来并重构的 $\tau \neq 0$。同样的， $P_1$（或 $P_2$）在协议 $\prod ^{\rm m}_{\rm Mul}$ 离线阶段的第四步时发送 $\chi_1 + \Delta$（或 $\chi_2 + \Delta$） 给 $P_0$ 使其重构的 $\chi' = \chi + \Delta$，也会导致 $\tau \neq 0$ 从而被诚实方发现并中止计算。

然后是另一种情况，假如 $P_1$（或 $P_2$）在重构 $m_z$ 时发送了错误的 $[m_z]_{P_i}$ 给另一方，会在最后一步验证哈希值是否相等的时候被检测出来；而在在线阶段第二步中 $P_0$ 对 $m^{\star}_x,m^{\star}_y$ 的一致性检查也确保了它所计算出来的 $m^{\star}_z$ 是正确的。



#### 公平的实现

作者通过一种公平重构协议 $\prod_{\rm fRec}$ 来重构电路输出，将 $\prod ^{\rm m}_{\rm 3pc}$ 的安全性提高到公平，保证交易的三方都不能通过损害别人的利益而得到自己不应得的利益。这里使用到了承诺方案，还是利用了服务器之间的公共随机源并使用PRF来为承诺方案引入随机性：

-----

**Offline:**

* Parties $P_0, P_1$ locally sample a random $r_1 \in \mathbb{Z}_{2^l}$, prepare and send commitments of $\lambda_{y,1}$ and $r_1$ to $P_2$. Similarly, parties $P_0, P_2$ locally sample a random $r_2 \in \mathbb{Z}_{2^l}$, prepare and send commitments of $\lambda_{y,2}$ and $r_2$ to $P_1$. The randomness needed for both commitments are sampled from the PRF key-setup.
* $P_1$ (resp. $P_2$) aborts if the received commitments mismatch.

**Online:**

* $P_1, P_2$ compute a commitment of $m_y$ using randomness sampled from their PRF key-setup and send it to $P_0$.
* If the commitments do not match, $P_0$ sends (**abort**, $o_1$) to $P_2$, while it sends (**abort**, $o_2$) to $P_1$ and aborts, where $o_i$ denotes the opening information for the commitment of $r_i$. Else $P_0$ sends **continue** to both $P_1$ and $P_2$.
* $P_1, P_2$ exchange the messages received from $P_0$.
* $P_1$ aborts if it receives either (i) (**abort**, $o_2$) from $P_0$ and $o_2$ opens the commitment of $r_2$ or (ii) (**abort**, $o_1$) from $P_2$ and $o_1$ is the correct opening information of $r_1$. The case for $P_2$ is similar to that of $P_1$.
* If no abort happens, parties obtain their missing share of $a$ as follows:
  * $P_0, P_1$ open $\lambda_{y,1}$ towards $P_2$.
  * $P_0, P_2$ open $\lambda_{y,2}$ towards $P_1$.
  * $P_1, P_2$ open $m_y$ towards $P_0$.
* Parties reconstruct the value $y$ using missing share that matches with the agreed upon commitment.

-----

当没有广播频道的时候，一个非常棘手的问题就会存在：一个腐败的 $P_0$ 可以发送不同的信号给 $P_1$ 和 $P_2$（一个为 abort 而另一个为 continue），以上的重构协议 $\prod_{\rm fRec}([\![y]\!], \mathcal{P})$ 解决了这个问题。在离线阶段 $P_0$ 和 $P_1$ 共同计算出一个关于 $r_1$ 的承诺发送给 $P_2$，同样地 $P_0$ 和 $P_2$ 共同计算出一个关于 $r_2$ 的承诺发送给 $P_1$。这两个承诺就可以确保 $P_1$ 跟 $P_2$ 可以验证它们收到的来自 $P_0$ 的信号是否一致：例如当 $P_1$ 收到 abort 而 $P_2$ 收到 continue，在它们交换信息时，$P_1$ 就可以利用收到的 $o_2$ 证明自己收到了 abort 信号，反过来也是一样的。同时还解决了一个问题，就是当一个腐败的 $P_1$ 收到了 $P_0$ 发送的 continue 信号，但它不能在与 $P_2$ 交换信息时宣称自己收到了 abort 信号，因为它没有 $o_2$，因此无法证明，对于 $P_2$ 腐败的情况也是一样的。

这里的承诺方案可以通过一个哈希函数来实现，例如：$(c, o) = (\mathcal{H}(x||r),x||r) = Com(x;r)$



## 隐私保护机器学习

### 模型

对于每一个预测函数，模型拥有者 $\rm M$ 拥有一个训练好的参数向量，想为拥有一个查询向量的客户 $\rm C$ 提供预测服务。在服务器辅助设置中，$\rm M$ 和 $\rm C$ 以共享方式将各自的输入外包给三个不受信任但非合谋的服务器 $\{P_0, P_1, P_2\}$，这些服务器通过为本文的3PC协议开发的技术以共享方式执行计算，并将输出单独重构到客户端。客户只能知道输出，除此之外什么也不知道。



### 对于ML的协议

#### 安全向量点积

对于向量的 $[\cdot]$-共享和 $[\![\cdot]\!]$-共享，就是对应每个维度的值进行$[\cdot]$-共享和 $[\![\cdot]\!]$-共享，容易知道对于向量来说两种共享仍然是线性的。对于两个 $d$ 维向量的点积，不考虑效率的情况下可以执行 $d$ 次 $\prod ^{\rm s}_{\rm Mul}$协议，再对这 $d$ 次执行的结果简单进行相加，各方就可以得到它们的份额。在这里作者给出一个更高效率的向量点积协议 $\prod ^{\rm s}_{\rm dp}$：

-----

**Offline:**

​		$P_0, P_1$ sample random $\lambda_{u,1}, \gamma_{pq,1} \in \mathbb{Z}_{2^l}$, while $P_0, P_2$ sample random $\lambda_{u,2} \in \mathbb{Z}_{2^l}$. $P_0$ locally computes $\gamma_{pq} = \vec{\lambda_p} \odot \vec{\lambda_q}$, sets $\gamma_{pq,2} = \gamma_{pq} - \gamma_{pq,1}$ and sends $\gamma_{pq,2}$ to $P_2$.

**Online:**

* $P_i$ for $i \in \{1,2\}$ locally computes $[m_u]_{P_i} = \sum^d_{j=1}((i-1)m_{p_j}m_{q_j} - m_{p_j}[\lambda_{q_j}]_{P_i} - m_{q_j}[\lambda_{p_j}]_{P_i}) + [\gamma_{pq}]_{P_i} + [\lambda_u]_{P_i}$.
* $P_1$ and $P_2$ mutually exchange their share of $[m_u]$ to reconstruct $m_u$.

-----

上面的协议的离线阶段中，$P_0$ 仅仅共享了 $\gamma_{pq} = \vec{\lambda_p} \odot \vec{\lambda_q}$ 而不是每一个 $\lambda_{p_i} \lambda_{q_i}$；在在线阶段，$P_1, P_2$ 直接计算 $[m_u]$（其中 $u = \vec{p} \odot \vec{q}$）而不是每一个 $m_{p_i q_i}$。



接下来作者还对恶意设置下的点积进行了讨论：由于在乘法协议中引入了对恶意对手的额外检查，所以上面针对半诚实协议所作的优化是不适用的。对两个 $d$ 维向量的点积，只能 $d$ 次调用协议 $\prod ^{\rm m}_{\rm Mul}$。不过作者还是对在线阶段的开销进行了改进：在在线阶段 $P_1$ 并行地发送 $m^{\star}_{p_i}, m^{\star}_{q_i}$ 给 $P_0$，而 $P_2$ 则发送对应的哈希值给 $P_0$。$P_0$ 收到这些值后进行验证，若一致则将它们“结合”所有的 $m^{\star}_{p_i q_i}$ 然后发送一个单独的 $m^\star_u$ 的哈希值给 $P_1, P_2$，最后 $P_1, P_2$ 在本地验证是否与 $m_u - \sum^d_{j=1}(m_{p_j}m_{q_j} - \delta_{u_j})$。这样做的话就节省了在线阶段的开销，不用每个 $m^{\star}_{p_i}, m^{\star}_{q_i}$ 都发送一次。



#### 安全比较

给定算术共享 $[\![u]\!], [\![v]\!]$，各方希望验证 $u$ 是否小于 $v$，等同于验证 $a$ 是否小于 $0$（其中 $a = u - v$），在定点表示中可以通过检查 ${\rm msb}(a)$ 来完成（二进制补码中第一位为符号位）。于是可以把在给定算术共享 $[\![a]\!]$ 的情况下生成 ${\rm msb}(a)$ 的布尔共享作为目标，在这里作者利用了秘密共享方案中的不对称性，放弃了 *SecureML*[48]和 *ABY3*[46]中的昂贵协议。

-----

**Offline:**

​		$P_1, P_2$ together sample a random $r, r' \in \mathbb{Z}_{2^l}$ and set $p= \rm{msb}$$(r)$. Parties non-interactively generate Boolean share of $p$ as $[\![p]\!]^{\rm B}_{P_0} = (0,0)$, $[\![p]\!]^{\rm B}_{P_1} = (p,0)$ and $[\![p]\!]^{\rm B}_{P_2} = (p,0)$.

**Online:**

​		$P_1$ set $[a]_{P_1}=m_a - \lambda_{a,1}$, $P_2$ set $[a]_{P_2}=- \lambda_{a,2}$.

* $P_1$ sends $[ra]_{P_1} = r[a]_{P_1} + r'$ to $P_0$, while $P_2$ sends $[ra]_{P_2} = r[a]_{P_1} - r'$ to $P_0$, who adds them to obtain $ra$.
* $P_0$ executes $\prod^{\rm s}_{\rm Sh}(P_0, q)$ over $\mathbb{Z}_{2^1}$ to generate $[\![q]\!]^{\rm B}$ where $q = {\rm msb}(ra)$.
* Parties locally compute $[\![\mbox{msb}(a)]\!]^{\rm B} = [\![p]\!]^{\rm B} \oplus [\![q]\!]^{\rm B}$.

-----

上面的协议用 $\prod ^{\rm s}_{\rm BitExt}([\![a]\!], \mathcal{P})$ 表示。这里上面用到了一个随机数 $r$ 来对 $a$ 的值进行盲化，并且可以注意到 ${\rm sign}(a \cdot r) = {\rm sign}(a) \oplus {\rm sign}(r)$，所以 $r$ 不会对生成共享份额造成影响且使得三者都不能从这个过程中知道关于 $a$ 的信息。

对于恶意的情况，就不能仅仅依靠 $P_0$ 来生成 $[\![{\rm msb}(ra)]\!]^{\rm B}$，下面给出了修改后的协议 $\prod ^{\rm m}_{\rm BitExt}([\![a]\!], \mathcal{P})$：

-----

**Offline:**

​		$P_1, P_2$ sample a random $r_1 \in \mathbb{Z}_{2^l}$ and set $p_1 = {\rm msb}(r_1)$ while $P_0, P_2$ sample a random $r_2 \in \mathbb{Z}_{2^l}$ and set $p_2 = {\rm msb}(r_2)$.

* Parties non-interactively generate $[\![\cdot]\!]$-shares of $r_1$ as $[\![r_1]\!]_{P_0}=(0,0)$, $[\![r_1]\!]_{P_1}=(r_1,0)$ and $[\![r_1]\!]_{P_2}=(r_1,0)$.
* Parties non-interactively generate $[\![\cdot]\!]$-shares of $r_1$ as $[\![r_2]\!]_{P_0}=(0,-r_2)$, $[\![r_1]\!]_{P_1}=(0,0)$ and $[\![r_1]\!]_{P_2}=(0,-r_2)$.
* Parties execute $\prod^{\rm m}_{\rm Mul}$ on $r_1$ and $r_2$ to generate $[\![r]\!] = [\![r_1 r_2]\!]$.
* Parties non-interactively generate Boolean shares of $p_1$ as $[\![p_1]\!]^{\rm B}_{P_0}=(0,0)$, $[\![p_1]\!]^{\rm B}_{P_1}=(p_1,0)$ and $[\![p_1]\!]^{\rm B}_{P_2}=(p_1,0)$.
* Parties non-interactively generate Boolean shares of $p_2$ as $[\![p_2]\!]^{\rm B}_{P_0}=(0,p_2)$, $[\![p_2]\!]^{\rm B}_{P_1}=(0,0)$ and $[\![p_2]\!]^{\rm B}_{P_2}=(0,p_2)$.
* Parties locally compute $[\![p]\!]^{\rm B}=[\![p_1]\!]^{\rm B} \oplus [\![p_2]\!]^{\rm B}$.

**Online:**

* Parties execute $\prod ^{\rm m}_{\rm Mul}$ on $[\![r]\!]$ and $[\![a]\!]$ to generate $[\![ra]\!]$ followed by enabling $P_0, P_1$ to reconstruct $ra$ (this is done by slightly modifying the protocol $\prod ^{\rm m}_{\rm Rec}$ ).
* $P_1$ executes $\prod ^{\rm m}_{\rm Sh}(P_1, q)$ over $\mathbb{Z}_{2^1}$ to generate $[\![q]\!]^{\rm B}$ where $q = {\rm msb}(ra)$. In parallel, $P_0$ locally computes $m_q$ and sends ${\rm H}(m_q)$ to $P_2$, who abort if the value mismatches with the hash of the value $m_q$ received from $P_1$ as part of $\prod ^{\rm m}_{\rm Sh}(P_1, q)$.
* Parties locally compute $[\![{\rm msb}(a)]\!]^{\rm B} = [\![p]\!]^{\rm B} \oplus [\![q]\!]^{\rm B}$.

-----



#### ML预测函数

* **线性回归**：$\rm M$ 有一个 $d$ 维的模型参数向量 $\vec{w}$ 和偏置项 $b$，$\rm C$ 有一个 $d$ 维的查询向量 $\vec{z}$。$\rm C$ 获得 $f_{\rm linr}((\vec{w}, b),\vec{z}) = \vec{w} \odot \vec{z} + b$，其中 $\vec{w} \odot \vec{z}$ 是向量 $\vec{w}$ 和向量 $\vec{z}$ 的点积；
* **SVM回归**：$\rm M$ 有$\{\alpha_j, y_j \}^k_{j=1}$ 和 $d$ 维的支持向量 $\{\vec{x_j}\}^k_{j=1}$，$\rm C$ 有一个 $d$ 维的查询向量 $\vec{z}$。$\rm C$ 获得 $f_{\rm svmr}((\{\alpha_j, y_j, \vec{x_j} \}^k_{j=1}, b), \vec{z}) = \sum^k_{j=1} \alpha_j y_j (\vec{x_j} \odot \vec{z}) + b$；
* **逻辑回归**：$\rm M$ 和 $\rm C$ 的输入和线性回归类似，$\rm M$ 还需要提供一个在 $[0,1]$ 范围内的额外输入 $t$。$\rm C$ 获得 $f_{\rm logr}((\vec{w}, b, t), \vec{z}) = {\rm sign}((\vec{w} \odot \vec{z} + b) - {\rm ln}(\frac{t}{1-t}))$，其中 ${\rm sign}(\cdot)$ 返回对象的符号位；
* **SVM分类**：$\rm M$ 和 $\rm C$ 的输入和SVM回归一样，但对 $\rm C$ 的输出变为 $f_{\rm svmr}((\{\alpha_j, y_j, \vec{x_j} \}^k_{j=1}, b), \vec{z}) = {\rm sign}(\sum^k_{j=1} \alpha_j y_j (\vec{x_j} \odot \vec{z}) + b)$。



## 参考文献

[1] V. A. Abril, P. Maene, N. Mertens, and N. P. Smart. 2019. *Bristol Fashion MPC Circuits.* https://homes.esat.kuleuven.be/~nsmart/MPC/.
[2] T. Araki, A. Barak, J. Furukawa, T. Lichter, Y. Lindell, A. Nof, K. Ohara, A. Watzman, and O. Weinstein. 2017. *Optimized Honest-Majority MPC for Malicious Adversaries - Breaking the 1 Billion-Gate Per Second Barrier.* In IEEE S&P. 843–862.
[3] T. Araki, A. Barak, J. Furukawa, Y. Lindell, A. Nof, and K. Ohara. 2016. *DEMO: High-Throughput Secure Three-Party Computation of Kerberos Ticket Generation.* In ACM CCS. 1841–1843.
[4] T. Araki, J. Furukawa, Y. Lindell, A. Nof, and K. Ohara. 2016. *High-Throughput Semi-Honest Secure Three-Party Computation with an Honest Majority.* In ACM CCS. 805–817.
[5] C. Baum, I. Damgård, T. Toft, and R. W. Zakarias. 2016. *Better Preprocessing for Secure Multiparty Computation.* In ACNS. 327–345.
[6] D. Beaver. 1991. *Efficient Multiparty Protocols Using Circuit Randomization.* In CRYPTO. 420–432.
[7] D. Beaver. 1995. *Precomputing Oblivious Transfer.* In CRYPTO. 97–109.
[8] Z. Beerliová-Trubíniová and M. Hirt. 2006. *Efficient Multi-party Computation with Dispute Control.* In TCC. 305–328.
[9] Z. Beerliová-Trubíniová and M. Hirt. 2008. *Perfectly-Secure MPC with Linear Communication Complexity.* In TCC. 213–230.
[10] M. Ben-Or, S. Goldwasser, and A. Wigderson. 1988. *Completeness Theorems for Non-Cryptographic Fault-Tolerant Distributed Computation (Extended Abstract).* In ACM STOC. 1–10.
[11] Christopher Bishop. 2006. *Pattern Recognition and Machine Learning.*
[12] D. Bogdanov, S. Laur, and J. Willemson. 2008. *Sharemind: A Framework for Fast Privacy-Preserving Computations.* In ESORICS. 192–206.
[13] D. Bogdanov, R. Talviste, and J. Willemson. 2012. *Deploying Secure Multi-Party Computation for Financial Data Analysis.* In FC. 57–64.
[14] M. Byali, A. Joseph, A. Patra, and D. Ravi. 2018. *Fast Secure Computation for Small Population over the Internet.* ACM CCS (2018), 677–694.
[15] O. Catrina and S. de Hoogh. 2010. *Secure Multiparty Linear Programming Using Fixed-Point Arithmetic.* In ESORICS. 134–150.
[16] N. Chandran, J. A. Garay, P. Mohassel, and S. Vusirikala. 2017. *Efficient, Constant-Round and Actively Secure MPC: Beyond the Three-Party Case.* In ACM CCS. 277–294.
[17] H. Chaudhari, A. Choudhury, A. Patra, and A. Suresh. 2019. *ASTRA: High-throughput 3PC over Rings with Application to Secure Prediction.* https://eprint.iacr.org/2019/429. In IACR Cryptology ePrint Archive.
[18] K. Chida, D. Genkin, K. Hamada, D. Ikarashi, R. Kikuchi, Y. Lindell, and A. Nof. 2018. *Fast Large-Scale Honest-Majority MPC for Malicious Adversaries.* In CRYPTO. 34–64.
[19] A. Choudhury and A. Patra. 2017. *An Efficient Framework for Unconditionally Secure Multiparty Computation.* IEEE Trans. Information Theory (2017), 428–468.
[20] R. Cleve. 1986. *Limits on the Security of Coin Flips when Half the Processors Are Faulty (Extended Abstract).* In ACM STOC. 364–369.
[21] R. Cramer, I. Damgård, D. Escudero, P. Scholl, and C. Xing. 2018. *SPDZ2k: Efficient MPC mod 2ˆk for Dishonest Majority.* CRYPTO (2018), 769–798.
[22] R. Cramer, I. Damgård, and Y. Ishai. 2005. *Share Conversion, Pseudorandom Secret-Sharing and Applications to Secure Computation.* In TCC. 342–362.
[23] Cryptography and Privacy Engineering Group at TU Darmstadt. 2017. ENCRYPTO Utils. https://github.com/encryptogroup/ENCRYPTO_utils.
[24] M. Dahl. 2018. *Private Image Analysis with MPC: Training CNNs on Sensitive Data using SPDZ.* (2018).
[25] I. Damgård, C. Orlandi, and M. Simkin. 2018. *Yet Another Compiler for Active Security or: Efficient MPC Over Arbitrary Rings.* CRYPTO (2018), 799–829.
[26] I. Damgård, V. Pastro, N. P. Smart, and S. Zakarias. 2012. *Multiparty Computation from Somewhat Homomorphic Encryption.* In CRYPTO. 643–662.
[27] S. de Hoogh, B. Schoenmakers, P.Chen, and H. Akker. 2014. *Practical Secure Decision Tree Learning in a Teletreatment Application.* In FC. 179–194.
[28] H. Eerikson, M. Keller, C. Orlandi, P. Pullonen, J. Puura, and M. Simkin. 2019. *Use your Brain! Arithmetic 3PC For Any Modulus with Active Security.* IACR
Cryptology ePrint Archive (2019).
[29] A. Esteva, B. Kuprel, R. A. Novoa, J. Ko, S. M. Swetter, H. M. Blau, and S. Thrun. 2017. *Dermatologist-level classification of skin cancer with deep neural networks.* Nature (2017), 115–118.
[30] J. Furukawa, Y. Lindell, A. Nof, and O. Weinstein. 2017. *High-Throughput Secure Three-Party Computation for Malicious Adversaries and an Honest Majority.* In EUROCRYPT. 225–255.
[31] A. Gascón, P. Schoppmann, B. Balle, M. Raykova, J. Doerner, S. Zahur, and D. Evans. 2016. *Secure Linear Regression on Vertically Partitioned Datasets.* IACR Cryptology ePrint Archive (2016).
[32] M. Geisler. 2007. *Viff: Virtual ideal functionality framework.*
[33] O. Goldreich, S. Micali, and A. Wigderson. 1987. *How to Play any Mental Game or A Completeness Theorem for Protocols with Honest Majority.* In STOC. 218–229.
[34] S. D. Gordon, S. Ranellucci, and X. Wang. 2018. *Secure Computation with Low Communication from Cross-Checking.* In ASIACRYPT. 59–85.
[35] Y. Ishai, R. Kumaresan, E. Kushilevitz, and A. Paskin-Cherniavsky. 2015. *Secure Computation with Minimal Interaction, Revisited.* In CRYPTO. 359–378.
[36] S. Kamara, P. Mohassel, and M. Raykova. 2011. *Outsourcing Multi-Party Computation.* IACR Cryptology ePrint Archive (2011).
[37] J. Katz, V. Kolesnikov, and X. Wang. 2018. *Improved Non-Interactive Zero Knowledge with Applications to Post-Quantum Signatures.* In CCS. 525–537.
[38] M. Keller, E. Orsini, and P. Scholl. 2016. *MASCOT: Faster Malicious Arithmetic Secure Computation with Oblivious Transfer.* In ACM CCS. 830–842.
[39] M. Keller, V. Pastro, and D. Rotaru. 2018. *Overdrive: Making SPDZ Great Again.* In EUROCRYPT. 158–189.
[40] J. Launchbury, D. Archer, T. DuBuisson, and E. Mertens. 2014. *Application-Scale Secure Multiparty Computation.* In ESOP. 8–26.
[41] S. Laur, H. Lipmaa, and T. Mielikäinen. 2006. *Cryptographically private support vector machines.* In ACM SIGKDD. 618–624.
[42] Yann LeCun and Corinna Cortes. 2010. *MNIST handwritten digit database.* (2010). http://yann.lecun.com/exdb/mnist/
[43] Y. Lindell and A. Nof. 2017. *A Framework for Constructing Fast MPC over Arithmetic Circuits with Malicious Adversaries and an Honest-Majority.* In ACM CCS. 259–276.
[44] J. Liu, M. Juuti, Y. L., and N. Asokan. 2017. *Oblivious Neural Network Predictions via MiniONN Transformations.* In ACM CCS. 619–631.
[45] E. Makri, D. Rotaru, N. P. Smart, and F. Vercauteren. 2018. *EPIC: Efficient Private Image Classification (or: Learning from the Masters).* CT-RSA (2018), 473–492.
[46] P. Mohassel and P. Rindal. 2018. *ABY3: A Mixed Protocol Framework for Machine Learning.* In ACM CCS. 35–52.
[47] P. Mohassel, M. Rosulek, and Y. Zhang. 2015. *Fast and Secure Three-party Computation: Garbled Circuit Approach.* In CCS. 591–602.
[48] P. Mohassel and Y. Zhang. 2017. *SecureML: A System for Scalable Privacy-Preserving Machine Learning.* In IEEE S&P. 19–38.
[49] V. Nikolaenko, S. Ioannidis, U. Weinsberg, M. Joye, N. Taft, and D. Boneh. 2013. *Privacy-preserving matrix factorization.* In ACM CCS. 801–812.
[50] V. Nikolaenko, U. Weinsberg, S. Ioannidis, M. Joye, D. Boneh, and N. Taft. 2013. *Privacy-Preserving Ridge Regression on Hundreds of Millions of Records.* In IEEE S&P. 334–348.
[51] P. S. Nordholt and M. Veeningen. 2018. *Minimising Communication in Honest-Majority MPC by Batchwise Multiplication Verification.* In ACNS. 321–339.
[52] T. Orekondy, B. Schiele, and M. Fritz. 2018. *Knockoff Nets: Stealing Functionality of Black-Box Models.* CoRR (2018).
[53] N. Papernot, P. McDaniel, I. Goodfellow, S. Jha, Z. B. Celik, and A. Swami. 2017. *Practical Black-Box Attacks Against Machine Learning.* In ASIA CCS. 506–519.
[54] A. Patra and D. Ravi. 2018. *On the Exact Round Complexity of Secure Three-Party Computation.* CRYPTO (2018), 425–458.
[55] M. S. Riazi, C. Weinert, O. Tkachenko, E. M. Songhori, T. Schneider, and F. Koushanfar. 2018. *Chameleon: A Hybrid Secure Computation Framework for Machine Learning Applications.* In AsiaCCS. 707–721.
[56] F. Schroff, D. Kalenichenko, and J. Philbin. 2015. *FaceNet: A unified embedding for face recognition and clustering.* In IEEE CVPR. 815–823.
[57] N. P. Smart and T. Wood. 2019. *Error Detection in Monotone Span Programs with Application to Communication-Efficient Multi-party Computation.* In CT-RSA. 210–229.
[58] F. Tramèr, F. Zhang, A. Juels, M. K. Reiter, and T. Ristenpart. 2016. *Stealing Machine Learning Models via Prediction APIs.* In USENIX. 601–618.
[59] S. Wagh, D. Gupta, and N. Chandran. 2019. *SecureNN: 3-Party Secure Computation for Neural Network Training.* PoPETs (2019), 26–49.
[60] A. C. Yao. 1982. *Protocols for Secure Computations.* In FOCS. 160–164.