---
title: 论文笔记 Practical Secure Aggregation for Privacy-Preserving Machine Learning
date: 2020-12-22 22:14:06
categories: Papers
tags: [PPML, Federated Learning, Secure Aggregation]
---

*Keith Bonawitz, Vladimir Ivanov, Ben Kreuter, Antonio Marcedone, H. Brendan McMahan, Sarvar Patel, Daniel Ramage, Aaron Segal, Karn Seth*

CCS 2017

https://dl.acm.org/doi/10.1145/3133956.3133982

<!--more-->



# INTRODUCTION

## 贡献

提出了一种安全计算向量和的协议，该协议具有轮数恒定、通信开销小、抗故障能力强、只需要一台信任有限的服务器等优点。在我们的设计中，服务器有两个角色：它在其他各方之间路由消息，并计算最终结果。我们介绍了该协议的两种变体：一种更高效，在普通模型中可以被证明是安全的，可以抵御诚实但好奇的对手。另一种保证隐私不被活跃的对手攻击(包括活跃的敌意服务器)，但是需要额外的一轮，并且在随机预言模型中被证明是安全的。在这两种情况下，我们都可以正式地证明，服务器只学习用户的总体输入，使用基于模拟的证明，这是MPC协议的标准。我们展示的这两种变体都是实用的，并且我们展示了原型实现的基准测试结果。



# SECURE AGGREGATION FOR FEDERATED LEARNING

![](http://images.yingwai.top/picgo/20201222190118.png)

考虑训练一个深度神经网络来预测用户在撰写文本消息时要输入的下一个单词。这类型号通常用于提高手机屏幕键盘的打字效率[30]。建模者可能希望对跨越大量用户的所有文本消息训练这样的模型。但是，文本消息经常包含敏感信息；用户可能不愿意将它们的副本上载到建模者的服务器。取而代之的是，我们考虑在联合学习环境中训练这样的模型，其中每个用户在她自己的移动设备上安全地维护她的文本消息的私有数据库，并且基于来自用户的高度处理的、最小范围的、短暂的更新，在中央服务器的协调下训练共享全局模型[43，50]。

这些更新是基于来自用户私有数据库的信息的高维矢量。训练神经网络通常是通过使用小批量随机梯度下降规则的变体反复迭代这些更新来完成的[15，29]。(详情见附录B)

尽管每次更新都是短暂的，并且包含的信息并不比用户的私有数据库多(通常要少得多)，但用户可能仍然关心剩余的信息。在某些情况下，可以通过检查用户的最新更新来了解用户键入的个人单词。但是，在联合学习设置中，服务器不需要访问任何单个用户的更新即可执行随机梯度下降；它只需要更新向量的元素加权平均值(取随机用户子集)。使用安全聚合协议来计算这些加权平均值1将确保服务器可以仅获知该随机选择的子集中的一个或多个用户写了给定词，而不是知道哪些用户写了给定词。

联邦学习系统面临着几个实际挑战。移动设备只能零星地访问电源和网络连接，因此参与每个更新步骤的用户集是不可预测的，并且系统必须对用户退出具有健壮性。因为神经网络可以由数百万个值参数化，所以更新可能很大，表示用户在计量网络计划上的直接成本。移动设备通常也不能与其他移动设备建立直接通信信道(依赖服务器或服务提供商来协调这种通信)，也不能本地认证其他移动设备。

因此，联合学习激发了对安全聚合协议的需求，该协议可以：

1. 在高维向量上操作
2. 通信效率很高，即使每次实例化都有一组新的用户
3. 对用户退出有健壮性
4. 在服务器中介的未经身份验证的网络模型的约束下提供最强的安全性



# TECHNICAL INTUITION

本文将实体分为了两类：一个单独的作*聚合*操作的服务器 $S$，以及包括 $n$ 个客户端的集合 $\mathcal{U}$。每个 $\mathcal{U}$ 中的用户都有一个私有的 $m$ 维向量 $\boldsymbol{x}_u$，且 $\sum_{u \in \mathcal{U}} \boldsymbol{x}_u$ 在域 $\mathbb{Z}_R$ 中。协议的目标为安全地计算 $\sum_{u \in \mathcal{U}} \boldsymbol{x}_u$：保证服务器只学习到最终的求和结果，而用户什么也学习不到。



## Masking with One-Time Pads

假设现在用户之间有顺序，且每对用户 $(u,v), u < v$ 之间共有一个随机的向量 $\boldsymbol{s}_{u,v}$，那么就可以对数据 $\boldsymbol{x}_u$ 做以下盲化操作：
$$
\boldsymbol{y}_{u}=\boldsymbol{x}_{u}+\sum_{v \in \mathcal{U}: u<v} s_{u, v}-\sum_{v \in \mathcal{U}: u>v} s_{v, u}(\bmod R)
$$
然后将 $\boldsymbol{y}_{u}$ 发送给服务器，其计算：
$$
\begin{aligned}
z &=\sum_{u \in \mathcal{U}} y_{u} \\
&=\sum_{u \in \mathcal{U}}\left(x_{u}+\sum_{v \in \mathcal{U}: u<v} s_{u, v}-\sum_{v \in \mathcal{U}: u>v} s_{v, u}\right) \\
&=\sum_{u \in \mathcal{U}} x_{u} \quad(\bmod R)
\end{aligned}
$$
可以看到在各用户之间共有的向量最终抵消了，完成了安全聚合的任务。

但这种方法有两个缺点。第一个是用户必须交换随机向量 $\boldsymbol{s}_{u,v}$，如果只是简单的做，将需要二次通信开销 ($|\mathcal{U}| \times|x|$)。第二，对于未能完成协议的一方是不可容忍的：如果用户 $u$ 在与其他用户交换向量之后退出，但在将 $\boldsymbol{y}_u$ 提交给服务器之前，与 $u$ 相关联的向量掩码将不会在总和 $z$ 中被抵消。

## Efficient Communication and Handling Dropped Users

可以将共享整个向量 $\boldsymbol{s}_{u,v}$ 替换为共享伪随机数生成器的种子来减少通信。这些共享种子将通过让各方广播Diffie-Hellman公钥并参与密钥协议来计算。

处理退出的用户的一个方法是通知全部用户有哪些用户退出了，然后其余的用户用他们与退出用户之间共有的随机种子进行回复。但有可能在回复种子前又会有用户退出，如此循环下去...

本文通过使用门限秘密共享方案并让每个用户向所有其他用户发送他们的Diffie-Hellman秘密份额来解决这个问题。这允许即使其他各方在恢复期间退出，只要某些最小数量的各方(等于阈值)仍然在线并且用丢弃的用户密钥的份额进行响应，也可以恢复成对种子。

## Double-Masking to Protect Security

双重盲化是为了防止用户发送信息过慢导致服务器以为他离线所产生的安全问题。

具体操作就是在使用向量 $\boldsymbol{s}_{u,v}$ 进行盲化的时候也把自己生成的向量加入：
$$
\begin{aligned}
\boldsymbol{y}_{u}=\boldsymbol{x}_{u} &+\mathbf{P R G}\left(\boldsymbol{b}_{u}\right) \\
&+\sum_{v \in \mathcal{U}: u<v} \mathbf{PRG}\left(s_{u, v}\right) \\
&-\sum_{v \in \mathcal{U}: u>v} \mathbf{PRG}\left(s_{v, u}\right) \quad(\bmod R)
\end{aligned}
$$

## Putting it all Together

本文协议的大体步骤：

![](http://images.yingwai.top/picgo/20201222143200.png)

本文协议的开销：

![](http://images.yingwai.top/picgo/20201222143305.png)



# A PRACTICAL SECURE AGGREGATION PROTOCOL

![](http://images.yingwai.top/picgo/20201222143821.png)

**标红的部分为保证主动安全性的操作，在半诚实设置下不是必要的。**



## Setup

![](http://images.yingwai.top/picgo/20201222144012.png)

设置阶段主要是各种参数的设置，如安全参数 $k$、用户数量 $n$、门限值 $t$ 等等。然后所有的用户都有一条与服务器交互的安全信道。

所有用户从可信第三方中获得签名密钥 $d^{SK}_u$ 以及与其他用户身份 $v$ 绑定的验证密钥 $d^{PK}_v$。



## Round 0 (AdvertiseKeys)

![](http://images.yingwai.top/picgo/20201222144441.png)

对于用户 $u$：

* 生成两对公私钥对 $c_u$ 和 $s_u$，然后使用签名密钥对这两个公钥进行签名得到 $\sigma_u$
* 然后把三者拼起来发送给服务器

对于服务器：

* 从不同的用户中收集到至少 $t$ 个消息（用户集合表示为 $\mathcal{U}_1$），否则中止协议
* 对 $\mathcal{U}_1$ 中的每个用户广播列表 $\{ (v, c^{PK}_v, s^{PK}_v, \sigma_v) \}_{v\in \mathcal{U}_1}$



## Round 1 (ShareKeys)

![](http://images.yingwai.top/picgo/20201222145435.png)

对于用户 $u$：

* 从服务器广播中收到列表 $\{ (v, c^{PK}_v), s^{PK}_v), \sigma)v \}_{v \in \mathcal{U}_1}$。判断 $|\mathcal{U}_1| \geq t$、所有公钥都不同、以及 $\forall v \in \mathcal{U}_{1}, \mathbf{ SIG.ver }\left(d_{v}^{P K}, c_{v}^{P K} \| s_{v }^{P K}, \sigma_{u}\right)=1$

对于服务器：

* 从不同的用户中收集到至少 $t$ 个消息（用户集合表示为 $\mathcal{U}_1$），否则中止协议
* 随机选取一个元素 $b_u \leftarrow \mathbb{F}$（作为 $\mathbf{PRG}$ 的种子）
* 生成 $s_{v}^{S K}$ 的 $t\text{-out-of-}|\mathcal{U}_1|$ 份额：$\left\{\left(v, s_{u, v}^{S K}\right)\right\}_{v \in \mathcal{U}_{1}} \leftarrow \mathbf { SS.share }\left(s_{u}^{S K}, t, \mathcal{U}_{1}\right)$
* 生成 $b_u$ 的 $t\text{-out-of-}|\mathcal{U}_1|$ 份额：$\left\{\left(v, b_{u,v}\right)\right\}_{v \in \mathcal{U}_{1}} \leftarrow \mathbf { SS.share }\left(b_{u,v}, t, \mathcal{U}_{1}\right)$
* 对于每个用户 $v \in \mathcal{U}_1 \backslash \{u\}$，计算 $e_{u,v} \leftarrow \mathbf{AE.enc}(\mathbf{KA.agree}(c^{SK}_u, c^{PK}_u), u \| v \| s^{SK}_{u,v} \| b_{u,v})$
* 如果上面任何步骤失败，中止协议
* 发送所有的密文 $e_{u,v}$ 给服务器（每个隐含地包含寻址信息 $u,v$ 作为元数据）
* 保存所有在本轮收到或生成的信息

对于服务器：

* 从不同的用户中收集到至少 $t$ 个消息（用户集合表示为 $\mathcal{U}_2 \subseteq \mathcal{U}_1$）
* 对 $\mathcal{U}_2$ 中的每个用户广播列表 $\{ e_{u,v} \}_{v\in \mathcal{U}_2}$



## Round 2 (MaskedInputCollection)

![](http://images.yingwai.top/picgo/20201222194737.png)

对于用户 $u$：

* 收到（以及保存）从服务器那里收到的密文列表 $\{ e_{u,v} \}_{v \in \mathcal{U}_2}$（并推导出集合 $\mathcal{U}_2$）。如果列表的大小 $< t$，中止协议
* 对于每个用户 $v \in \mathcal{U}_2 \backslash \{u\}$，计算 $s_{u,v} \leftarrow \mathbf{KA.agree}(s^{SK}_u, s^{PK}_v)$ 然后使用 $\mathbf{PRG}$ 将该值扩展成一个随机向量 $\boldsymbol{p}_{u,v} = \Delta_{u,v} \cdot \mathbf{PRG}(s_{u,v})$，其中当 $u > v$ 时 $\Delta_{u,v} = 1$，当 $u < v$ 时 $\Delta_{u,v} = -1$（注意到 $\boldsymbol{p}_{u,v} + \boldsymbol{p}_{v,u} = 0 \ \  \forall u \neq v$）。另外定义 $\boldsymbol{p}_{u,u} = 0$
* 计算用户自己的私有盲化向量 $\boldsymbol{p}_u = \mathbf{PRG}(b_u)$。然后计算盲化后的输入向量 $\boldsymbol{y}_{u} \leftarrow \boldsymbol{x}_{u}+\boldsymbol{p}_{u}+\sum_{v \in \mathcal{U}_{2}} \boldsymbol{p}_{u, v}(\bmod R)$
* 如果上面任何步骤失败，中止协议。否则将 $\boldsymbol{y}_u$ 发送给服务器

对于服务器：

* 从不同的用户中收集到至少 $t$ 个 $\boldsymbol{y}_u$（用户集合表示为 $\mathcal{U}_3 \subseteq \mathcal{U}_2$）。发送 $\mathcal{U}_3$ 中的用户列表给 $\mathcal{U}_3$ 中的所有用户



## Round 3 (ConsistencyCheck)

![](http://images.yingwai.top/picgo/20201222195915.png)

对于用户 $u$：

* 从服务器收到包括至少 $t$ 个用户的列表 $\mathcal{U}_3 \subseteq \mathcal{U}_2$（包括他自己）。如果 $\mathcal{U}_3$ 大小小于 $t$，中止协议
* 发送 $\sigma'_u \leftarrow \mathbf{SIG.sign}(d^{SK}_u, \mathcal{U}_3)$

对于服务器：

* 从至少 $t$ 个用户处收集 $\sigma'_u$（用户集合表示为 $\mathcal{U}_4 \subseteq \mathcal{U}_3$），发送集合 $\{v, \sigma'_v \}_{v \in \mathcal{U}_4}$ 给 $\mathcal{U}_4$ 中的所有用户



## Round 4 (Unmasking)

![](http://images.yingwai.top/picgo/20201222222442.png)

对于用户 $u$：

* 从服务器处收到列表 $\{ v, \sigma'_v \}_{v \in \mathcal{U}_4}$。验证是否满足 $\mathcal{U}_4 \subseteq \mathcal{U}_3$、$|\mathcal{U}_4| \geq t$ 以及对于所有 $v \in \mathcal{U}_4$ 有 $\mathbf{SIG.ver}(d^{PK}, \mathcal{U}_3, \sigma'_v) = 1$（否则中止协议）
* 对于每个用户 $v \in \mathcal{U}_2 \backslash \{u\}$，解密从**MaskedInputCollection**步骤中收到的密文 $v' \| u' \| s^{SK}_{v,u} \| b_{v,u} \leftarrow \mathbf{AE.dec}(\mathbf{KA.agree}(c^{SK}_u, c^{PK}_v), e_{v,u})$ 然后定义 $u = u' \and v = v'$
* 如果任何解密操作失败，中止协议
* 发送份额列表给服务器，份额包括了用户 $v \in \mathcal{U}_2 \backslash \mathcal{U}_3$ 的 $s^{SK}_{u,v}$ 以及用户 $v \in \mathcal{U}_3$ 的 $b_{v,u}$

对于服务器：

* 收集至少 $t$ 个用户的响应（用户集合表示为 $\mathcal{U}_5$）
* 对于每个用户 $u \in \mathcal{U}_2 \backslash \mathcal{U}_3$，重构 $s_{u}^{S K} \leftarrow \mathbf{SS.recon}\left(\left\{s_{u, v}^{S K}\right\}_{v \in \mathcal{U}_{5}}, t\right)$ 然后使用它（以及在**AdvertiseKeys**步骤收到的公钥）使用 $\mathbf{PRG}$ 来计算所有 $v \in \mathcal{U}_3$ 的 $\boldsymbol{p}_{v,u}$
* 对于每个用户 $u \in \mathcal{U}_3$，重构 $b_u \leftarrow \mathbf{SS.recon}\left(\left\{ b_{u,v} \right\}_{v \in \mathcal{U}_{5}}, t\right)$ 然后使用 $\mathbf{PRG}$ 来计算 $\boldsymbol{p}_{u}$
* 根据 $\sum_{u \in \mathcal{U}_{3}} \boldsymbol{x}_{u}=\sum_{u \in \mathcal{U}_{3}} \boldsymbol{y}_{u}-\sum_{u \in \mathcal{U}_{3}} \boldsymbol{p}_{u}+\sum_{u \in \mathcal{U}_{3}, v \in \mathcal{U}_{2} \backslash \mathcal{U}_{3}} \boldsymbol{p}_{v, u}$ 计算然后输出 $z=\sum_{u \in \mathcal{U}_{3}} \boldsymbol{x}_{u}$