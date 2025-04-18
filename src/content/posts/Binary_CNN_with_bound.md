---
title: Binary_CNN_with_bound
published: 2025-04-18
description: ''
image: ''
tags: [binary CNN, limit bound]
category: Binary CNN
draft: false 
lang: 'zh'
---

如何提高binaryCNN的准确性，这边文章是来自NIPS的Towards Accurate Binary Convolutional Neural
这篇文章有点旧了，时间是NIPS 2017，作者有Wei Pan,因此我可以在曼大找到他，我希望不会有太大的问题，我希望的是如何使用里面的bound 方法去利用bound,因为esbmc 同样使用了bound去找bug,这或许在神经网络训练也可以apply esbmc？


首先来看Abstract:
本文提到他们的贡献是introduce a novel scheme to train binary CNN. binary CNN是一种将原本float或者double的概率或者权重改成[-1,1]的binary CNN.好处是可以reduce memory size accesses.并且用bitwise operations代替了arithmetic operations.作者提到，好处是可以提高计算速度，并且减少功耗。对CNN进行binarization会造成预测精度的严重下降。
本文提出两种方法去解决它。
- 使用多个二值权重基的线性组合来表示近似全精度权重。
- 使用多个二值激活函数来减少信息丢失。
结果是正在ImageNet和forest trail数据集上的表现和全精度CNN的预测精度相当。


## 第一个方法并不难理解
方程如下：
$$
    W = \sum_{i=1}^{k} \alpha_i b_i
$$
这里面W是一个全精度权重，而b_i是二进制权重，而alpha是系数。这样如果我们能构造一个a_i可以使得W=B\alpha,那么我们就可以用二进制权重去表示全精度权重。

### 如何计算这个二进制权重来表示全精度权重
我们得到一个公式，应该让这样一个二进制权重去尽可能的靠近W
$$
\min_{\alpha,B} J(\alpha,B) = ||w-B\alpha||^2 , s.t B_(ij) \in \{-1,1\}
$$
这个是W和B的方差。可是直接要求解这个公式是很难的，因此作者提出了对B_i进行一些简化。
    1.用一个固定B_i,可以通过对W进行标准化来生成：
        $$B_i = F_{u_i}(W) := \text{sign}(\overline{W} + u_i \cdot \text{std}(W))$$
        其中
        $$\overline{W} = W-mean(W)$$
        
将B_i代入到公式中，可以得到：
$$
    \min_{\alpha} J(\alpha) = ||W-\sum_{i=1}^{k} \alpha_i F_{u_i}(W)||^2
$$
因此，我们可以通过求解这个公式来得到alpha。

alpha的计算公式可以转化成
$$
    \alpha = \frac{W^T B}{B^T B}
$$
