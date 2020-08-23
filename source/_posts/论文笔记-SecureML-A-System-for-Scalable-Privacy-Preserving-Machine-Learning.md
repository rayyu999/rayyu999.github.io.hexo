---
title: '论文笔记 SecureML: A System for Scalable Privacy-Preserving Machine Learning'
date: 2020-06-17 11:25:47
categories: Papers
tags: [MPC, PPML, Neural Network, Logistic Regression, Linear Regression, Secret Sharing]
---

*Payman Mohassel,  Yupeng Zhang*

[IEEE Symposium on Security and Privacy 2017](https://dblp.uni-trier.de/db/conf/sp/sp2017.html#MohasselZ17)

https://eprint.iacr.org/2017/396.pdf

<!--more-->

# 文章贡献

* **两方计算：**第一个提出在两方计算模型下处理机器学习模型问题；
* **秘密分享：**第一个将秘密分享引入该问题，支持在两方上任意分布的数据；
* **Secure SGD：**在两方下提出了训练模型的安全SGD算法；
* **秘密截断：**分析了在秘密分享下如何进行定点数截断问题。



# 计算模型

本文考虑一个客户端集合 $\mathcal{C}_1,...,\mathcal{C}_m$ 想要使用它们的数据训练一个模型，客户端将数据 $x_i$ 以秘密分享的模式分发给两个非合谋的服务器 $\mathcal{S}_0, \mathcal{S}_1$（例如一方拥有随机数 $r_i$ 而另一方拥有 $x_i - r_i$），服务器交互式地计算函数 $f(x_i)$，得到预测标签 $y_i$。

![](http://images.yingwai.top/picgo/smlf3.png)

![](http://images.yingwai.top/picgo/smlppt1.png)



# PPML

本文给出了满足隐私保护的线性回归、逻辑回归以及神经网络的协议。



## 线性回归

$y = \sum^d_{j=1} x_{ij} w_j = \vec{x_i} \cdot \vec{w}$



**损失函数：**

$C(\mathbf{w}) = \frac{1}{n} \sum C_i(\mathbf{w})$，

其中 $C_i(\mathbf{w}) = \frac{1}{2}(\mathbf{x}_i \cdot \mathbf{w} - y_i)^2$。

更新权重的公式为：
$$
\begin{align}
w_j : &= w_j - \alpha \frac{\partial C_i(\mathbf{w})}{\partial w_j} \\
&= w_j - \alpha (\mathbf{x}_i \cdot \mathbf{w} - y_i) x_{ij}
\end{align}
$$
可以看到只有加法和乘法的运算。

数据是在两个服务器中加法共享的，计算两个数 $\langle a \rangle, \langle b \rangle$ 的和比较简单，两个服务器各自在本地将对应的份额相加即可；计算乘法则比较麻烦，需要借助乘法三元组 $(\langle u \rangle, \langle v \rangle, \langle z \rangle)$ 满足$z = uv \bmod 2^l$：

![](http://images.yingwai.top/picgo/smlppt2.png)

接下来就可以得到上述算法的向量版本，以下是文章给出的具体协议：

![](http://images.yingwai.top/picgo/smlf4.png)

为了提高效率，使用了batch的方法来更新权重。



## 逻辑回归

$g(\mathbf{x}_i) = f(\mathbf{x}_i \cdot \mathbf{w}) = \frac{1}{1+e^{-\mathbf{x}_i \cdot \mathbf{w}}}$

和线性回归相比，逻辑回归在线性回归的基础上增加了一个Sigmoid函数，其计算起来很困难，通常是使用多项式去拟合，但需要多项式次数较高且其并不收敛，导致在 $x$ 较大时准确率不高。

因此作者提出了他们称为Secure-computation-friendly activation function：
$$
f(x) = \left\{ \begin{array}{lcl}
0, & \mbox{if} & x < - \frac{1}{2} \\
x + \frac{1}{2}, & \mbox{if} & - \frac{1}{2} \leq x \leq \frac{1}{2} \\
1, & \mbox{if} & x > \frac{1}{2}
\end{array}\right.
$$
该函数的图像与Sigmoid函数类似：

![](http://images.yingwai.top/picgo/smlf5.png)

于是计算完矩阵-向量乘法后还需要计算上面的分段函数，因为要判断 $u+\frac{1}{2}$ 和 $u- \frac{1}{2}$ 的符号，这里算术共享就不是很适合，而Yao共享很适合，因此需要先把共享转换成Yao共享，在[ABY](https://yuyingwai.cn/2020/06/11/论文笔记-ABY-A-Framework-for-Efficient-Mixed-Protocol-Secure-Two-Party-Computation/)框架中有介绍。

这里作者将函数 $f(u)$ 改写成了如下形式，方便乱码电路的计算：
$$
f(u) = (\neg b_2) + (b_2 \wedge (\neg b_1))u
$$
其中 $b_1, b_2$ 分别为：
$$
b_1 = \left\{ \begin{array}{ll}
0, & \mbox{if} \ \  u+\frac{1}{2} \geq 0 \\
1, & \mbox{otherwise}
\end{array}\right.
$$

$$
b_2 = \left\{ \begin{array}{ll}
0, & \mbox{if} \ \  u-\frac{1}{2} \geq 0 \\
1, & \mbox{otherwise}
\end{array}\right.
$$

于是构造电路以 $\langle u \rangle_0 + \frac{1}{2}$ 和 $\langle u \rangle_1$ 作为输出，将它们相加并把结果的符号位 $b_1$ 输出，$b_2$ 同理，最后计算上面的 $f(u)$ 即可。下面是完整的协议：

![](http://images.yingwai.top/picgo/smlf13.png)



## 神经网络

神经网络可以看作很多个与逻辑回归计算方式一致的神经元组成，用上面逻辑回归中的方法处理即可。