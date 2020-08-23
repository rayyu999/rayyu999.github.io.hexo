---
title: PPML中的2PC对比3PC
date: 2020-06-19 11:03:28
categories: Study
tags: [MPC, 密码学, PPML, Secret Sharing]
---

----



<!--more-->

# 共享语义

不同的论文可能设置不太一样，这里2PC我参考的是[SecureML](https://yuyingwai.cn/2020/06/17/论文笔记-SecureML-A-System-for-Scalable-Privacy-Preserving-Machine-Learning/)中的设置，3PC参考的是[ABY3](https://yuyingwai.cn/2020/06/18/论文笔记-ABY3-A-Mixed-Protocol-Framework-for-Machine-Learning/)和[ASTRA](https://yuyingwai.cn/2020/04/20/论文笔记-ASTRA-High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/)中的设置。即对于一个数 $x$ 的算术共享，2PC的情况是将其拆成两份 $x_0$ 和 $x_1$，它们的和为 $x$，两方各拥有一份；3PC的情况是将其拆成三份 $x_0, x_1, x_2$，三方各拥有其中的两份，例如 $P_0$ 有 $(x_0, x_2)$，任意两方即可重构出 $x$。

布尔共享是各份额二进制表示的每一位异或等于原秘密 $x$ 的对应位。

![](http://images.yingwai.top/picgo/2vs3ppt2.png)



# 加法

2PC和3PC下算术共享的加法是一样的，都只需要在本地将对应数的份额相加，不需要交互。



# 乘法

## 2PC

乘法则有比较大的区别，首先是2PC，计算乘法 $ab = c$ 需要生成额外的乘法三元组 $(u,v,z)$ 满足 $z = uv$，这个三元组中的每个数也是在两方间加法共享的。

首先两方交互计算 $e = a-u$ 和 $f = b - v$，然后 $P_i$ 将结果 $c = ab$ 的份额设为 $c_i = -ief+a_if+eb_i+z_i$，其中 $i \in \{0,1\}$。

![](http://images.yingwai.top/picgo/2vs3ppt1.png)



## 3PC

接下来是3PC，由于重复共享的设置（各方拥有三个份额中的两个），计算乘法时几乎不需要交互。计算 $x,y$ 两个数的积 $xy = z$ 时，观察到
$$
\begin{align}
xy &= (x_0 + x_1 + x_2)(y_0 + y_1 + y_2) \\
&= x_0y_0 + x_0y_1 + x_0y_2 \\
&+ x_1y_0 + x_1y_1 + x_1y_2 \\
&+ x_2y_0 + x_2y_1 + x_2y_2
\end{align}
$$
可以看到拆开的式子一共有九项，我们可以直接让 $P_i$ 各计算其中三项，令
$$
z_0 = x_0y_0 + x_0y_2 + x_2y_0 \\
z_1 = x_0y_1 + x_1y_0 + x_1y_1 \\
z_2 = x_1y_2 + x_2y_1 + x_2y_2
$$
这时结果 $z$ 就以加法共享的方式在三方中共享，最后 $P_i$ 将自己的 $z_i$ 发送给 $P_{i+1}$（$i=2$ 时 $i+1 = 0$），$z$ 就重新在三方中重复共享了。

![](http://images.yingwai.top/picgo/aby3ppt3.png)

可以看到计算过程中只涉及本地操作，只有在生成重复共享时才需要三方各一次发送，效率比2PC要高。



# 比较

比较两个数 $u,v$ 的大小，等同于提取 $a = u-v$ 的 $\mbox{msb}$，$u>v \rightarrow u-v>0$ 时 $\mbox{msb}(a) = 0$，反之 $\mbox{msb}(a) = 1$。两方下没有给出具体的细节，[SecureML](https://yuyingwai.cn/2020/06/17/论文笔记-SecureML-A-System-for-Scalable-Privacy-Preserving-Machine-Learning/)中只提到使用乱码电路，这是一个比较昂贵的协议（将算术共享转换为布尔或姚共享也需要开销），而三方下的[ASTRA](https://yuyingwai.cn/2020/04/20/论文笔记-ASTRA-High-Throughput-3PC-over-Rings-with-Application-to-Secure-Prediction/)中则利用了秘密共享方案的不对称性，下面的协议是基于 $\mbox{sign}(r \cdot a) = \mbox{sign}(r) \oplus \mbox{sign}(a)$ 的事实：

首先在离线阶段，$P_1,P_2$ 共同选取两个随机数 $r, r' \in \mathbb{Z}_{2^l}$ 并设 $p = \mbox{msb}(r)$，然后各方可以非交互式地设置 $p$ 的布尔共享 $[\![p]\!]^{\rm B}_{P_0} = (0,0)$, $[\![p]\!]^{\rm B}_{P_1} = (p,0)$ 以及 $[\![p]\!]^{\rm B}_{P_2} = (p,0)$。

![](http://images.yingwai.top/picgo/2vs3ppt3.png)

到了在线阶段，$P_1$ 和 $P_2$ 设置它们对 $a$ 的份额，使得它们两方各自的份额之和为 $a$ ：$P_1$ 设 $[a]_{P_1} = x_0+x_1$，$P_2$ 设 $[a]_{P_2}=x_2$。然后 $P_1$ 和 $P_2$ 利用离线阶段生成的随机数 $r'$ 对 $[ra]_{P_i}$ 进行盲化并发送给 $P_0$（由于 $P_0$ 知道 $x_2$，若不进行盲化则会暴露 $r$ 给 $P_0$），$P_0$ 重构 $ra$ 并设 $q = \mbox{msb}(ra)$，然后生成 $q$ 在三方中的布尔共享。最后各方在本地计算 $[\![\mbox{msb}(a)]\!]^B = [\![p]\!]^B \oplus [\![q]\!]^B$，此时 $a$ 的符号位在三方中是布尔共享的。

![](http://images.yingwai.top/picgo/2vs3ppt4.png)

