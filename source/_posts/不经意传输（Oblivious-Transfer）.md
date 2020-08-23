---
title: 不经意传输（Oblivious Transfer）
date: 2020-04-25 16:22:21
categories: Study
tags: [密码学, MPC]
---

*设S有一个秘密，想以1/2的概率传递给R，即B有50%的机会收到这个秘密，另外50%的机会什么也没有收到，协议执行完后，B知道自己是否收到了这个秘密，但S却不知道R是否收到了这个秘密。这种协议就称为不经意传输协议。*

<!--more-->

​		例如A是机密的出售者，S列举了很多问题，意欲出售各个问题的答案，R想买其中一个问题的答案，但又不想让A知道自己买的是哪个问题的答案。



## 1-out-of-2 OT

OT最早在1981年被 Michael O. Rabin提出，在Rabin的OT协议中，发送者S发送一个信息m给接收者R，接收者R以1/2的概率接受信息m。所以在协议交互的结束的时候，S并不知道R是否接受了消息。该方案是基于RSA加密体系构造的。

1985年S. Even, O. Goldreich, and A. Lempel提出了1-out-2 OT,在新的方案中S每次发送2个信息 $m_0$ 和 $m_1$，而R每次输入一个选择 $b$。当协议结束的时候，S无法获得关于 $b$ 的任何有价值的信息，而R只能获得 $m_b$ ，对于 $m_{1-b}$ ，R也一无所知。
![](http://images.yingwai.top/picgo/OTf1.png)

## 协议

S要传送两条信息，不妨设为两个比特 $b_0$ 和 $b_1$，而R只能选择接受其中一个比特 $b_{\theta}$。协议要保证S和R的安全，即：

* S不能知道任何关于 $\theta$ 的信息；
* R不能知道任何关于 $b_{1-\theta}$ 的信息。

为了达成上面两点要求，构造如下协议：

1. S选择好两个比特信息 $b_0$ 和 $b_1$；

2. S运行密钥生成算法（例如RSA）生成公私钥对（$sk,pk$）；

3. S保密私钥，同时S要生成两个随机数 $x_0$ 和 $x_1$，并将这两个随机数和公钥一并传送给R；

4. R选择 $\theta$，并生成一个数 $r$，R用S的公钥加密 $r$，并生成信息

   ​								$$v =x_{\theta}+E_{pk}(r)$$

5. S在接收到 $v$ 之后，进行如下计算：

   ​								$r_{0}=D_{s k}\left(v-x_{0}\right)$
   ​								$r_{1}=D_{s k}\left(v-x_{1}\right)$

6. S进行如下计算：

   ​								$b_{0}^{\prime}=b_{0}+r_{0}$
   ​								$b_{1}^{\prime}=b_{1}+r_{1}$

   并将 $b'_0$ 和 $b'_1$ 传送给R；

7. 接收到 $b'_0$ 和 $b'_1$ 后，R进行如下计算：

   ​								$b_\theta = b'_\theta - r$



### 正确性

进行如下推导：

​								$b_{\theta}^{\prime}-r=b_{\theta}+r_{\theta}-r=b_{\theta}+D_{s k}\left(v-x_{\theta}\right)-r$

而

​						$D_{s k}(v-x_\theta)=D_{s k}\left(x_\theta+E_{p k}(r)-x_{\theta}\right)=D_{s k}\left(E_{p k}(r)\right)=r$

因此可得

​											$b'_\theta - r = b_\theta$



### 对于S的安全性

对于S来说，它的安全性要求R不能推断出 $b_{1-\theta}$。R得到 $b'_\theta$ 和 $b'_{1-\theta}$ 之后，由协议可知R可以正确计算 $b'_\theta$ ，下面证明R不能计算出 $b_{1-\theta}$。

​						$b_{1-\theta}^{\prime}-r=b_{1-\theta}+D_{s k}\left(v-x_{\theta}\right)-r$

而

​					$D_{s k}\left(v-x_{\theta}\right)=D_{s k}\left(x_{1-\theta}+E_{p k}(r)-x_{\theta}\right) \neq r$

而且根据加密的特性，$D_{s k}\left(x_{1-\theta}+E_{p k}(r)-x_{\theta}\right)-r$ 与随机数是不可区分的，所以 $b'_{1-\theta}-r$ 与随机数是不可区分的。因此协议对S是安全的。



### 对于R的安全性

R的安全性要求S不能获得关于 $\theta$ 任何有价值的信息。

因为 $r$ 是随机数，所以 $E_{pk}(r)$ 与随机数是不可区分的。因此 $v$ 与随机数是不可区分的，所以S不能从 $v$ 获得关于 $\theta$ 有价值的信息。

