---
layout: post
title:  "Blender基础 Level2"
date:   2020-12-02 00:30:00 +0800
tags: Blender
color: rgb(255,90,90)
cover: '../assets/blender-guru-donut-part2/final.jpg'
subtitle: '给甜甜圈加点细节'
---
Blender入门，内容为 [Blender Guru][link-1] 的 [tutorial][link-2] 的第二部分。

[link-1]: https://www.youtube.com/channel/UCOKHwx1VCdgnxwbjyb9Iu1g
[link-2]: https://www.youtube.com/watch?v=NyJWoyVx_XI&list=PLjEaoINr3zgEq0u2MzVgAaHEBt--xLB6U&index=1&ab_channel=BlenderGuru

## 内容梗概

粒子（Particles）

展 UV（UV Unwrapping）

绘制纹理（Texture Painting）

程序式纹理（Procedural Texturing）



## 粒子（Particles）

**在糖霜上撒糖！**

先捏出糖果的基础形状，添加一些长短或弯曲的细节，将这些 Objects 合并成为一个 Collection。

![spks]({{site.url}}\assets\blender-guru-donut-part2\springkles.jpg)

- 将 Sphere 使用拉伸操拉长
- 使用 loop cut (Ctrl + R) 实现弯曲
- 将捏好的糖粒合并成一个 Collection

选择糖霜物体，在属性栏选择粒子面板，添加一个粒子设置。

![p1]({{site.url}}\assets\blender-guru-donut-part2\p1.jpg)

![p2]({{site.url}}\assets\blender-guru-donut-part2\p2.jpg)

将粒子类型设置为 Hair。

![p3]({{site.url}}\assets\blender-guru-donut-part2\p3.jpg)

在 Render 面板中将 Render as 设置为 Collection，在Collection子面板中将 Instance Collection 设置为刚刚创建的糖粒的 Collection。调整 Scale 的值使糖粒大小合适。

![p4]({{site.url}}\assets\blender-guru-donut-part2\p4.jpg)

此时所有的糖粒的朝向相同。选中 Advance 选项，找到 Rotation 并选中，面板中具体参数设置参考如下。

![p5]({{site.url}}\assets\blender-guru-donut-part2\p5.jpg)

- 适当增加 Randomize 的值可以使糖粒不完全贴合表面而更加真实。

在 Emission 面板中可以更改粒子的数量和随机种子的值。

![p6]({{site.url}}\assets\blender-guru-donut-part2\p6.jpg)

在编辑界面使用快捷键 Ctrl + Tab 选择 Weight Paint Mode 绘制粒子分布的热图。

![p7]({{site.url}}\assets\blender-guru-donut-part2\p7.jpg)

在粒子属性面板关闭粒子显示来省去粒子的渲染从而更加流畅的绘制热图。

![p8]({{site.url}}\assets\blender-guru-donut-part2\p8.jpg)

**为 Springkles 添加材质。**

使用 ColorRamp Node 给 Springkles 添加多种色彩。首先在添加一个 Object Info Node 作为输入，其 Random 

作为 ColorRamp Node 的输入，ColorRamp Node 的 Color 作为 Principled BSDF 的 Base Color 的输入。

![p9]({{site.url}}\assets\blender-guru-donut-part2\p9.jpg)

使用 Constant 参数可以让每种颜色皆为纯色，每种颜色在色带上的占比就是对应颜色的糖粒在所有糖粒中的比例。

效果如下。

![d1]({{site.url}}\assets\blender-guru-donut-part2\donut1.jpg)



## 展 UV（UV Unwrapping）

一张直观解释 UV Mapping 的图。

![uvmapping]({{site.url}}\assets\blender-guru-donut-part2\UVMapping.jpg)

在通过绘制纹理给甜甜圈增加细节之前，按理说应该先进性展 UV 的操作。不过由于甜甜圈是用基础几何图形捏的，所以 blender 已经完成了展 UV 的操作。

## 绘制纹理（Texture Painting）

进入 Texture Paint 界面。可以看到甜甜圈变成了粉色。这代表这个物体现在缺少纹理。

![tp1]({{site.url}}\assets\blender-guru-donut-part2\tp1.jpg)

在绘制纹理窗口新建一个图片。

![tp2]({{site.url}}\assets\blender-guru-donut-part2\tp2.jpg)

将图片底色涂成面包表面的颜色并保存。在甜甜圈物体的 shading 界面，为其添加一个 Image Texture Node，将创建的 Texture 导入并作为Principled BSDF 的 Base Color 的输入。再切回 Texture Paint 界面，此时甜甜圈已经贴上了纹理，此时给甜甜圈用画笔增加细节即可。注意应直接在三维物体上绘制，如果在纹理上绘制会造成三维物体上产生颜色的断面。

在 Texture Properties 面板可以新建一个纹理作为 Texture Paint 画笔的遮罩。

![tp3]({{site.url}}\assets\blender-guru-donut-part2\tp3.jpg)

在绘制界面使用热键 N 呼出画笔的属性面板，在 Texture Mask 一项选中刚才创建的纹理作为遮罩并选中 random 以使每次绘制时的图案都不同。

![tp4]({{site.url}}\assets\blender-guru-donut-part2\tp4.jpg)

贴上绘制纹理的甜甜圈效果如下。

![tp5]({{site.url}}\assets\blender-guru-donut-part2\tp5.jpg)

## 程序式纹理（Procedural Texturing）

使甜甜圈表面具有凹凸不平的纹理，并模拟出油炸时因气泡产生的大的隆起以及由于温度更高而随着隆起变深的颜色的效果。

使用 Noise Texture Node 来模拟表面凹凸起伏的分布情况。

**在 Shading 界面显示某个结点对物体的效果的方法**

在 Edit 菜单打开 Preferences，在 Blender Preferences 窗口选择 Add-ones 面板，查找关键词 node 并选中选项 Node Wrangler。

然后使用热键 Ctrl + Shift + LMB 单击想要查看效果的输出接口，就会自动创建一个 Viewer Node 与之相连并连接到材质输出的 Surface 接口以显示效果。

![pt1]({{site.url}}\assets\blender-guru-donut-part2\pt1.jpg)

显示：

![pt2]({{site.url}}\assets\blender-guru-donut-part2\pt2.jpg)

此时的噪声为絮状，和油炸面团的表面并不相似。使用 Texture Coordinate Node 来改变噪声在物体表面的分布。将 Texture Coordinate Node 的 Texture 与 Noise Texture Node 的 Vector 输入接口相连并增大 Noise Texture Node 的 Scale 属性值，得到效果如下。

![pt3]({{site.url}}\assets\blender-guru-donut-part2\pt3.jpg)

将 Noise Texture Node 的 Fac 输出与 Displacement Node 的 Height 输入相连，Displacement 的输出连接到材质的 Displacement 接口，调整 Noise Texture Node 的参数得到效果如下：

![pt4]({{site.url}}\assets\blender-guru-donut-part2\pt4.jpg)

此时物体的边缘并没有凹凸的效果。

在材质属性面板的 Setting 一栏中将 Displacement 的值选为 Displacement and Bump，此时甜甜圈会“炸开”。

![pt5]({{site.url}}\assets\blender-guru-donut-part2\pt5.jpg)

调低 Displacement Node 中 Scale 属性的值，得到效果如下。

![pt6]({{site.url}}\assets\blender-guru-donut-part2\pt6.jpg)

此时甜甜圈表面更像是墙壁而不是食物，给其增加一点 Subserface 并设置 Subserface Color，使得表面看上去更加柔软。

![pt7]({{site.url}}\assets\blender-guru-donut-part2\pt7.jpg)

为了模拟油炸产生的凸起，添加一个 Scale 值小一些的 Noise Texture，将其输出连入一个 ColorRamp Node，调整噪声黑白分布的比例，效果如下。

![pt8]({{site.url}}\assets\blender-guru-donut-part2\pt8.jpg)

将其和细密的 Noise Node 的输出相加，使用 MixRGB Node 的 Add 模式，其原理为 Color1 + Color2，Fac 的值此时为 Color2 的系数，要固定大的凸起调整小的凸起故将粗糙的 Texture Node 的输出作为 Color1。效果如下。

![pt8]({{site.url}}\assets\blender-guru-donut-part2\pt8.png)

将其输出作为一个 MixRGB Node 的 Fac 输入，可以使其作为一个遮罩，将纹理颜色作为 Color1 的输入，Color2 选择烤焦的颜色，混合方式选择 Overlay，得到效果如下。

![pt9]({{site.url}}\assets\blender-guru-donut-part2\pt9.jpg)

具体的 Shading Map 如下。

![pt10]({{site.url}}\assets\blender-guru-donut-part2\pt10.jpg)

最终效果如下。

![final]({{site.url}}\assets\blender-guru-donut-part2\final.jpg)

