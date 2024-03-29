---
title: '论文笔记 ABY3: A Mixed Protocol Framework for Machine Learning'
date: 2020-06-18 20:06:46
categories: Papers
tags: [PPML, MPC, Neural Network, Linear Regression, Logistic Regression, Secret Sharing]
---

*Payman Mohassel, Peter Rindal*

CCS 2018

https://dl.acm.org/doi/10.1145/3243734.3243760

<!--more-->

![](http://images.yingwai.top/picgo/ABY3.png)

# 本文贡献

* 新的共享十进制数近似定点乘法协议；
* 提出了三方的ABY框架，其中许多转换都是基于新技术，并在本文中首次进行了设计和优化；
* 提高了矩阵乘法的效率；
* 有效的分段多项式函数评估。



# 计算模型

本文采用三方的重复共享，即把一个数 $x$ 拆成三份，各方都拥有三份中的两份，下面是ABY三种共享形式：

![](http://images.yingwai.top/picgo/aby3ppt4.png)

在这种设置下，三方交互式地计算函数 $f(x)$，本篇文章主要针对机器学习中的算法。

![](http://images.yingwai.top/picgo/aby3ppt2.png)



## 加法

在秘密共享的情况下，计算两个数 $[\![x]\!], [\![y]\!]$ 的和 $[\![z]\!] = [\![x]\!] + [\![y]\!]$ 比较简单，各方直接将其对应的份额在本地相加即可，不需要交互：
$$
z_i = x_i + y_i
$$


## 乘法

计算两个数 $[\![x]\!], [\![y]\!]$ 的积 $[\![z]\!] = [\![x]\!] * [\![y]\!]$ 则复杂一点。在两方共享的情况下，需要借助额外的乘法三元组来计算，开销非常大。而三方下的复制共享这时候就起作用了，将 $xy$ 展开可以观察到有九项，每一项是两个份额的乘积，因此每一方只需要计算其可以计算的九项中的其中三项即可保持结果 $z$ 继续在三方中是加法共享的。计算完成后再把对应的结果发送给另一方，即可保持 $z$ 是重复共享的。

![](http://images.yingwai.top/picgo/aby3ppt3.png)

可以看到乘法操作只包含了本地操作和各方各一次发送，开销比两方的情况下小很多。



# 本文的框架

## 定点运算

定点值定义为使用二进制补码表示的一个 $k$ 位整数，其中底部 $d$ 位表示小数，即对于正值，位 $i$ 表示 $2$ 的 $(i−d)$ 次方。由于预计结果将保持在 $2^k$ 以下，因此可以使用相应的整数运算来执行加法和减法。乘法也可以以相同的方式执行，但是十进制位的数量加倍，因此必须除以 $2^d$ 以保持 $d$ 个十进制位不变。

在得到乘法结果后，各方可以直接把本地的份额后 $d$ 位截断，但这样会**把原本份额相加的进位也截断了**，导致截断后重构出来的 $\tilde{x} \neq x$。作者参考了[SecureML](https://yuyingwai.cn/2020/06/17/论文笔记-SecureML-A-System-for-Scalable-Privacy-Preserving-Machine-Learning/)中的两方协议，给出了他们自己的在三方下的实现：

![](http://images.yingwai.top/picgo/aby3f1.png)

一开始作者的想法是把 $x'$ 的截断 $x = x'/2^d$ 拆成 $[\![x]\!] := (x_1, x_2, x_3) = (x'_1/2^d, (x'_2 + x'_3)/2^d-r, r)$，其中 $r$ 是 $\mathbb{Z}^k_2$ 中的随机数，但发现这样做有两轮通信的限制，于是进一步优化得到上面的协议。



## 矢量化乘法

两个向量作内积定义为 $\vec{x} \cdot \vec{y} := \sum^n_{i=1} x_i y_i$。在本文的背景下计算内积，朴素的做法是对两个向量的每一个元素都执行一次乘法+截断协议，但这样需要通信 $O(n)$ 次通信。

于是作者进行了优化：各方只需要在本地将两个向量 $\vec{x},\vec{y}$ 的 $n$ 个分量计算相乘的份额，然后把这 $n$ 个积相加就得到了 $z'$ 的3/3共享，最后执行上一小节中的截断协议将 $[\![z']\!]$ 截断为 $[\![z]\!]:= (z'+r')/2^d - [\![r]\!]$。

这样做不仅可以减少通信开销，而且误差也比执行 $n$ 次乘法+截断协议低（总共只进行了一次截断，相对于整体内积的误差为 $2^{-d}$）。



## 共享转换

机器学习中有很多不同的函数，而不同的函数有适合其计算的不同共享方式：算术共享适合计算乘法和加法，而二进制共享适合计算非线性函数、最大池化以及平均值等。两方的情况可以参考[ABY](https://yuyingwai.cn/2020/06/11/论文笔记-ABY-A-Framework-for-Efficient-Mixed-Protocol-Secure-Two-Party-Computation/)，本文给出了三方下的共享转换。

![](http://images.yingwai.top/picgo/aby3ppt6.png)



### 位分解

$[\![x]\!]^A \rightarrow [\![\vec{x}]\!]^B$

最基础的想法是各方将其份额 $[\![x]\!]^A = (x_1, x_2, x_3)$ 输入到布尔电路中计算它们的和，但可以进行优化。观察到 $[\![x]\!]^A = (x_1, x_2, x_3)$ 可以转换为 $[\![x_1]\!]^B := (x_1,0,0), [\![x_2]\!]^B := (0,x_2,0), [\![x_3]\!]^B := (0,0,x_3)$ 而不需要交互。

这里没太看懂，用到了各种加法器。作者首先观察到 $x_1 + x_2 + x_3$ 的计算可以减少成执行 $k$ 个相互独立的全加器计算 $2c+s$：$\mathsf{FA}(x_1[i], x_2[i], x_3[i-1]) \rightarrow (c[i], s[i])$，其中 $i \in \{0,...,k-1\}$。然后再使用平行前缀加法器计算 $2[\![c]\!]^B + [\![s]\!]^B$。对于半诚实的情况，$P_2$ 可以提供 $(x_1 + x_2)$ 作为输入计算 $[\![x]\!]^B := [\![x_1 + x_2]\!]^B + [\![x_3]\!]^B$。



### 位提取

$[\![x]\!]^A \rightarrow [\![x[i]]\!]^B$

跟上面的情况类似，只是这次是提取比特串中的单一比特，把上一小节的电路中不必要的门去掉即可。



### 位组合

$[\![x]\!]^B \rightarrow [\![x]\!]^A$

用到的还是与上面类似的电路，只是操作顺序有点变化。首先 $P_1, P_2$ 输入一个随机共享 $[\![-x_2]\!]^B$，$P_2, P_3$ 同样输入一个随机共享 $[\![-x_3]\!]^B$，这两个将会是最终的转换结果的部分。这里可以利用两方间的密钥作为随机种子来生成随机数。

然后三方计算 $\mathsf{FA}([\![x[i]]\!]^B, [\![-x_2[i]]\!]^B, [\![-x_3[i]]\!]^B) \rightarrow ([\![c[i]]\!]^B, [\![s[i]]\!]^B)$，其中 $i \in \{0,...,k-1\}$。然后使用并行前缀加法器计算 $[\![x_1]\!]^B := 2[\![c]\!]^B + [\![s]\!]^B$。同样在半诚实下可以进一步优化，$P_2$ 提供 $(-x_2, -x_3)$ 作为输入计算 $[\![x_1]\!]^B := [\![x]\!]^B + [\![-x_2 - x_3]\!]^B$，$x_1$ 对 $P_1, P_3$ 开放，最后的共享定义为 $[\![x]\!]^A := (x_1, x_2, x_3)$。



### 位注入

$[\![x]\!]^B \rightarrow [\![x]\!]^A$

当需要将以二进制共享方式编码的单个位 $x$ 提升为算术共享 $[\![x]\!]^A$ 时，通常会出现另一种特殊情况，参考3.4。



### 联合姚输入

在姚共享中，对于一个比特 $x$，$P_1$（评估者）拥有 $k^x_{\mathsf{x}}$ 而另外两方拥有 $k^0_{\mathsf{x}} \in \{0,1\}^{\kappa}$ 和一个全局随机串 $\Delta \in \{0,1\}^{\kappa}$ 使得 $k^1_{\mathsf{x}} := k^0_{\mathsf{x}} \oplus \Delta$。转换到姚共享和从姚共享转换的一个有用的原语是双方提供双方都知道的输入的能力。比如说 $P_1, P_2$ 知道一个比特 $x$，想生成共享 $[\![x]\!]^Y$。在半诚实设置中比较简单，$P_2$ 可以本地生成并发送 $[\![x]\!]^Y$ 给 $P_1$。但是在恶意设置下 $P_1$ 需要在不知道 $\Delta$ 的情况下验证 $[\![x]\!]^Y$ 正确地将 $x$ 编码。于是 $P_3$ 可以帮助 $P_1$ 进行验证：$P_2, P_3$ 同时发送 $k^0_{\mathsf{x}}$ 和 $k^1_{\mathsf{x}}$ 的承诺方案给 $P_1$ ，后者可以检查两个承诺是否一致以及是否能打开承诺。



### 姚共享到二进制共享

$[\![x]\!]^Y \rightarrow [\![x]\!]^B$

在两方的情况下，密钥的最低有效位（置换位）形成 $x$ 的两方共享，即 $x \oplus p_{\mathsf{x}} = k^x_{\mathsf{x}}[0]$，其中 $p_{\mathsf{x}} = k^0_{\mathsf{x}}[0]$。三方下 $P_3$ 也知道 $p_{\mathsf{x}}$，首先 $P_1,P_2$ 本地生成随机比特 $r$ 然后 $P_1$ 将 $k^x_{\mathsf{x}}[0] \oplus r = x \oplus p_{\mathsf{x}} \oplus r$ 发送给 $P_3$。于是就完成了共享的转换，原本的共享被转换为 $[\![x]\!]^B = (x \oplus p_{\mathsf{x}} \oplus r, r, p_{\mathsf{x}})$。

在恶意的设置下，需要确认 $P_1$ 发送给 $P_3$ 的 $x \oplus b \oplus r$ 中的 $b = p_{\mathsf{x}}$。首先 $P_1, P_2$ 选取 $k^r_{\mathsf{r}} \leftarrow \{0,1\}^{\kappa}$，后者发送 $k^0_{\mathsf{r}} := k^r_{\mathsf{r}} \oplus (r \Delta)$ 给 $P_3$。然后 $P_2,P_3$ 发送承诺 $C_0 = \mathsf{Comm}(k^{p_{\mathsf{x}}}_{\mathsf{y}}), C_1 = \mathsf{Comm}(k^{\overline{p_{\mathsf{x}}}}_{\mathsf{y}})$ 给 $P_1$，其中 $k^0_{\mathsf{y}} := k^0_{\mathsf{x}} \oplus k^0_{\mathsf{r}}$。$P_1$ 发送 $k^{x \oplus r}_{\mathsf{y}} := k^x_{\mathsf{x} } \oplus k^r_{\mathsf{r}}$ 给 $P_3$，后者验证其是否在集合 $\{k^0_{\mathsf{y}}, k^1_{\mathsf{y}}\}$ 中。$P_1$ 同样验证承诺 $C_{p_{\mathsf{x}} \oplus x \oplus r}$ 是否可以打开为 $k^{x \oplus r}_{\mathsf{y}}$ 以及 $C_0, C_1$ 是否一致。注意到 $x \oplus p_{\mathsf{x}} = k^x_{\mathsf{x}}[0]$，于是三方就可以计算共享 $[\![x]\!]^B = (x \oplus p_{\mathsf{x}} \oplus r, r, p_{\mathsf{x}})$。观察到 $P_3$ 将 $x \oplus p_{\mathsf{x}} \oplus r$ 计算为 $k^{x \oplus r}_{\mathsf{y}}[0] \oplus p_{\mathsf{r}}$。



### 二进制共享到姚共享

$[\![x]\!]^B \rightarrow [\![x]\!]^Y$

设 $[\![x]\!]^B = (x_1, x_2, x_3)$。各方使用前面讨论的程序共同输入份额 $[\![x_1]\!]^Y,[\![x_2]\!]^Y,[\![x_3]\!]^Y$ 作为联合姚输入。然后可以使用乱码电路来计算最终份额，该电路计算三个值的异或 $[\![x]\!]^Y := [\![x_1]\!]^Y \oplus [\![x_2]\!]^Y \oplus [\![x_3]\!]^Y$ 。使用Free-XOR技术，这不需要三方之间的任何通信，并且可以由 $P_1$ 本地计算。在半诚实的设置中，可以基于 $P_2$ 知道 $x_2$ 和 $x_3$ 这一事实来进一步优化。因此，它们可以本地计算 $x_2 \oplus x_3$，并将 $[\![x_2 \oplus x_3]\!]^Y$ 发送给 $P_1$，后者本地计算 $[\![x]\!]^Y := [\![x_1]\!]^Y \oplus [\![x_2 \oplus x_3]\!]^Y$。



### 姚共享到算术共享

$[\![x]\!]^Y \rightarrow [\![x]\!]^A$

自然的想法是将姚共享先转换成二进制共享，再用位组合中的方法转换为算术共享。作者在这里也给出了优化方案，还是使用到了乱码电路，对这部分看得不是很懂。



### 算术共享到姚共享

$[\![x]\!]^A \rightarrow [\![x]\!]^Y$

各方联合输入 $[\![x]\!]^A = (x_1, x_2, x_3)$ 的共享 $[\![x_1]\!]^Y,[\![x_2]\!]^Y,[\![x_3]\!]^Y$。然后使用一个乱码电路生成 $[\![x]\!]^Y := [\![x_1]\!]^Y + [\![x_2]\!]^Y + [\![x_3]\!]^Y$。



## 计算 $[\![a]\!]^A [\![b]\!]^B = [\![ab]\!]^A$

虽然将共享转换后再计算上面的公式也可以，但作者在这里给出了更有效的特定协议。在训练Logistic回归和神经网络模型中通常用于逼近非线性激活函数的分段线性或多项式函数的计算需要重复该操作。



### 半诚实

#### 三方OT

作者首先给出了三方下的不经意传输协议，在两方的OT中有两个角色：sender和receiver。在这里作者添加了一个*helper*的角色，他不会收到输出且知道receiver的选择位。这个对于(sender, receiver, helper)的功能记为 $((m_0, m_1),c, c) \mapsto (\perp, m_c, \perp)$。

首先sender和helper选取随机串 $w_0, w_1 \leftarrow \{0,1\}^k$。sender将消息盲化：$m_0 \oplus w_0, m_1 \oplus w_1$ 然后发送给receiver。helper知道receiver希望收到消息 $m_c$，这样helper可以发送 $w_c$ 给receiver来让后者重构 $m_c$。



#### 计算 $a[\![b]\!]^B = [\![ab]\!]^A$

最简单的一种情况是 $P_1$ 知道的一个公共值 $a \in \mathbb{Z}_{2^k}$ 和一个共享比特 $b \in \{0,1\}$ 的乘法。首先 $P_3$ (sender)选取随机数 $r \leftarrow \mathbb{Z}_{2^k}$ 和定义两个消息 $m_i := (i \oplus b_1 \oplus b_3)a - r$，其中 $i \in \{0,1\}$。$P_2$ (receiver)为了学习消息 $m_{b_2} = (b_2 \oplus b_1 \oplus b_3)a - r = ba - r$，定义他的输入为 $b_2$。注意到 $P_1$ (helper)也知道 $b_2$，因此这里可以使用前面提到的三方OT。然后三方本地生成重复0共享 $(s_1, s_2, s_3)$ 来计算 $[\![c]\!] = [\![ab]\!] = (s_1 + r, ab-r+s_3, s_3)$。但是为了令这个2/3秘密共享合法，$c_2 = ab - r + s_3$ 要发送给 $P_1$，导致需要总共两轮通信。或者可以(并行地)重复执行三方OT过程，$P_3$ 担任sender输入 $(i+b_2+b_3)a - r + s_3$，其中 $i \in \{0,1\}$。于是 $P_1$ (receiver)输入 $b_2$，在第一轮中获知消息 $c_2$（非 $m_{b_2}$），总共 $6k$ 比特和 $1$ 轮通信。



#### 计算 $[\![a]\!]^A [\![b]\!]^B = [\![ab]\!]^A$

在半诚实下，并行地执行两次 $a[\![b]\!]^B = [\![ab]\!]^A$ 过程就足够了。关键在于观察到上述计算中的 $a$ 不必是公开的。也就是说，$P_1$ 可以私下选择 $a$ 的值。利用这一点，可以观察到表达式可以写成 $[\![a]\!][\![b]\!]^B = a_1[\![b]\!]^B + (a_2 + a_3)[\![b]\!]^B$。$P_1$ 充当第一次的sender，$P_3$ 充当第二次的sender。每一方总共在 $1$ 轮上传送 $4k$ 比特。



### 恶意

#### 计算 $a[\![b]\!]^B = [\![ab]\!]^A$

因为 $P_1$ 可以任意选择他输入到OT的值 $a$，所以半诚实方法在恶意设置下不适用。为了避开这一点，作者在这里先执行对 $b$ 的*位注入*。即计算 $[\![b]\!]^B \rightarrow [\![b]\!]^A$ 然后 $a[\![b]\!]^A = [\![ab]\!]^A$。在3.3节中提到，各方可以在本地计算共享 $[\![b_1]\!]^A, [\![b_2]\!]^A, [\![b_2]\!]^A$，其中 $[\![b]\!]^B = (b_1, b_2, b_3)$。现在可以通过计算 $[\![b_1 \oplus b_2]\!]^A = [\![d]\!]^A :=$ $[\![b_1]\!]^A + [\![b_2]\!]^A - 2[\![b_1]\!]^A[\![b_2]\!]^A$ 和 $[\![b]\!]^A := [\![d \oplus b_3]\!]^A$ 在算术电路中模拟这些值的异或。然后可以将最终结果计算为 $[\![ab]\!]^A := a[\![b]\!]^A$，其中每一方用 $a$ 去乘以自己 $b$ 的份额。



#### 计算 $[\![a]\!]^A [\![b]\!]^B = [\![ab]\!]^A$

同样，这里可以重复位注入过程，将 $[\![b]\!]^B$ 转换为 $[\![b]\!]^A$，然后使用乘法协议计算 $[\![a]\!]^A[\![b]\!]^A$。



## 多项式分段函数

设 $f_1,...,f_m$ 表示具有公共系数的多项式以及有 $-\infty = c_0 < c_1 < ... < c_{m-1} < c_m = \infty$ 使得 $c_{i-1}<x \leq c_i$ 时 $f(x) = f_i(x)$。本文的方法是先计算向量 $b_1,...,b_m \in \{0,1\}$ 的秘密共享值使得 $b_i = 1 \Leftrightarrow c_{i-1} < x \leq c_i$，然后 $f$ 可以计算为 $f(x) = \sum_i b_i f_i(x)$。

![](http://images.yingwai.top/picgo/aby3ppt5.png)

比如说要比较 $[\![x]\!]$ 与 $c$ 的大小，可以看成提取 $[\![x-c]\!]$ 的最高有效位(MSB)（为1时 $x-c<0$）。这里可以对 $[\![x-c]\!]$ 使用5.3中的位提取来获得二进制共享 $[\![b]\!]^B := [\![\mathsf{msb}(x-c)]\!]^B$。

每个 $f_i$ 函数被表示为多项式 $f_i([\![x]\!]) = a_{i,j}[\![x]\!]^j + ... + a_{i,1}[\![x]\!] + a_{i,0}$，其中所有的 $a_{i,l}$ 都是公知的常数。当 $f_i$ 为0次多项式时，使用3.4中的技术可以将计算 $b_if_i([\![x]\!])$ 优化为 $a_{i,0}[\![b]\!]^B$。另外当 $f_i$ 的系数为整数时，给定 $[\![x]\!]^l$，$a_{i,l} [\![x]\!]^l$ 的计算可以在本地进行。但是当 $a_{i,j}$ 有非零小数时，将按照3.1的方法执行交互式截断。



# 应用到机器学习

作者还给出了本文框架分别在线性回归、逻辑回归以及神经网络中的应用。



## 线性回归

![](http://images.yingwai.top/picgo/aby3ppt7.png)

线性回归是比较简单的一种模型，前向运算和更新权重都只包含了加法和乘法操作，因此只要利用前面提到的加法和乘法协议就可以完成一个线性回归模型的训练。



## 逻辑回归

![](http://images.yingwai.top/picgo/aby3ppt8.png)

逻辑回归是在线性回归的基础上增加了一个Sigmoid函数，因为在秘密共享的情况下计算指数函数很困难，因此可以定义一个近似函数或用多项式去逼近。文中3.5节给出了计算此类函数的方法，这里近似函数可以使用[SecureML](https://yuyingwai.cn/2020/06/17/论文笔记-SecureML-A-System-for-Scalable-Privacy-Preserving-Machine-Learning/)中定义的：
$$
f(x) = \left\{ \begin{array}{lcl}0, & \mbox{if} & x < - \frac{1}{2} \\x + \frac{1}{2}, & \mbox{if} & - \frac{1}{2} \leq x \leq \frac{1}{2} \\1, & \mbox{if} & x > \frac{1}{2}\end{array}\right.
$$
![](http://images.yingwai.top/picgo/smlf5.png)



## 神经网络

![](http://images.yingwai.top/picgo/aby3ppt9.png)

一个神经网络由很多个神经元组成，每一个神经元可以看成是与逻辑回归计算一致的单元，只是最后的非线性函数不一定是Sigmoid函数（替换成ReLU）。