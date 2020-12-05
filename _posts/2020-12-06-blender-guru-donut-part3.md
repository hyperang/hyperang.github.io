---
layout: post
title:  "Blender基础 Level3"
date:   2020-12-06 03:44:00 +0800
tags: Blender
color: rgb(255,90,90)
cover: '../assets/blender-guru-donut-part3/final.jpg'
subtitle: '来一杯美式吧'
---
Blender入门，文章框架以及主要内容为 [Blender Guru][link-1] 的 [tutorial][link-2] 的第三部分。部分内容来自 [DECODED][link-3] 的 [tutorial][link-4]。笔记一节记录了操作过程中具体实现的细节以及遇到的问题和解决方法。展 UV 和法线贴图一节中使用了[插件][link-5]和[法线贴图][link-6]。

[link-1]: https://www.youtube.com/channel/UCOKHwx1VCdgnxwbjyb9Iu1g
[link-2]: https://www.youtube.com/watch?v=NyJWoyVx_XI&list=PLjEaoINr3zgEq0u2MzVgAaHEBt--xLB6U&index=1&ab_channel=BlenderGuru
[link-3]: https://www.youtube.com/channel/UC6gJtkKLq7MTkm0SJRpYBWg
[link-4]: https://www.youtube.com/watch?v=PCuVNF5RQHg&ab_channel=DECODED
[link-5]: https://github.com/Radivarig/UvSquares
[link-6]: https://www.dropbox.com/s/9j6m709qmu4bfvz/WaterDropletsMixedBubbled001_NRM16_1K.tif?dl=0

## 内容梗概

精准建模（Precision Modelling）

透明（Transparency）

照片纹理化（Photo texturing）

体积（Volumetrics）

展 UV 和法线贴图（UV Unwarpping & Normal Texture）

笔记（Notes）

## 精准建模（Precision Modelling）

将图片导入编辑界面：

Shift + A $\longrightarrow$ Image $\longrightarrow$ Reference

注意选项 Align to view，如果选中则图片导入时的位置和角度与当前的视角相关。

新建一个 Cylinder，将其编辑至与图片中的杯子贴合。

- 使用 Alt + LMB 选中一个 Loop
- Ctrl + R 添加一个 Loop
- 新加 Loop 的位置为杯子把手和杯壁连接的顶部和底部

![pm1]({{site.url}}\assets\blender-guru-donut-part3\pm1.jpg)

选中最上方的 Loop，删除（热键 X）最上方的面。

为其添加一个 Solidify Modifier 以添加厚度。

**在为把手建模之前先将 Solidify Modifier 确认**。

将视角调整至俯视图，旋转至一个分面和图片垂直。选中一个分面按 E 拉伸把手的第一段，重复。或者使用 Spin 工具，选择 Spin 工具，Shift + RMB 设置 Cursor，（选中要拉伸的面的状态下）按住 LMB 拖拽。Spin 工具见下图。

**在 Edit Mode 中，数字键 1 为 Vertices 模式，即每次选中一个 vertex，数字键 2 为 Edges 模式，数字键 3 为 Faces 模式**。

![spin]({{site.url}}\assets\blender-guru-donut-part3\spin.jpg)

选中要组成面的点后按 F 创建一个面，或者选中要连接的两端的所有点后 F3 $\longrightarrow$ bridge edge loops，或者 Ctrl + E  $\longrightarrow$ bridge edge loops

在添加了 Subsurface Modifier 后，杯底会出现非常不和谐的褶皱。

![pm2]({{site.url}}\assets\blender-guru-donut-part3\pm2.jpg)

解决方法就是在杯底添加 Loop。**在圆内添加 Loop 的快捷键为 I**。

![pm3]({{site.url}}\assets\blender-guru-donut-part3\pm3.jpg)

效果如下。

![pm4]({{site.url}}\assets\blender-guru-donut-part3\pm4.jpg)

接下来捏杯碟。

热键 Shift + S $\longrightarrow$ Cursor to Select 使光标移动到选中物体的中心（即杯子的中心）。这一步是为了方便确定盘子的位置。

新建一个 Circle，其 Fill Type 设置为 Ngon 以使圆中有一个面。Vertices 设置为 16。

新建一个 Loop 作为盘底（热键 I），然后增加 Loop 作为细节（热键 Ctrl + R + Scroll Wheel）。开启 Proportional Editing Objects Mode 并将其模式选择为 Sphere。选中杯底的 Loop 向下移动到合适的位置即可产生平滑的弧面。

给 Circle 添加一个 Solidify Modifier 并确认。

添加 Loop 并向上拉起来模拟杯碟上用以卡住杯子的环凸。注意要**将 Snap 关闭**。在环凸的两侧各添加一个 Loop 来使得凸起更加明显，类似的技巧可以用于使杯口和碟边更加厚实圆润。

![pm5]({{site.url}}\assets\blender-guru-donut-part3\pm5.jpg)

效果：

![pm6]({{site.url}}\assets\blender-guru-donut-part3\pm6.jpg)

## 透明（Transparency）

实现玻璃材质有两种方法，一种是直接给物体一个内置的玻璃材质。具体实现如下图。

![t1]({{site.url}}\assets\blender-guru-donut-part3\t1.jpg)

另一种方式是调节 Principled BSDF 的参数。透明度 Transmission 的值设为 1，Roughness 以及 Transmission Toughness 的值设置为 0，Base Color 设置为纯白即可得到全透明的玻璃材质。

液体的参数与玻璃相同，也可以通过上面的两种方式实现。

IOR（折射率）也是一个重要的参数，不同材质的 IOR 参考如下表。

| 英文名称               | 中文名称                       | 最小值 | 最大值 |
| ---------------------- | ------------------------------ | ------ | ------ |
| Acrylic glass          | 有机玻璃(树脂、亚克力)         | 1.490  | 1.492  |
| Air                    | 空气                           | 1.000  |        |
| Alcohol, Ethyl (grain) | 酒类(酒精、乙醇、含沉淀物颗粒) | 1.360  |        |
| Aluminum               | 铝                             | 1.390  | 1.440  |
| Asphalt                | 沥青、柏油                     | 1.635  |        |
| Beer                   | 啤酒                           | 1.345  |        |
| Bronze                 | 青铜                           | 1.180  |        |
| Copper                 | 铜                             | 1.100  | 2.430  |
| Crystal                | 水晶、石英、结晶               | 2.000  |        |
| Diamond                | 钻石                           | 2.418  |        |
| Emerald                | 翡翠、绿宝石、祖母绿           | 1.560  | 1.605  |
| Eye,Lens               | 眼睛、镜头                     | 1.410  |        |
| Glass                  | 玻璃                           | 1.500  |        |
| Glass,Pyrex            | 耐热玻璃                       | 1.474  |        |
| Gold                   | 金                             | 0.470  |        |
| Ice                    | 冰                             | 1.309  |        |
| Iron                   | 铁                             | 2.950  |        |
| Ivory                  | 象牙                           | 1.540  |        |
| Lead                   | 铅                             | 2.010  |        |
| Lucite                 | 透明的合成树脂                 | 1.495  |        |
| Mercury(liquid)        | 汞、水银(液体)                 | 1.620  |        |
| Milk                   | 奶、乳汁                       | 1.350  |        |
| Nickel                 | 镍                             | 1.080  |        |
| Nylon                  | 尼龙(丝袜)                     | 1.530  |        |
| Pearl                  | 珍珠                           | 1.530  | 1.690  |
| Plastic                | 塑料                           | 1.460  |        |
| Teflon                 | 特氟隆(不粘涂层、氟碳涂料)     | 1.350  | 1.380  |
| Titanium               | 钛                             | 2.160  |        |
| Vodka                  | 伏特加                         | 1.363  |        |
| Water (35 deg C)       | 水(35°)                        | 1.325  |        |

更多材质可参考[网站][link-7]。

[link-7]: https://www.btbat.com/12032.html

**在实现杯子中的液体的时候需要将液体的边缘和杯壁重合**，这样光线的折射才是正确的。

采用第一种方式实现玻璃材质，第二种方式实现液体材质的效果如下。

![t2]({{site.url}}\assets\blender-guru-donut-part3\t2.png)



## 体积（Volumetrics）

**模拟咖啡的质感**

清水的透明度很高，但是浑浊的水透明度很低。这是因为浑浊的水中有很多杂质，光线在这些杂质粒子弹射的过程中其能量被消耗，无法射入液体更深的位置。咖啡颜色深也是同样的原理。

为了模拟这种效果，添加一个 Volume Absorption Node 并将其输出连接到 Meterial Output 的 Volume 接口。将Principled BSDF 中的 Base Color 设置为纯白，而将 Volume Absorption 中的 Color 设置为最上层咖啡液体的颜色（浅棕色），即水是无色的而咖啡萃取物的颜色为浅棕色，更深位置的黑色是由 Volume Absorption Node 中的深度 Density 带来的，为其赋一个较大的值。具体设置如下。

![t3]({{site.url}}\assets\blender-guru-donut-part3\t3.jpg)

效果如下：

![t4]({{site.url}}\assets\blender-guru-donut-part3\t4.jpg)



## 展 UV 和法线贴图（UV Unwarpping & Normal Texture）

选中一个 Loop $\longrightarrow$ Ctrl + E $\longrightarrow$ Mark Seam $\longrightarrow$ 选中物体的所有 Vertices $\longrightarrow$ U $\longrightarrow$ Unwarp。

在 UV Editing 面板选中要铺平的部分（选中一个点后按 L 全选） $\longrightarrow$ N $\longrightarrow$ UV Squares  $\longrightarrow$ To Grid By Shape。

选中杯子，新建一个材质，复用 Glass 并独立这个材质，并命名为 Condensation。

![c0]({{site.url}}\assets\blender-guru-donut-part3\c0.jpg)

选中水面上方的杯子内侧的面后点击材质属性面板的 Assign 使被选中的面与材质 Condensation 关联。

打开 Shading 界面，选择 Condensation 材质，为其添加一个 Image Texture Node，选择准备好的法线贴图。

将 Image Texture 中的 Color Space 选择为 Non-Color，在 Image Texture 和 Glass BSDF 的 Normal 接口之间要加一个 Normal Map Node。

![c2]({{site.url}}\assets\blender-guru-donut-part3\c2.jpg)

改变 Normal Map Node 中的 Strength 调节法线贴图的效果。

最终效果如下。

![final]({{site.url}}\assets\blender-guru-donut-part3\final.jpg)



## 笔记（Notes）

- 由于 Guru 为了课程分割了制作过程，故其建模的方式并不是最优的。建模之前应明确建模的对象并规划好整个流程以优化效率。
- 在处理圆面时可以使用 Merge（热键 M） $\longrightarrow$ Collapse 使边上的各点交汇于圆心一点。
- 材质渲染或 Subsurface Modifier 产生错误时，可能是由于物体表面的法线方向错误。

![Subsurface Error]({{site.url}}\assets\blender-guru-donut-part3\SubsurfaceError.jpg)

检测方式为在 Overlays 显示选项中勾选 Face Orientation 选项。

![Overlays]({{site.url}}\assets\blender-guru-donut-part3\overlays.png)

此时会显示每个面的法线方向。

![FaceOrientation]({{site.url}}\assets\blender-guru-donut-part3\FaceOrientation.jpg)

确保每个面都为蓝色。反转法线方向的方式为选中一个面 $\longrightarrow$ Shift + N。

![NormalRight]({{site.url}}\assets\blender-guru-donut-part3\NormalRight.jpg)

- 在液体底部的 Loop 缩放到杯壁外，可以做出玻璃杯杯底大块玻璃的效果。
- 在相机视角下按 Ctrl + B 后框选可以选择一部分进行渲染以提高渲染速度并专注细节。
- 提示 Object has non-uniform scale. Unwarp will operate on a ...，选中物体 $\longrightarrow$ Ctrl + A 使物体的 Scale 为 1。

