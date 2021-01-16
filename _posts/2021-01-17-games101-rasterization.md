---
layout: post
title:  "光栅化"
date:   2021-01-17 04:46:30 +0800
tags: 图形学
color: rgb(255,90,90)
cover: '../assets/games101-rasterization/title.gif'
subtitle: '采样，反走样与深度缓存'
---

本文为课程[games101][link-1]第5课至第6课的课程笔记。

[link-1]: https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html


# 屏幕

屏幕可以看作是一个由**像素**构成的矩阵。矩阵的大小称作**分辨率**。像素是构成屏幕的最小元素，其物理实现方式多种多样，但在此文中我们将其抽象成一个可以显示**颜色**的方格。

![screen]({{site.url}}\assets\games101-rasterization\00.jpg)

上图是一个分辨率为 $4\times3$ 的屏幕。我们将其与屏幕空间对应，左下角的像素为 $(0,0)$，右上角的像素为 $(3,2)$，像素 $(x,y)$ 的中点为 $(x+0.5,y+0.5)$。对于一个分辨率为 $width\times height$ 的屏幕，其像素分布为从 $(0,0)$ 到 $(width-1,height-1)$，可映射的最大屏幕空间为 $[0,width]\times[0,height]$。

光栅（raster）一词源于德语，意为屏幕。光栅化（rasterize）就是将图像画在屏幕上的过程。

# 采样

采样是对一个函数进行**离散化**的过程。

采样在计算机图形学中是一个非常重要的概念。生活中有各种的采样，例如照片是相机用感光元件对光线采样，视频（动画）是时间方向上的采样。

这里我们讨论的是利用像素中心对屏幕空间进行的采样。

## 三角形

三角形是计算机图形学中的基本图元（fundamental shape primitive）。

- 三角形是最基础的多边形（polygon）
  - 任何多边形都可以被拆分成三角形
- 三角形的性质
  - 三角形一定是平面的
  - 三角形对于内外的定义清晰而简洁（向量的叉乘）
  - 确定了三角形的三个顶点后可以轻松地在三角形内部进行插值计算（重心插值）

## 采样过程

![2d sampling]({{site.url}}\assets\games101-rasterization\01.jpg)

一种简单直接的想法就是如果像素中心在三角形内则该像素对三角形进行采样。

![cross product]({{site.url}}\assets\games101-rasterization\02.jpg)

使用三次向量叉乘的方法判断像素中点是否在三角形内。

![edge cases]({{site.url}}\assets\games101-rasterization\03.jpg)

当像素中点恰好落在三角形的边上，或者如上图所示同时落在两个三角形共有的边上的时候，像素中点和三角形位置关系的判断规则由我们自己定义。

![bounding box]({{site.url}}\assets\games101-rasterization\04.jpg)

对于一个三角形的光栅化，并不需要遍历整个屏幕。可以先确定该三角形的**包围盒**（bounding box）。更严格的，此处的包围盒是轴向包围盒（axis aligned bounding box, AABB），即构成包围盒的边与坐标轴为水平（垂直）的关系。

还可通过增量遍历的方式，只考虑三角形内部的像素中心点。这种方法更适合对斜向放置的狭长三角形使用。

![incremental triangle traversal]({{site.url}}\assets\games101-rasterization\05.jpg)

## 走样

观察采样后得到的结果，我们可以发现屏幕上显示的三角形与被采样的三角形区别很大。这种差别即为**走样**（aliasing）。在光栅化中走样的具体表现就是**锯齿**（jaggies）。

![aliasing]({{site.url}}\assets\games101-rasterization\06.jpg)

走样是由采样而产生的，即 Aliasing 可以等同为 Sampling Artifacts。此处 Artifacts 为 Errors / Mistakes / Inaccuracies in Computer Graphics 之意。典型的 Sampling Artifacts 有锯齿，摩尔纹（Moiré），车轮效应（Wagon wheel effect）等。

Sampling Artifacts 产生的根本原因是**信号变化的频率太快而采样的频率过慢**。

![aliases]({{site.url}}\assets\games101-rasterization\07.jpg)

这里给出走样更加正规的定义：同样的采样方法对两种不同频率的函数的采样结果无法区分。

# 傅里叶变换

## 傅里叶级数

简单来说，任何周期函数都可以用正弦函数和余弦函数构成的无穷级数来表示。

![Fourier series]({{site.url}}\assets\games101-rasterization\08.jpg)
$$
f(x)=\frac{A}{2}+\frac{2A\text{cos}(tw)}{\pi}-\frac{2A\text{cos}(3tw)}{3\pi}+\frac{2A\text{cos}(5tw)}{5\pi}-\frac{2A\text{cos}(7tw)}{7\pi}+\cdots
$$

## 一维傅里叶变换

傅里叶变换可以将函数从时域空间转换至频域空间。其过程可以理解为将函数写成傅里叶级数的形式，再将傅里叶级数映射到频域空间。频域空间的横轴为频率，纵轴为振幅。

![ft]({{site.url}}\assets\games101-rasterization\09.jpg)

变换公式为：
$$
F(\omega)=\int^{\infty}_{-\infty}f(x)e^{-2\pi iwx}dx
$$
其中：$e^{ix}=\text{cos}x+i\text{sin}x$（欧拉公式）

傅里叶变换实质上是基的变换：
$$
\bf Mf(x)=F(\omega)​
$$


![a change of bias]({{site.url}}\assets\games101-rasterization\10.png)

由频域变为时域的逆变公式为：
$$
f(x)=\int^{\infty}_{-\infty}F(\omega)e^{2\pi iwx}d\omega
$$

## 二维傅里叶变换

将一维的傅里叶变换扩展到二维的情况。

![2dft]({{site.url}}\assets\games101-rasterization\11.jpg)

二维傅里叶变换将一个图像分解成若干个复平面波 $e^{i2\pi(ux+vy)}$ 的叠加。与复指数波相比，复平面波不仅有频率 $w$，幅度 $A$，相位 $\varphi$ 三个属性，还有方向 $\vec n$。

![direction]({{site.url}}\assets\games101-rasterization\12.gif)

复平面波的表达式 $e^{i2\pi(ux+vy)}$ 中，$(u,v)$ 表示波的方向 $\vec n$，$\sqrt{u^2+v^2}$ 表示波的频率 $\omega$。则可以将所有的复平面波在**二维频率域**（K-Space）中表示。

![k-space]({{site.url}}\assets\games101-rasterization\13.jpg)

图片可以分解成二维频率域中元素的线性组合。

![photo]({{site.url}}\assets\games101-rasterization\14.jpg)

如果一个图片在频域的线性组合表达式中某一个复平面波的系数为 $0$，则其在二维频率域的位置为黑色。相对的，一个复平面波在线性组合表达式中的系数越大，其在二维频率域的位置越亮。

如果将一张图片的高频复平面波去除，那么这张图片会变得模糊。如果将其低频复平面波去除，则会仅保留图片中物体的轮廓。

![photo]({{site.url}}\assets\games101-rasterization\15.jpg)

将某些特定的复平面波从图像中去除的操作叫做**滤波**。

## 卷积

此文中我们不谈论严格定义的卷积（convolution）而只考虑离散情况的卷积。

先看一维的卷积。一维的卷积核可以看作为一个滑动窗口。

![sliding window]({{site.url}}\assets\games101-rasterization\16.jpg)

原始信号为 
$\{1,3,5,3,7,1,3,8,6,4\}$
，卷积核为 
$\{\frac{1}{4},\frac{1}{2},\frac{1}{4}\}$
。卷积核初始位置为首部与原始信号首部对齐，对应元素相乘后相加得到卷积结果的第一个元素值。

![result]({{site.url}}\assets\games101-rasterization\17.jpg)

然后卷积核向后移动一格，首部与原始信号的第二个元素对齐。

![sliding]({{site.url}}\assets\games101-rasterization\18.jpg)

对应元素相乘后相加得到卷积结果的第二个元素值。

![result]({{site.url}}\assets\games101-rasterization\19.jpg)

扩展至二维：

![2dConvolution]({{site.url}}\assets\games101-rasterization\20.gif)

这里引入卷积定理：**在空间域中的卷积等于在频率域中的乘积，反之亦然**。

![convolution theorem]({{site.url}}\assets\games101-rasterization\21.jpg)

频域的乘积即频域中对应位置的值相乘，若该位置为黑色则值为 $0$。卷积核在频域中大部分位置值都为 $0$，就好像一个筛子把原信号在其频域非零位置的复平面波筛出，故卷积核也称为**滤波器**（filter）。上图中的滤波器如果用图像来表示就是一个小方格，我们称它为盒滤波器（box-filter）。

![box-filter]({{site.url}}\assets\games101-rasterization\22.jpg)

如果将盒滤波器的尺寸放大，那么其在频域上非零的区域会缩小。作用在图片后得到的结果会更加模糊。一个直观的理解为如果将盒滤波器的尺寸放大至与被作用的图片相同，那么得到的结果的每一个像素的值都为原图的所有像素值的平均，而如果其尺寸只有一个像素，则得到的结果与原图相同。

![large size box]({{site.url}}\assets\games101-rasterization\23.jpg)

## 混叠

现在，我们重新来看采样。用狄拉克梳状函数对一个连续函数进行采样。

![sampling]({{site.url}}\assets\games101-rasterization\24.jpg)

狄拉克梳状函数在时域和频域都是脉冲状的（如图 c，d 所示）。由于时域的乘积等于频域的卷积，采样后信号的频谱为信号的频谱和狄克拉梳状函数卷积的结果。信号的频谱被周期延拓了（如图 f 所示），延拓的周期为采样频率，即狄卡拉梳状函数时间间隔的倒数。**采样的过程就是重复原始信号频谱的过程**。

注意此时原信号的最大频率 $f_0$ 小于采样频率 $f_s$ 的一半，满足奈奎斯特采样定理：为了不失真地恢复模拟信号，采样频率应该大于模拟信号频谱中最高频率的2倍。

![mixed frequency contents]({{site.url}}\assets\games101-rasterization\25.jpg)

如果违背了奈奎斯特采样定理，采样后信号的频率就会重叠，即高于采样频率一半的频率成分将被重建成低于采样频率一半的信号。这种频谱的重叠导致的失真称为**混叠**。

**光栅化中的走样现象的本质就是混叠**。

# 反走样

## 抗混叠滤波

将原信号的高频信号削去即可避免混叠，即对原信号进行低通滤波后再取样。

![limiting then repeating]({{site.url}}\assets\games101-rasterization\26.jpg)

最简单的实现为使用单像素盒滤波器。计算每一个像素中在三角形内部部分和其他部分的平均值作为整个像素的值。

![one pixel box]({{site.url}}\assets\games101-rasterization\27.jpg)

在使用像素对屏幕空间进行采样前，先使用单像素盒滤波器对屏幕空间图像进行低通滤波，然后再对滤波后的信号使用像素进行采样。

![Antialiased Sampling]({{site.url}}\assets\games101-rasterization\28.jpg)

## 超采样

超采样（Multi-Sampling Anti-Aliasing, MSAA）是另一种反走样方法，其基本思路是增加单个像素内采样点的个数。

考虑使用一个分辨率为 $7\times 4$ 的屏幕对一个三角形进行采样的情况。

![aliasing]({{site.url}}\assets\games101-rasterization\29.jpg)

使用 $2\times 2$ 的超采样，每一个像素中有四个采样点。

![averaging down]({{site.url}}\assets\games101-rasterization\30.jpg)

采样后再将每个像素内的采样点做平均得到最后的结果。

![sampling]({{site.url}}\assets\games101-rasterization\31.jpg)

超采样的方式增加了采样过程的计算量，且增加的计算量的大小看起来是非常直观的，即 $2\times 2$ 的超采样的计算量为原计算量的四倍。在工业应用上，人们使用特定的分布来设置采样点的位置，某些采样点会被不同的像素复用。这样就减少了计算量的增量。

## 其他方法

- 快速近似抗锯齿（FXAA, Fast Approximate AA）：使用图像匹配的方式将带有锯齿的边界找到，然后将其替换为没有锯齿的图像，计算速度非常快。
- 时间性抗锯齿（TAA, Temporal AA）：在时间维度上进行超采，如果相邻的两帧屏幕空间的画面没有改变，则在这两帧中分别使用像素内不同的位置进行采样。
- 超分辨率（Super Resolution / Super Sampling）：超分辨率是将小分辨率的图片放大并消除放大后图片锯齿的过程。将图片放大的过程与将大分辨率图像使用低分辨率屏幕采样并进行超采样抗锯齿的过程的问题相似，即样本不足。可以通过深度学习（Deep Learning Super Sampling, DLSS）来处理此种问题。

# 深度缓存

此处假设 $z$ 值都为正且 $z$ 值越小距离摄像机越近。

添加一个深度缓存专门来存储像素当前的最小 $z$ 值。具体地，所有像素的深度缓存的初始化都为无限大，依次序对三角形进行采样并在深度缓存中更新每个像素的最小 $z$ 值。时间复杂度为 $O(n)$，$n$ 为三角形的个数（假设对单个三角形采样的时间复杂度为常数）。

![z-buffer algorithm]({{site.url}}\assets\games101-rasterization\32.jpg)

每生成一张图片都有一个与之对应的深度缓存，可以将其以灰度图的形式具象化，$z$ 值越小的像素颜色越深。

![z-buffer]({{site.url}}\assets\games101-rasterization\33.jpg)

