---
layout: post
title:  "GAN的基础理论"
date:   2020-10-13 15:35:10 +0800
tags: 机器学习
color: rgb(255,90,90)
cover: '../assets/gan_basic_theory/mnist.jpg'
subtitle: '李宏毅GAN课程笔记'
---


## 任务是什么？

假设要生成二次元头像图片。

每一个二次元头像图片都是一个高维空间中的点。

二次元头像图片有一个固定的分布 （distribution），记作$P_{data}(x)$。



![jpg]({{site.url}}\assets\gan_basic_theory\p_data_distribution.jpg)

上图表示二次元头像的分布$P_{data}(x)$，图中蓝色区域表示大概率为二次元头像图片的区域，在其他区域的点是二次元头像图片的概率小。

$P_{data}(x)$是未知的分布，但是数据集中的每一个二次元头像是服从这个分布。



**任务是找出一个分布来拟合$P_{data}(x)$**。



## 可以用常见的数学分布拟合吗？

很自然地可以想到通过MLE用一个参数为$\theta$的分布$P_G(x;\theta)$来拟合$P_{data}(x)$。

比如$P_G(x;\theta)$是高斯分布，$\theta$为期望和方差。

从$P_{data}(x)$中抽样得到样本集$X = \{ x^1,x^2,...,x^m \} $。

似然函数为：

$$
L=\prod_{i=1}^m P_G(x^i;\theta)
$$

对数似然函数为：

$$
L = \sum_{i=1}^m \log P_G(x^i;\theta)
$$

极大似然函数求解$\theta$：

$$
\theta ^* = \mathop {argmax}_\theta \sum_{i=1}^m \log P_G(x^i;\theta)
$$


**此时，MLE等价于最小*K-L Divergence***。



证明：

由于$P_{data}(x)$与$\theta$无关，故

$$
\begin{align}
\theta ^* &= \mathop {argmax}_\theta \sum_{i=1}^m P_{data}(x^i)\log P_G(x^i;\theta)\\
&=\mathop {argmax}_\theta E_{X\sim P_{data}}[\log P_G(X;\theta)]

\end{align}
$$

把期望写成积分的形式

$$
\begin{align}
\theta ^* &=\mathop {argmax}_\theta \int_x P_{data}(x)\log P_G(x;\theta)dx
\end{align}
$$

由于$P_{data}(x)$与$\theta$无关，上式可加一无关项

$$
\begin{align}
\theta ^* &=\mathop {argmax}_\theta \int_x P_{data}(x)\log P_G(x;\theta)dx-\int_x P_{data}(x)\log P_{data}(x)dx\\
&=\mathop {argmax}_\theta\int_x P_{data}(x) \log\frac{P_G(x;\theta)}{P_{data}(x)}dx\\
&=\mathop {argmax}_\theta [-\int_x P_{data}(x) \log\frac{P_{data}(x)}{P_G(x;\theta)}dx]\\
&=\mathop {argmin}_\theta D_{KL}(P_{data}||P_G)
\end{align}
$$

证毕。



在上世纪80年代已经有用混合高斯模型（Gaussian Mixture Model）来拟合某一类图片的分布的工作。不过得到的图片都很模糊，可能的原因是某一类图片的分布是高维空间中的一个低维 manifold（流形分布定律），而混合高斯模型不是 manifold，故无法很好的拟合。



## GAN的做法

![jpg]({{site.url}}\assets\gan_basic_theory\gan_gen.jpg)



GAN 用一个神经网络生成器来模拟复杂的分布。

生成器 G 可以看作是一个函数，输出 G(z) 为图片。

目标是优化 G 的参数使得输出$x=G(z)$的分布$P_G(x)$和真实分布$P_{data}(x)$的散度最小。



难点是$P_G(x)$和$P_{data}(x)$都是未知分布。

GAN 计算$P_G(x)$和$P_{data}(x)$的散度的思路是：

![jpg]({{site.url}}\assets\gan_basic_theory\div.jpg)

虽然$P_G(x)$和$P_{data}(x)$未知，但是可以分别从中取样。

然后用 Discriminator 来对样本进行分类。

![jpg]({{site.url}}\assets\gan_basic_theory\discriminator.jpg)

Discriminator 给不同的样本打分，当样本属于$P_{data}(x)$时打高分，当样本属于$P_G(x)$时打低分。

样本$x$得分为$D(x)\in(0,1)$。

$$
V(G,D)=E_{x\sim P_{data}}[\log D(x)]+E_{x\sim P_G}[\log (1-D(x))]
$$

$D(x)$越大，$\log D(x)$越大；

$D(x)$越小，$\log (1-D(x))$越大。

显然$V$的值越大，Discriminator 的分类效果越好。

优化 Discriminator 的参数使得$V$最大，即：

$$
D^*=\mathop {argmax}_D V
$$

$V$的最大值与*Jensen-Shannon divergence*相关。

证明：

$$
\begin{align}
V&=E_{x\sim P_{data}}[\log D(x)]+E_{x\sim P_G}[\log (1-D(x))]\\
&=\int_xP_{data}(x)\log D(x)dx+\int_xP_G(x)\log (1-D(x))dx\\
&=\int_x[P_{data}(x)\log D(x)+P_G(x)\log (1-D(x))]dx\\
&=\int_x f(D)dx

\end{align}
$$


假设$D(x)$可以为任意的 function，即每一个$x$都可以对应一个$D(x)$得到$f(D)$的值。

要使$V$最大即对于每一个给定的$x$使其对应的$f(D)$最大。

$$
f(D)=P_{data}(x)\log D+P_G(x)\log (1-D)
$$

$$
\frac{\partial f(D)}{\partial D}=P_{data}(x)\frac{1}{D}-P_G(x)\frac{1}{1-D}=0
$$


易得

$$
D^*(x)=\frac{P_{data}(x)}{P_{data}(x)+P_G(x)}
$$

则

$$
0<D^*(x)<1
$$


将$D^*(x)$带入$V$

$$
\begin{align}
\mathop{max}_DV&=\int_xP_{data}(x)\log \frac{P_{data}(x)}{P_{data}(x)+P_G(x)}dx+\int_xP_G(x)\log (1-\frac{P_{data}(x)}{P_{data}(x)+P_G(x)})dx\\
&=\int_xP_{data}(x)\log \frac{P_{data}(x)}{P_{data}(x)+P_G(x)}dx+\int_xP_G(x)\log \frac{P_{G}(x)}{P_{data}(x)+P_G(x)}dx\\
&=\int_xP_{data}(x)\log \frac{\frac{1}{2}P_{data}(x)}{\frac{1}{2}(P_{data}(x)+P_G(x))}dx+\int_xP_G(x)\log \frac{\frac{1}{2}P_{G}(x)}{\frac{1}{2}(P_{data}(x)+P_G(x))}dx\\
&=2\log\frac{1}{2}+\int_xP_{data}(x)\log \frac{P_{data}(x)}{\frac{1}{2}(P_{data}(x)+P_G(x))}dx+\int_xP_G(x)\log \frac{P_{G}(x)}{\frac{1}{2}(P_{data}(x)+P_G(x))}dx\\
&=-2\log 2+D_{KL}(P_{data}||\frac{P_{data}+P_G}{2})+D_{KL}(P_{G}||\frac{P_{data}+P_G}{2})\\
&=-2\log 2+D_{JS}(P_{data}||P_G)

\end{align}
$$

证毕。

故GAN的目标：优化 G 的参数使得输出$x=G(z)$的分布$P_G(x)$和真实分布$P_{data}(x)$的散度最小，可以写做：

$$
G^*=arg\mathop {min}_G \mathop {max}_DV(G,D)
$$

![jpg]({{site.url}}\assets\gan_basic_theory\min_max.jpg)



解$G^*$的算法：

![jpg]({{site.url}}\assets\gan_basic_theory\alg.jpg)

Loss Function：

$$
L(G)=V(G,D^*)
$$

$\theta _G$是$G$的参数，用梯度下降最小化$L(G)$：

$$
\theta _G \leftarrow \theta _G - \eta \frac{\partial L(G)}{\partial \theta _G}
$$

- 初始化$G_0$

- 用梯度上升最大化$V(G_0,D)$得到$D_0^*$（$V(G_0,D_0^*)$即$P_{data}(x)$与$P_{G_0}(x)$之间的*J-S divergence*）

- 用梯度下降最小化$L(G)$得到$G_1$
- 用梯度上升最大化$V(G_1,D)$得到$D_1^*$（$V(G_1,D_1^*)$即$P_{data}(x)$与$P_{G_1}(x)$之间的*J-S divergence*）

- 用梯度下降最小化$L(G)$得到$G_2$

![jpg]({{site.url}}\assets\gan_basic_theory\alg1.jpg)

如果$G_0$和$G_1$差距很大，则 *J-S divergence* 可能不下降反而上升，故每一次迭代不能更新$G$太多步。



## 实际算法

![jpg]({{site.url}}\assets\gan_basic_theory\alg2.jpg)

![jpg]({{site.url}}\assets\gan_basic_theory\alg3.jpg)

由于$\widetilde{V}$中只有第二项与$G$有关，故更新$\theta _g$仅计算第二项即可。

![jpg]({{site.url}}\assets\gan_basic_theory\alg4.jpg)

由于$\log (1-D(x))$在优化开始的时候有梯度小的问题，故可以用$-\log D(x)$来替代。

替换之后，$\widetilde{V}$的表达式为：

$$
\widetilde{V}=\frac{1}{m}\sum_{i=1}^m\log D(x^i)-\frac{1}{m}\sum_{i=1}^m\log D(G(z^i))
$$
